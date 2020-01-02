# rqlited test on OCP4 example

## todo
 - helmify
 - adding auth
 - adding SSL
 - PR for dockerifle  https://github.com/rqlite/rqlite-docker

# create objects
```sh
$ oc create -f .
service/rqlite-client-access created
deployment.extensions/rq1 created
service/rq1 created
deployment.extensions/rq2 created
service/rq2 created
deployment.extensions/rq3 created
service/rq3 created

# expose route
$ oc expose service rqlite-client-access --name=rqlite-client 
route.route.openshift.io/rqlite-client exposed
$ oc get route
NAME            HOST/PORT                               PATH   SERVICES               PORT    TERMINATION   WILDCARD
rqlite-client   rqlite-client-rqlite.apps-crc.testing          rqlite-client-access   httpd                 None
$ oc get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/rq1-74d9fc9dd-2zlql    1/1     Running   0          24s
pod/rq2-55bfcf6b55-p245g   1/1     Running   0          24s
pod/rq3-54c7cf6b9b-sfl4b   1/1     Running   0          24s

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/rq1                    ClusterIP   172.30.182.141   <none>        4001/TCP,4002/TCP   24s
service/rq2                    ClusterIP   172.30.105.240   <none>        4001/TCP,4002/TCP   24s
service/rq3                    ClusterIP   172.30.43.43     <none>        4001/TCP,4002/TCP   24s
service/rqlite-client-access   ClusterIP   172.30.244.37    <none>        4001/TCP            24s

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rq1   1/1     1            1           24s
deployment.apps/rq2   1/1     1            1           24s
deployment.apps/rq3   1/1     1            1           24s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/rq1-74d9fc9dd    1         1         1       24s
replicaset.apps/rq2-55bfcf6b55   1         1         1       24s
replicaset.apps/rq3-54c7cf6b9b   1         1         1       24s

NAME                                     HOST/PORT                               PATH   SERVICES               PORT    TERMINATION   WILDCARD
route.route.openshift.io/rqlite-client   rqlite-client-rqlite.apps-crc.testing          rqlite-client-access   httpd                 None

```

## test rqlite connection
```sh
./rqlite -H rqlite-clients-rqlite.apps-crc.testing -p 80
Welcome to the rqlite CLI. Enter ".help" for usage hints.
rqlite-clients-rqlite.apps-crc.testing:80> .status
runtime:
  numCPU: 4
  numGoroutine: 16
  version: go1.10
  GOARCH: amd64
  GOMAXPROCS: 4
  GOOS: linux
store:
  snapshot_threshold: 8192
  sqlite3:
    path: :memory:
    version: 3.28.0
    dns: 
    fk_constraints: disabled
  apply_timeout: 10s
  dir: /rqlite/file/data
  leader: 172.30.34.141:4002
  meta:
    APIPeers:
      172.30.150.33:4002: rq2:4001
      172.30.166.51:4002: rq3:4001
      172.30.34.141:4002: rq1:4001
  open_timeout: 2m0s

```
