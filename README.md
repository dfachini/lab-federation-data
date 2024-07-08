# lab-federation-data
Laboratory to federate data from PostgreSQL and MongoDB using Trino

# Minikube [Local]

### cluster size configuration
```sh
# kubernetes cluster information
* Provider = Minikube
* Version = 1.30.0
* Type = Ubuntu 20.04.6 LTS
* CPUs = 4
* Memory = 8 GiB
* Node Count = 1
```

### log into development cluster
```sh
# access kubernetes cluster
minikube start [minikube start --memory 5000 --cpus 4]
minikube tunnel
minikube dashboard
```