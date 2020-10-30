odo 2.0: Cloud-native application development on OpenShift
extra cluster:
oc login --token=xH_2aNr5vcnLcWnitLDFrmGSF8MNfwbkdCDJhtqewnI --server=https://api.cluster-6da1.6da1.example.opentlc.com:6443

# Setup

* Access to an OpenShift cluster
  * Install odo with binary installation from odo.dev
  * Devfile file reference at odo.dev/file-reference

# Script

## Thank Edson
* Give background info
* Red Hat Developer channel work
* Thanks for being here

## Introduce odo
* Give background information (openshift do)
* Functunality, basic info
* Give outline of talk (devfiles, updating S2I, starting application)

## Introduce self
* Position, school, kubernetes experience

## Resources
* How to install, resources/docs
* Katacoda scenario

## What is odo
* CLI tool for creating apps on k8s
* Supplement for oc for allowing dev's to focus on code
  * Simple syntax, client based, node/java support
  * Speeds up the inner cycle
  
## Devfiles
* Portable YAML file to describe dev enviroment
  * Can reproduce dev enviroment on any k8s system
* Can describe container definition, commands, & cloning project
* Uses default registry but can add new
```
odo create
```

## Deploy NodeJS App
* Create a new project
```
odo project create weather
```
* Clone weather project
```
git clone https://github.com/phattp/nodejs-weather-app.git
```
* cd, create component configuration with name weather
```
odo create nodejs weather
```
* create url to access deployed content
```
odo url create
```
* push the component to cluster, create & run pod
``` 
odo push
```
* list urls of component
```
odo url list
```

## Watch changes
* watch for changes in directory for component
```
odo watch
```

## Set enviroment variables
* To set an enviroment variable
```
odo config set --env TITLE="Hello Red Hat"
```
* Push the configuration changes to the cluster
```
odo push --config
```

## S2I Updating
* Import openjdk to cluster
```
oc import-image openjdk18 \
--from=registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift --confirm
```
* tag image as builder to make accessible for odo
```
oc annotate istag/openjdk18:latest tags=builder
```
* check to see created image
```
odo catalog list components
```
* clone demo repo into folder
```
git clone https://github.com/openshift-evangelists/Wild-West-Backend backend
```
* build source files with maven to create JAR
```
mvn package
```
* create a component configuration of java 
```
odo create openjdk18 backend --binary target/wildwest-1.0.jar
```
* run convert command to create devfile
```
odo utils convert-to-devfile
```
* push component to your cluster
```
odo push
```
## Minikube
* Start minikube
```
minikube start
```
* find ingress ip of kubernetes
```
minikube ip
```
* create a new project
```
odo project create minikube
```
* list all devfile components
```
odo catalog list components
```
* download example nodejs component
```
git clone https://github.com/odo-devfiles/nodejs-ex
```
* cd, then create component
```
odo create nodejs mynodejs
```
* create url in order to access deployed content
```
odo url create --host x
```
* push component to the cluster
```
odo push
```
* list the urls of component
```
odo url list
```
* view your deployed application using url
```
curl x
```
* delete your deployed application
```
odo delete
```
