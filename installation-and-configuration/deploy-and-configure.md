### Download and install Elasticsearch on MacOS

```
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.1-darwin-x86_64.tar.gz
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.1-darwin-x86_64.tar.gz.sha512
$ shasum -a 512 -c elasticsearch-7.2.1-darwin-x86_64.tar.gz.sha512
$ tar -xzf elasticsearch-7.2.1-darwin-x86_64.tar.gz
$ cd elasticsearch-7.2.1/
```

#### Running Elasticsearch
```
$ ES_PATH_CONF=elasticsearch-x.yml ./bin/elasticsearch
```

#### Check Elasticsearch is running
```
$ curl -X GET "localhost:9200/?pretty"
```

### Download and install Kibana on MacOS

```
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-7.2.1-darwin-x86_64.tar.gz
$ shasum -a 512 kibana-7.2.1-darwin-x86_64.tar.gz
$ tar -xzf kibana-7.2.1-darwin-x86_64.tar.gz
$ cd kibana-7.2.1-darwin-x86_64/ 
```



#### Configuring Kibana

```
$ vi KIBANA_HOME/config/kibana.yml
```


#### Running Kibana
```
$ ./bin/kibana
```

#### Deploy and Configure

#### Deploy the cluster `eoc-01-cluster`, so that it satisfies the following requirements: (i) has three nodes, named `node1`, `node2`, and `node3`, (ii) all nodes are eligible master nodes

Update each `elasticsearch.yaml` file: 
	* set `cluster.name: eoc-01-cluster`
	* set `node.name: nodeX` respectively 
	* set `cluster.initial_master_nodes: ["node1", "node2", "node3"]`
	* `node.master` is set to `true` by default, hence no additional configuration required to fulfill the requirements

Verify configuration by running in Kibana: 
```
GET _cat/nodes?v=true
```

#### Bind `node1` to the IP address “151.101.2.217” and port “9201”

Update configuration on `node1`: 
```
network.host: 151.101.2.217
http.port: 9201
discovery.seed_hosts: ["151.101.2.218", "151.101.2.219"]
```

#### Bind `node2` to the IP address “151.101.2.218” and port “9202”

Update configuration on `node2`: 
```
network.host: 151.101.2.218
http.port: 9202
discovery.seed_hosts: ["151.101.2.217", "151.101.2.219"]
```

#### Bind `node3` to the IP address “151.101.2.219” and port “9203”

Update configuration on `node3`: 
```
network.host: 151.101.2.219
http.port: 9203
discovery.seed_hosts: ["151.101.2.217", "151.101.2.218"]
```

#### Configure the cluster discovery module of `node2` and `node3` so as to use `node1` as seed host

Update configuration on `node1`: 
```
cluster.initial_master_nodes: ["node1"]
```

Add to the configuration on `node2` and `node3`: 
```
discovery.seed_hosts: ["151.101.2.217"]
```

#### Configure the nodes to avoid the split brain scenario

"split brain scenario" is the one when two clusters are excidentaly formed instead of one due to wrong bootstrapping configuration. To avoid this happenning, `cluster.initial_master_nodes` must contain and odd number of nodes configured.

Cluster 7.x contains a default coordination implementation which will prevent from "brain split".

There should normally be an odd number of master-eligible nodes in a cluster.

Update configuration on all nodes: 
```
cluster.initial_master_nodes: ["node1", "node2", "node3"]
```

#### Configure `node1` to be a data node but not an ingest node

Update configuration on `node1`: 
```
node.data: true
node.ingest: false
```

#### Configure `node2` and `node3` to be both an ingest and data node

Update configuration on `node2` and `node3`: 
```
node.data: true
node.ingest: true
```

#### Configure `node1` to disallow swapping on its host

Update configuration on `node1`: 
```
bootstrap.memory_lock: true
```

#### Configure the JVM settings of each node so that it uses a minimum and maximum of 2 GB for the heap

Update `jvm.options` on all nodes: 
```
-Xmx2g
-Xms2g
```

#### Configure the logging settings of each node so that (i) the logs directory is not the default one, (ii) the log level for transport-related events is set to "debug"

Update configuration on all nodes: 
```
path.logs: no_default_logs
logger.org.elsticsearch.transport: debug
```

#### Configure the nodes so as to disable the possibility to delete indices using wildcards

Update configuration on all nodes: 
```
action.destructive_requires_name: true
```

#### Create the user `francisco` with password "francisco-password"

```
POST _security/user/francisco
{
  "password": "francisco-password",
  "roles": []
}
```

#### Assign the role `francisco_role` to the `francisco` user

```
POST _security/user/francisco
{
  "roles": ["francisco_role"]
}
```

#### Login using the `francisco` user credentials, and run some queries on `hamlet` to verify that the role privileges were correctly set

```
curl -X POST -u francisco:francisco-password "http://151.101.2.217:9201/hamlet/_search" -H "Content-Type: application/json" -d '{"query": {"match_all": {}}}'
```

#### Create the security role `bernardo_role` in the native realm, so that the role (i) has "monitor" privileges on the cluster, (ii) has read-only privileges on the `hamlet` index, (iii) can see only those documents having "BERNARDO" as a `speaker`, (iv) can see only the `text_entry` field

```
POST _security/role/bernardo_role
{
  "cluster": ["monitor"],
  "indices": [
    {
      "names": ["hamlet"],
      "privileges": ["read"],
      "field_security": {
        "grant": ["text_entry"]
      }, 
      "query": "{\"match\": {\"speaker\": \"BERNARDO\"}}"
    }
  ]
}
```

#### Create the user `bernardo` with password "bernardo-password"

```
POST _security/user/bernardo
{
  "password": "bernardo-password",
  "roles": []
}
```

#### Assign the role `bernardo_role` to the `bernardo` user

```
POST _security/user/bernardo
{
  "roles": ["bernardo_role"]
}
```

#### Login using the `bernardo` user credentials, and run some queries on `hamlet` to verify that the role privileges were correctly set

```
curl -X POST -u bernardo:bernardo-password "http://151.101.2.217:9201/hamlet/_search" -H "Content-Type: application/json" -d '{"query": {"match_all": {}}}'
```

#### Change the password of the `bernardo` user to "poor-bernardo"

```
POST /_security/user/bernardo/_password
{
  "password": "poor-bernardo"
}
```