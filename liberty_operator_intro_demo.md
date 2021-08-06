# Demo Liberty Operator
## What does the operator do?
The operator creates an abstraction of the kubernetes native objects to simplify the deployment and operations of Liberty based java applications on Kubernetes,

## Deploy and manage
Set namespace
```bash
export NAMESPACE=johan
```

### Dependencies
* persistent volume claim required for serviceability, this requires are read, write, many volume
```bash
echo "apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-app-serviceability
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ocs-storagecluster-cephfs
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi" | oc apply -n $NAMESPACE -f -

```

### Create the Operator
Show this in the console - you can talk through the different sections ect
For this demo a default deployment with:
* label: app=frontend
* serviceability size=1G, volumeClaimName= demo-app-serviceability


Commandline version
```bash
echo "apiVersion: openliberty.io/v1beta1
kind: OpenLibertyApplication
metadata:
  name: demo-app
  labels:
    app: frontend
spec:
  expose: true
  applicationImage: >-
    registry.connect.redhat.com/ibm/open-liberty-samples@sha256:8ea2d3405ff2829d93c5dda4dab5d695ea8ead34e804aaf6e39ea84f53a15ee4
  serviceability:
    size: 1G
    volumeClaimName: demo-app-serviceability
  replicas: 1" | oc apply -n $NAMESPACE -f -
```

### Talk through
* All the components that the operator just created: Route, Deployement, Service, Secrets ect.
* Remember the operator is managing the app - show two screen one showing the deployments in the console and one the developer topology. Delete the app and notice how the operator brings back the kubernetes components and brings the app back.

### Pod scaling...
* This is done via the operator, non-operator pods can do this via the console. For this go to the operator and edit the pod count - also show the autoscaling policies.

## Visibility and debugging

### Get the application route and generate some load
```bash
export APP_ROUTE=`oc get routes | grep -i demo-app | awk '{printf "http://%s", $2}'`
```
Generate some load
```bash
for ((i=1;i<=1000;i++)); do curl $APP_ROUTE; done
```

### Basic Monitoring
* Basic monitoring is available right from the developer topology. Show the monitoring tab first. 
* Show the Grafana dashboards and talk through the option to customise: 
* Dashbord showing how resources in an application namespace is behaving: Default / Kubernetes / Compute Resources / Namespace (Pods)
* You can also see the complete cluster to see resource intensive namespaces: Default / Kubernetes / Compute Resources / Cluster 
* Liberty also has its own specific dashboards


### General debugging
Show how you can access the pod terminal in the console.

### Dump log files
Create PVC
```bash
```
Get the Pod Name and setup environment variables
```bash
export PODNAME=`oc get pod | grep -i demo-app | awk '{print $1}'`
```
Create Dump
```bash
# create Dump
echo "apiVersion: openliberty.io/v1beta1
kind: OpenLibertyDump
metadata:
  name: example-dump
  labels:
    app: frontend
spec:
  include:
    - heap
    - thread
    - system
  podName: $PODNAME" | oc apply -n $NAMESPACE -f -

#Check dump needs to be started=true
oc get OpenLibertyDump
```
Copy Dump
```bash
# to see the files
oc exec $PODNAME ls /serviceability/$NAMESPACE/$PODNAME

# oc cp [name_space]/[podname]:/serviceability/[name_space]/[podname] . 
oc cp $NAMESPACE/$PODNAME:/serviceability/$NAMESPACE/$PODNAME .

```
Delete the dump
```bash
oc delete OpenLibertyDump example-dump
# oc exec $PODNAME "rm /serviceability/$NAMESPACE/$PODNAME/2021-08-06_02:06:41.zip"
```

### Create Trace
Create the trace
```bash
echo "apiVersion: openliberty.io/v1beta1
kind: OpenLibertyTrace
metadata:
  name: example-trace
  labels:
    app: frontend
spec:
  podName: $PODNAME
  traceSpecification: '*=info:com.ibm.ws.webcontainer*=all'" | oc apply -n $NAMESPACE -f -

```
Validate trace
```bash
oc get OpenLibertyTrace
```
See the trace files
```bash
# to see the files
oc exec $PODNAME ls /serviceability/$NAMESPACE/$PODNAME

# Should show messages.log and trace.log

# oc cp [name_space]/[podname]:/serviceability/[name_space]/[podname] . 
oc cp $NAMESPACE/$PODNAME:/serviceability/$NAMESPACE/$PODNAME .

```
Delete the trace
```bash
oc delete OpenLibertyTrace example-trace
```

### what about central metrics and monitoring
This is part of what needs to be setup and managemed: [Observability Deployment](https://github.com/OpenLiberty/open-liberty-operator/blob/master/doc/observability-deployment.adoc)