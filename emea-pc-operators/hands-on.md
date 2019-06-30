# Hands On Operator Notes

1. [Create the Operator Scaffold](#scaffold)
1. [Create Custom Resource Definitions](#custom-resource-definitions)
1. [Create a Controller](#controllers)
1. [Add Handling Logic to the Controller](#controller-logic)
1. [Run the Operator](#running)
1. [Test the Operator](#testing)

## Scaffold

* Create the necessary directory (following Golang conventions) under `$GOPATH`. For example (replace "jdob" with your GitHub username):
```bash
cd $GOPATH
mkdir -p src/github.com/jdob
cd src/github.com/jdob
```

* Use the SDK to create the scaffold for the operator code:
```bash
operator-sdk new demo-operator --dep-manager=dep
cd demo-operator
OP_DIR=`pwd`
```

* Don't change any of the files that were created :)

## Custom Resource Definitions

* For each custom resouce the operator will manage, a definition (CRD) must be created. The `api-version` is used as a namespace for the definition and the `kind` refers to the custom resource name. This is done through the `add api` command in the SDK and should be run from the root of the operator scaffold:

```bash
operator-sdk add api --api-version=demo.dobtech.dev/v1alpha1 --kind=DemoSite 
```

* The `*_types.go` file will be edited to include any fields the custom resource should accept. In this example, that file is found at `pkg/apis/demo/v1alpha1/demosite_types.go`.

* Using your favorite editor, open the `_types.go` file and add the following to the `DemoSiteSpec` struct (being careful to indent appropriately):

```go
Size int32 `json:"size"`
MinikubeIP string `json:"minikube"`
```

* In the same file, add the following to the `DemoSiteStatus` struct:

```go
Visitors int32 `json:"visitors"`
```

* Run the `generate` command to update the generated files with the new fields (this must be done from the root of the operator code):

```bash
cd $OP_DIR
operator-sdk generate k8s
```

## Controllers

* Generate the stub code for a controller that will manage the custom resource:

```bash
cd $OP_DIR
operator-sdk add controller --api-version=demo.dobtech.io/v1alpha1 --kind=DemoSite 
```

## Controller Logic

For simplicity, much of the code will be downloaded directly from git. These will be explained during the guided hands on demo.

* Copy the deployment and service definitions for the pods that are necessary to run the demo application:

```bash
cd $OP_DIR/pkg/controller/demosite
wget https://raw.githubusercontent.com/jdob/session-materials/master/emea-pc-operators/operator-code/common.go
wget https://raw.githubusercontent.com/jdob/session-materials/master/emea-pc-operators/operator-code/backend.go
wget https://raw.githubusercontent.com/jdob/session-materials/master/emea-pc-operators/operator-code/frontend.go
wget https://raw.githubusercontent.com/jdob/session-materials/master/emea-pc-operators/operator-code/mysql.go
```

* **Important** Update all of the downloaded files, changing the `demov1alpha1` import to reflect your GOPATH accordingly. In particular, replace the "jdob" portion of the filepath with your GitHub username.

* All of our custom logic will go in the `Reconcile()` function. We will replace *most* of the stub code, so the first step is to delete the lines we don't need. Everything from `// Define a new Pod object` (this should be around line 103) until the end of the function (be careful to not delete the closing `}` should be deleted.

* Starting at the point where `// Define a new Pod object` was deleted, insert the following code (be careful to honor the correct indentation required by Golang):

```go
	var result *reconcile.Result

	// == MySQL ==
	result, err = r.ensureDeployment(request,
		instance,
		"mysql",
		r.mysqlDeployment(instance))
	if result != nil {
		return *result, err
	}

	result, err = r.ensureService(request,
		instance,
		"mysql",
		r.mysqlService(instance))
	if result != nil {
		return *result, err
	}
	r.waitForMysql(instance)

	// == Visitors Service ==
	result, err = r.ensureDeployment(request,
		instance,
		instance.Name+"-backend",
		r.backendDeployment(instance))
	if result != nil {
		return *result, err
	}

	result, err = r.ensureService(request,
		instance,
		instance.Name+"-backend-service",
		r.backendService(instance))
	if result != nil {
		return *result, err
	}

	// == Visitors Web UI ==
	result, err = r.ensureDeployment(request,
		instance,
		instance.Name+"-frontend",
		r.frontendDeployment(instance))
	if result != nil {
		return *result, err
	}

	result, err = r.ensureService(request,
		instance,
		instance.Name+"-frontend-service",
		r.frontendService(instance))
	if result != nil {
		return *result, err
	}

	// Everything went fine, don't requeue
	return reconcile.Result{}, nil
```

* The replacement code does not use some of the imports that were originally used by the generated stub and must be deleted. The two import lines to remove are as follows (they should be on lines 11 and 14):

```
	"k8s.io/apimachinery/pkg/types"
	"sigs.k8s.io/controller-runtime/pkg/controller/controllerutil"
```

## Running

* Deploy the CRD into the cluster:

```bash
cd $OP_DIR/deploy/crds
kubectl create -f *_crd.yaml
```

* Set the operator name as an environment variable:

```bash
export OPERATOR_NAME=demo-operator
```

* Start the operator in the "default" namespace, which will run outside of the cluster to simplify testing/debugging:

```bash
cd $OP_DIR
operator-sdk up local --namespace default
```

## Testing

To test the operator, a custom resource of the defined type must be created. The SDK generates a sample template that can be deployed, though some minor additions are needed.

* Determine the IP for the minikube cluster:

```bash
minikube ip
```

* Edit the generated custom resource template, which can be found in the same directory as the CRD (`$OP_DIR/deploy/crds`). In the `spec` section, make the following changes (be sure to honor the existing indentation):
  * Change the value of `size` to 2 (this is simply to save resources in the cluster)
  * Add a new key `minikube` with the value being the IP of the minikube cluster as found above.

* Create the DemoSite resource in the cluster:

```bash
kubectl create -f demo_v1alpha1_demosite_cr.yaml
```

* Follow the deployment of the created resources. This can be done in a number of ways, but the simplest is to create a watch on pod creation:

```bash
watch -n 3 kubectl get pods
```

The output should appear similar to the following:
```bash
NAME                                         READY   STATUS    RESTARTS   AGE
example-demosite-backend-76d7f79c8b-lg6jm    1/1     Running   0          102s
example-demosite-backend-76d7f79c8b-tblvh    1/1     Running   0          102s
example-demosite-frontend-568dbd5888-79vh6   1/1     Running   0          102s
mysql-6794f4f95c-ndmdb                       1/1     Running   0          115s
```

The demo application is fully deployed once all of the pods are in the `Running` state (see the above output for an example).

* Open the web UI in a browser. The URL is the IP of the minishift cluster and the port is `30686`. For example: `http://192.168.99.100:30686/`.

* On each refresh of the page, a new entry should be added to the table. The "Service IP" should alternate between requests, showing the load balancing across the two backend services.

## Clean Up

* The DemoSite resource can be deleted using the `demo_v1alpha1_demosite_cr.yaml` file:

```bash
kubectl delete -f demo_v1alpha1_demosite_cr.yaml
```

This will delete not only the resource itself, but any child resources (deployments, pods, services) as well.
