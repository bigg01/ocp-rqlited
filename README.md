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

## add some data
```sh
127.0.0.1:4001> CREATE TABLE team (id INTEGER NOT NULL PRIMARY KEY, name TEXT)
0 row affected (0.000000 sec)
127.0.0.1:4001> INSERT INTO team(name) VALUES("oliver")
1 row affected (0.000000 sec)
127.0.0.1:4001> INSERT INTO team(name) VALUES("marc")
1 row affected (0.000000 sec)
127.0.0.1:4001> INSERT INTO team(name) VALUES("dominik")
1 row affected (0.000000 sec)
127.0.0.1:4001> INSERT INTO team(name) VALUES("cyrill")
1 row affected (0.000000 sec)
127.0.0.1:4001>  SELECT * FROM team
+----+---------+
| id | name    |
+----+---------+
| 1  | oliver  |
+----+---------+
| 2  | marc    |
+----+---------+
| 3  | dominik |
+----+---------+
| 4  | cyrill  |
+----+---------+
```

## remove 1 node from the cluster
```sh
oc scale deployment/rq1 --replicas=0
deployment.extensions/rq1 scaled


## logs

020/01/02 21:56:37 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 50.293674952s
2020/01/02 21:56:37 [ERR] raft: Failed to AppendEntries to 172.30.182.141:4002: dial tcp 172.30.182.141:4002: i/o timeout
2020/01/02 21:56:38 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 50.78653682s
2020/01/02 21:56:38 [ERR] raft: Failed to heartbeat to 172.30.182.141:4002: dial tcp 172.30.182.141:4002: i/o timeout
2020/01/02 21:56:38 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 51.258268341s
2020/01/02 21:56:39 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 51.692517926s
2020/01/02 21:56:39 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 52.181990248s
2020/01/02 21:56:40 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 52.651911995s
2020/01/02 21:56:40 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 53.096541439s
2020/01/02 21:56:41 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 53.576479233s
2020/01/02 21:56:41 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 54.068531769s
2020/01/02 21:56:42 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 54.561593211s
2020/01/02 21:56:42 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 55.016412108s
2020/01/02 21:56:42 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 55.494893472s
2020/01/02 21:56:43 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 55.98017453s
2020/01/02 21:56:43 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 56.461085325s
2020/01/02 21:56:44 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 56.952894598s
2020/01/02 21:56:44 [DEBUG] raft: Failed to contact 172.30.182.141:4002 in 57.393325755s

$ oc logs rq3-54c7cf6b9b-sfl4b

            _ _ _
           | (_) |
  _ __ __ _| |_| |_ ___
 | '__/ _  | | | __/ _ \   The lightweight, distributed
 | | | (_| | | | ||  __/   relational database.
 |_|  \__, |_|_|\__\___|
         | |
         |_|

[rqlited] 2020/01/02 21:33:53 rqlited starting, version v4.5.0, commit 8336150318dfb2b1f196f6a4919041b65071f3fd, branch 4.3.0-patch
[rqlited] 2020/01/02 21:33:53 go1.10, target architecture is amd64, operating system target is linux
[store] 2020/01/02 21:33:53 ensuring /rqlite/file/data exists
[mux] 2020/01/02 21:33:53 mux serving on [::]:4002, advertising 172.30.43.43:4002
[store] 2020/01/02 21:33:53 SQLite in-memory database opened
[store] 2020/01/02 21:33:53 waiting for up to 2m0s for application of initial logs
2020/01/02 21:33:53 [INFO] raft: Node at 172.30.43.43:4002 [Follower] entering Follower state (Leader: "")
[cluster] 2020/01/02 21:33:53 service listening on 172.30.43.43:4002
[rqlited] 2020/01/02 21:33:53 join addresses are: [http://rq1:4001]
2020/01/02 21:33:55 [WARN] raft: EnableSingleNode disabled, and no known peers. Aborting election.
2020/01/02 21:33:56 [DEBUG] raft-net: 172.30.43.43:4002 accepted connection from: 10.128.1.51:48852
2020/01/02 21:33:56 [WARN] raft: Failed to get previous log: 3 log not found (last: 0)
[rqlited] 2020/01/02 21:33:56 successfully joined cluster at http://rq1:4001/join
2020/01/02 21:33:56 [DEBUG] raft: Node 172.30.43.43:4002 updated peer set (2): [172.30.182.141:4002]
2020/01/02 21:33:56 [DEBUG] raft: Node 172.30.43.43:4002 updated peer set (2): [172.30.105.240:4002 172.30.182.141:4002]
2020/01/02 21:33:56 [DEBUG] raft-net: 172.30.43.43:4002 accepted connection from: 10.128.1.51:48854
2020/01/02 21:33:56 [WARN] raft: Failed to get previous log: 4 log not found (last: 3)
2020/01/02 21:33:56 [DEBUG] raft: Node 172.30.43.43:4002 updated peer set (2): [172.30.43.43:4002 172.30.182.141:4002 172.30.105.240:4002]
2020/01/02 21:33:56 [DEBUG] raft-net: 172.30.43.43:4002 accepted connection from: 10.128.1.51:48868
[rqlited] 2020/01/02 21:33:57 set peer for 172.30.43.43:4002 to rq3:4001
[http] 2020/01/02 21:33:57 service listening on :4001
2020/01/02 21:52:36 [WARN] raft: Heartbeat timeout from "172.30.182.141:4002" reached, starting election
2020/01/02 21:52:36 [INFO] raft: Node at 172.30.43.43:4002 [Candidate] entering Candidate state
2020/01/02 21:52:36 [DEBUG] raft: Votes needed: 2
2020/01/02 21:52:36 [DEBUG] raft: Vote granted from 172.30.43.43:4002. Tally: 1
2020/01/02 21:52:36 [DEBUG] raft-net: 172.30.43.43:4002 accepted connection from: 10.128.1.53:51016
2020/01/02 21:52:36 [INFO] raft: Duplicate RequestVote for same term: 2
2020/01/02 21:52:37 [INFO] raft: Node at 172.30.43.43:4002 [Follower] entering Follower state (Leader: "")
2020/01/02 21:52:37 [DEBUG] raft: Node 172.30.43.43:4002 updated peer set (2): [172.30.105.240:4002 172.30.43.43:4002 172.30.182.141:4002]
2020/01/02 21:52:38 [DEBUG] raft-net: 172.30.43.43:4002 accepted connection from: 10.128.1.53:51034
2020/01/02 21:52:46 [ERR] raft: Failed to make RequestVote RPC to 172.30.182.141:4002: dial tcp 172.30.182.141:4002: i/o timeout


# check if data is still there
$ oc rsh rq2-55bfcf6b55-p245g /bin/bash
groups: cannot find name for group ID 1000520000
1000520000@rq2-55bfcf6b55-p245g:/$ cd rqlite-v4.5.0-linux-amd64
1000520000@rq2-55bfcf6b55-p245g:/rqlite-v4.5.0-linux-amd64$ ./rqlite

127.0.0.1:4001> select * from team
+----+---------+
| id | name    |
+----+---------+
| 1  | oliver  |
+----+---------+
| 2  | marc    |
+----+---------+
| 3  | dominik |
+----+---------+
| 4  | cyrill  |
+----+---------+
127.0.0.1:4001> .status
build:
  branch: 4.3.0-patch
  build_time: 2019-04-24T21:36:43-0400
  commit: 8336150318dfb2b1f196f6a4919041b65071f3fd
  version: v4.5.0
http:
  addr: [::]:4001
  auth: disabled
  redirect: rq2:4001
mux:
  addr: 172.30.105.240:4002
  encrypted: false
  timeout: 30s
node:
  start_time: 2020-01-02T21:33:57.569158685Z
  uptime: 24m36.617874473s
runtime:
  GOOS: linux
  numCPU: 4
  numGoroutine: 23
  version: go1.10
  GOARCH: amd64
  GOMAXPROCS: 4
store:
  heartbeat_timeout: 1s
  open_timeout: 2m0s
  peers: [172.30.105.240:4002 172.30.43.43:4002 172.30.182.141:4002]
  sqlite3:
    dns: 
    fk_constraints: disabled
    path: :memory:
    version: 3.28.0
  addr: 172.30.105.240:4002
  apply_timeout: 10s
  dir: /rqlite/file/data
  election_timeout: 1s
  snapshot_threshold: 8192
  db_conf:
    DSN: 
    Memory: true
  leader: 172.30.105.240:4002
  meta:
    APIPeers:
      172.30.105.240:4002: rq2:4001
      172.30.182.141:4002: rq1:4001
      172.30.43.43:4002: rq3:4001
  raft:
    last_snapshot_index: 0
    last_snapshot_term: 0
    num_peers: 2
    commit_index: 14
    fsm_pending: 0
    last_contact: 0
    last_log_index: 14
    applied_index: 14
    last_log_term: 3
    state: Leader
    term: 3


# new leader
leader: 172.30.34.141:4002 leader: 172.30.105.240:4002

```



