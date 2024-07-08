# Deployment

### cluster selection
```sh
# connect into k8s cluster
kubectx minikube
```

### create namespace[s]
```sh
# create namespaces
k create namespace orchestrator
k create namespace database
k create namespace ingestion
k create namespace processing
k create namespace datastore
k create namespace deepstorage
k create namespace tracing
k create namespace logging
k create namespace monitoring
k create namespace viz
k create namespace cicd
k create namespace app
k create namespace cost
k create namespace misc
```

### add & update repositories
```sh
# add & update helm list repos
helm repo add argo https://argoproj.github.io/argo-helm
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add yugabytedb https://charts.yugabyte.com
helm repo add elastic https://helm.elastic.co
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com
helm repo add strimzi https://strimzi.io/charts/
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo add infracloudio https://infracloudio.github.io/charts
helm repo add jahstreet https://jahstreet.github.io/helm-charts
helm repo add valeriano-manassero https://valeriano-manassero.github.io/helm-charts
helm repo add astronomer https://helm.astronomer.io
helm repo add spark-operator https://googlecloudplatform.github.io/spark-on-k8s-operator
helm repo add confluentinc https://confluentinc.github.io/cp-helm-charts/
helm repo add minio https://charts.min.io/
helm repo update
```

### install crd's [custom resources]
```sh
# install strimzi
helm install kafka strimzi/strimzi-kafka-operator --namespace ingestion --version 0.21.0
```

### install apps
```sh
# cluster name
https://kubernetes.default.svc

# deploy secrets
k create secret generic mssql-credentials --from-file=2-repository/yamls/ingestion/kafka-connect/mssql-credentials.properties --namespace ingestion
k create secret generic minio-credentials --from-file=2-repository/yamls/ingestion/kafka-connect/minio-credentials.properties --namespace ingestion

# services
k apply -f 3-services/lb/svc-lb-airflow-ui.yaml
k apply -f 3-services/lb/svc-lb-minio.yaml
k apply -f 3-services/lb/svc-lb-zeppelin-ui.yaml

# database
helm install yugabytedb yugabytedb/yugabyte -f 2-repository/helm-charts/yugabyte/values-development.yaml -n database
helm install postgres bitnami/postgresql -f 2-repository/helm-charts/postgresql/values-development.yaml -n database

# ingestion
k apply -f 2-repository/yamls/ingestion/broker/kafka-ephemeral.yaml -n ingestion

# deep storage
helm install minio minio/minio -f 2-repository/helm-charts/minio/values-development.yaml -n deepstorage

# processing
helm install spark spark-operator/spark-operator -f 2-repository/helm-charts/spark-operator/values-development.yaml -n processing
k apply -f 2-repository/yamls/processing/zeppelin/zeppelin-server.yaml -n processing
helm install zeppelin 2-repository/helm-charts/zeppelin --namespace processing

# orchestrator
k create -n orchestrator configmap requirements --from-file=/mnt/d/lab-k8s-data/4-products/airflow/requirements.txt
#helm install airflow bitnami/airflow -f 2-repository/helm-charts/airflow/values-development.yaml -n orchestrator
helm install airflow -n airflow apache-airflow/airflow -f 2-repository/helm-charts/airflow/values-development.yaml -n orchestrator

# housekeeping
k delete svc-lb-airflow-ui
k delete svc svc-lb-minio
k delete svc-lb-zeppelin-ui

helm delete yugabytedb -n database
helm delete postgres -n database
helm delete kafka -n ingestion
k delete statefulset edh-kafka -n ingestion
k delete statefulset edh-zookeeper -n ingestion
k delete deployment edh-entity-operator -n ingestion
k delete deployment edh-kafka-exporter -n ingestion
helm delete minio -n deepstorage
helm delete spark -n processing
k delete deployment zeppelin-server -n processing
helm delete airflow -n orchestrator
helm delete zeppelin -n processing

minikube delete
minikube stop
minikube start
```
