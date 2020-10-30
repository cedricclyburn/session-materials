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





## Create the project
```
odo project create demo
```
* Show the current project using odo
```
odo project get
```
* Show the current project is the same in `oc`
```
oc project
```

## Deploy the Backend
* Change to the backend directory
```
cd concession-kiosk-backend-java
```
* Build binary locally
```
mvn package
```
* Create the application
```
odo create java:8 backend --binary target/concessions-1.0.jar --app concessions
```
* Open up a separate watch to show resources being created
```
watch oc get all
```
* Push the local odo changes to the cluster
```
odo push
```
* Show all odo applications
```
odo list
```
* Show the deployed application pod
```
oc get pods
```

## Create the Backend URL
* Show that no URL is created by default
```
odo url list
oc get route
```
* Create the application URL (will show an error)
```
odo url create
```
* Create the application URL with all of the necessary data
```
odo url create --port 8080
```
* Show the URL doesn't exist in the cluster yet
```
oc get route
```
* Push the URL changes to the cluster
```
odo push
```
* Get the generated URL
```
odo url list
```
* Show the URL working (will show no connected database)
```
curl $ROUTE/debug
```

## Deploy MongoDB
* Show all possible services
```
odo catalog list services
```
* Show the details for mongo
```
odo catalog describe service mongodb-ephemeral
```
* Run interactively (accept defaults but wait for service to be ready):
```
odo service create --app concessions
```
* Show the application components
```
odo list
odo service list
```

## Link Backend to Database
* Show the database isn't connected automatically
```
curl $ROUTE/debug
```
* Link the database to the application (show `oc get pods`)
```
odo link mongodb-ephemeral
```
* Show the database is connected
```
curl $ROUTE/debug
```

## Deploy the Frontend
* Change to the frontend directory
```
cd concession-kiosk-frontend
```
* Create the odo application
```
odo create nodejs frontend --app concessions
```
* Create the URL before pushing
```
odo url create
```
* Push the application to the cluster
```
odo push
```
* Get the generated URL
```
odo url list
```
* Open the URL in a browser
* Submit an order (it crashes)

## Link the Frontend and Backend
* Link the two applications
```
odo link backend --port 8080
```
* Return to browser and resubmit (it works)

## Live Update
* Establish a watch for the frontend source
```
odo watch
```
* Edit views/index.ejs
* Show the pod didn't restart
```
watch oc get pods
```
* Refresh browser

## Environment Variable Example
* Edit views/index.ejs and change title to
```
<%= process.env.TITLE %>
```
* Show browser and empty title
* Set an environment variable through odo
```
odo config set --env TITLE="Hello World"
```
* Push the configuration changes to the cluster
```
odo push --config
```
* Show that this time there is a pod restart
* Show the configuration changes through odo
```
odo config view
```
* Show the odo tracking files
```
cat .odo/config.yaml
```
* Refresh browser and see the new title