## issues
advertise adr vor API is using internal address with the cli

```sh
./rqlite -H rqlite-client-rqlite.apps-crc.testing -p 80

rqlite-client-rqlite.apps-crc.testing:80> .tables
ERR! Get http://rq1:4001/db/query?q=SELECT+name+FROM+sqlite_master+WHERE+type%3D%22table%22: dial tcp: lookup rq1 on 127.0.0.1:53: no such host
rqlite-client-rqlite.apps-crc.testing:80> 

# in the pod it works of course

oc rsh rq1-74d9fc9dd-2zlql /bin/bash
groups: cannot find name for group ID 1000520000
1000520000@rq1-74d9fc9dd-2zlql:/$ cd r  
root/                             rqlite-v4.5.0-linux-amd64/        run/                              
rqlite/                           rqlite-v4.5.0-linux-amd64.tar.gz  
1000520000@rq1-74d9fc9dd-2zlql:/$ cd rqlite-v4.5.0-linux-amd64
1000520000@rq1-74d9fc9dd-2zlql:/rqlite-v4.5.0-linux-amd64$ ls
rqlite	rqlited
1000520000@rq1-74d9fc9dd-2zlql:/rqlite-v4.5.0-linux-amd64$ ./rqlite
127.0.0.1:4001> .status
node:
  start_time: 2020-01-02T21:33:54.873235658Z
  uptime: 14m46.246668029s
runtime:
  GOMAXPROCS: 4
  GOOS: linux
  numCPU: 4
  numGoroutine: 22
  version: go1.10
  GOARCH: amd64
store:
  peers: [172.30.43.43:4002 172.30.182.141:4002 172.30.105.240:4002]
  addr: 172.30.182.141:4002
  apply_timeout: 10s
  dir: /rqlite/file/data
  election_timeout: 1s
  leader: 172.30.182.141:4002
  meta:
    APIPeers:
      172.30.105.240:4002: rq2:4001
      172.30.182.141:4002: rq1:4001
      172.30.43.43:4002: rq3:4001
  open_timeout: 2m0s
  raft:
    fsm_pending: 0
    last_contact: never
    last_log_term: 1
    last_snapshot_term: 0
    num_peers: 2
    applied_index: 6
    commit_index: 6
    last_log_index: 6
    last_snapshot_index: 0
    state: Leader
    term: 1
  snapshot_threshold: 8192
  db_conf:
    DSN: 
    Memory: true
  heartbeat_timeout: 1s
  sqlite3:
    dns: 
    fk_constraints: disabled
    path: :memory:
    version: 3.28.0
build:
  branch: 4.3.0-patch
  build_time: 2019-04-24T21:36:43-0400
  commit: 8336150318dfb2b1f196f6a4919041b65071f3fd
  version: v4.5.0
http:
  addr: [::]:4001
  auth: disabled
  redirect: rq1:4001
mux:
  addr: 172.30.182.141:4002
  encrypted: false
  timeout: 30s
127.0.0.1:4001> .schema
+-----+
| sql |
+-----+
127.0.0.1:4001> 





```