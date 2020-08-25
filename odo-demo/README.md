# Setup

* OpenShift cluster
  * Install the service catalog (playbook at https://raw.githubusercontent.com/jdob/session-materials/master/odo-demo/install-service-broker-playbook.yaml)
* Checkout:
  * https://github.com/openshift-roadshow/concession-kiosk-backend-java
  * https://github.com/openshift-roadshow/concession-kiosk-frontend

# Script

## Create the project
```
odo project create demo
```
```
odo project get
```
```
oc project
```

## Deploy the Backend
```
mvn package
```
```
odo create java:8 backend --binary target/concessions-1.0.jar --app concessions
```
```
watch oc get all
```
```
odo push
```
```
odo list
```
```
oc get pods
```

## Create the Backend URL
```
odo url list
oc get route
```
```
odo url create
```
```
odo url create --port 8080
```
```
oc get route
```
```
odo push
```
```
curl $ROUTE/debug
```

## Deploy MySQL
```
odo catalog list services
```
```
odo catalog describe service monogodb-ephemeral
```
Run interactively:
```
odo create service --app concessions
```
```
odo list
odo service list
```

## Link Backend to Database
```
curl $ROUTE/debug
```
```
odo link database
```
```
curl $ROUTE/debug
```

## Deploy the Frontend
```
odo create nodejs frontend --app concessions
```
```
odo url create
```
```
odo push
```
* Open the URL in a browser
* Submit an order

## Link the Frontend and Backend
```
odo link backend --port 8080
```
* Back to browser and resubmit

## Live Update
```
odo watch
```
* Edit views/index.ejs
* Show the pod didn't restart
```
watch oc get po
```
* Refresh browser

## Environment Variable Example
* Edit views/index.ejs and change title to
```
<%= process.env.TITLE %>
```
* Show browser and empty title
```
odo config set --env TITLE="Hello DevNation"
```
```
odo push --config
```
* Show that this time there is a pod restart
```
odo config view
```
```
cat .odo/config.yaml
```
* Refresh browser
