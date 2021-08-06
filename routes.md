# Some concepts behind routes
* Depedency deploy main and v1.0 tags of this repo: https://github.com/yohanswanepoel/flaskbook-api/blob/main/app.py
* base cluster url export as BASE_URL

### Get the application route and generate some load
```bash
export APP_ROUTE_V1=`oc get routes | grep -i flaskbook-api-git-1 | awk '{printf "http://%s/books", $2}'`
export APP_ROUTE_MAIN=`oc get routes | grep -i flaskbook-api-git-main | awk '{printf "http://%s/books", $2}'`
```

### Create blue/green route for both version to hit the same route entry point
Create bluegreen route in the UI it is guided and works well
* name: books-api
* 50/50 split

Command line route creation
```bash
echo "kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: books-api
  labels:
    app: flaskbook-api-git-1
spec:
  host: books-api-$NAMESPACE.$BASE_URL
  to:
    kind: Service
    name: flaskbook-api-git-1
    weight: 50
  alternateBackends:
    - kind: Service
      name: flaskbook-api-git-main
      weight: 50
  port:
    targetPort: 8080-tcp
  wildcardPolicy: None" | oc apply -n $NAMESPACE -f -
```

Get the route
```bash
export APP_ROUTE_SHARED=`oc get routes | grep -i books-api | awk '{printf "http://%s/books", $2}'`
```

Generate some load
```bash
for ((i=1;i<=1000;i++)); do curl $APP_ROUTE_SHARED; done
```