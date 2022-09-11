## Cluster Administration

```
./elasticsearch -Ehttp.port=9200 -Etransport.port=9300 -Ecluster.name=eoc-06-cluster -Enode.name=node0 -Epath.data=data0 -Epath.logs=log0  -Ecluster.initial_master_nodes="node0,node1,node2" -Ediscovery.seed_hosts="127.0.0.1:9300,127.0.0.1:9301,127.0.0.1:9302"
``` 

```
./elasticsearch -Ehttp.port=9201 -Etransport.port=9301 -Ecluster.name=eoc-06-cluster -Enode.name=node1 -Epath.data=data1 -Epath.logs=log1  -Ecluster.initial_master_nodes="node0,node1,node2" -Ediscovery.seed_hosts="127.0.0.1:9300,127.0.0.1:9301,127.0.0.1:9302"
```

```
./elasticsearch -Ehttp.port=9202 -Etransport.port=9302 -Ecluster.name=eoc-06-cluster -Enode.name=node2 -Epath.data=data2 -Epath.logs=log2  -Ecluster.initial_master_nodes="node0,node1,node2" -Ediscovery.seed_hosts="127.0.0.1:9300,127.0.0.1:9301,127.0.0.1:9302"
```


#### Download the the exam version of Elasticsearch
#### Deploy the cluster `eoc-06-cluster`, with three nodes named `node1`, `node2`, and `node3`
#### Configure the Zen Discovery module of each node so that they can communicate with each other
#### Connect a Kibana instance to `node3`
#### Start the cluster


#### Create the index `hamlet-1` with two primary shards and one replica

```
PUT hamlet-1
{
  "settings": {
    "number_of_replicas": 1,
    "number_of_shards": 2
  }
}
```

#### Add some documents to `hamlet-1` by running the following _bulk command

```
PUT hamlet-1/_bulk
{"index":{"_index":"hamlet-1","_id":0}}
{"line_number":"1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet-1","_id":1}}
{"line_number":"2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet-1","_id":2}}
{"line_number":"3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet-1","_id":3}}
{"line_number":"4","speaker":"FRANCISCO","text_entry":"Bernardo?"}
{"index":{"_index":"hamlet-1","_id":4}}
{"line_number":"5","speaker":"BERNARDO","text_entry":"He."}
```

#### Create the index `hamlet-2` with two primary shard and one replica

```
PUT hamlet-2
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  }
}
```


#### Add some documents to `hamlet-2` by running the following _bulk command
```
PUT hamlet-2/bulk
{"index":{"_index":"hamlet-2","_id":5}}
{"line_number":"6","speaker":"FRANCISCO","text_entry":"You come most carefully upon your hour."}
{"index":{"_index":"hamlet-2","_id":6}}
{"line_number":"7","speaker":"BERNARDO","text_entry":"Tis now struck twelve; get thee to bed, Francisco."}
{"index":{"_index":"hamlet-2","_id":7}}
{"line_number":"8","speaker":"FRANCISCO","text_entry":"For this relief much thanks: tis bitter cold,"}
{"index":{"_index":"hamlet-2","_id":8}}
{"line_number":"9","speaker":"FRANCISCO","text_entry":"And I am sick at heart."}
{"index":{"_index":"hamlet-2","_id":9}}
{"line_number":"10","speaker":"BERNARDO","text_entry":"Have you had quiet guard?"}
```

#### Check that the replicas of indices `hamlet-1` and `hamlet-2` have been allocated

```
GET _cluster/health/hamlet-*
```

#### Check the distribution of the primary shards and replicas of indices `hamlet-1` and `hamlet-2` across the nodes of the cluster

```
GET _cat/shards/hamlet-*?v=true
```

#### Configure `hamlet-1` to allocate both primary shards to `node2`, using the node name

```
PUT hamlet-1/_settings
{
  "index.routing.allocation.require._name": "node2"
}
```

#### Verify the success of the last action by using the _cat API

```
GET _cat/shards/hamlet-1?v
```

#### Configure `hamlet-2` so that no primary shard is allocated to `node3`

```
PUT hamlet-2/_settings
{
  "index.routing.allocation.exclude._name": "node3"
}
```

#### Verify the success of the last action by using the _cat API

```
GET _cat/shards/hamlet-2?v
```

#### Remove any allocation filter setting associated with `hamlet-1` and `hamlet-2`

```
PUT hamlet-1,hamlet-2/_settings
{
  "index.routing.allocation.require._name": null,
  "index.routing.allocation.include._name": null,
  "index.routing.allocation.exclude._name": null
}
```

#### Let's assume that we have deployed the `eoc-06-cluster` cluster across two availability zones, named `earth` and `mars`. Add the attribute `AZ` to the nodes configuration, and set its value to "earth" for `node1` and `node2`, and to "mars" for `node3`

Update configuration on `node1` and `node2`:
```
node.attr.AZ: earth
```

Update configuration on `node3`:
```
node.attr.AZ: mars
```

#### Restart the cluster

#### Configure the cluster to force shard allocation awareness based on the two availability zones, and persist such configuration across cluster restarts

Using the cluster-update-settings API:
```
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "AZ" 
  }
}
```

Updating configuration on master-eligible nodes:
```
cluster.routing.allocation.awareness.attributes: AZ 
```

#### Verify the success of the last action by using the _cat API

```
GET /_cat/nodeattrs?v
```

#### Configure the cluster to reflect a hot/warm architecture, with `node1` as the only hot node

Hot-warm-cold relies on shard allocation awarness and thus we start by labeling which nodes are hot, warm, and (optionally) cold nodes.

Update configuration on `node1`:
```
node.attr.data: hot
```

Update configuration on `node2` and `node3`:
```
node.attr.data: warm
```

#### Configure the `hamlet-1` index to allocate its shards only to warm nodes

```
PUT hamlet-1/_settings
{
  "index.routing.allocation.include.data": "warm"
}
```

#### Verify the success of the last action by using the _cat API

```
GET _cat/shards/hamlet-1?v
```

#### Remove the hot/warm shard filtering configuration from the `hamlet-1` configuration

```
PUT hamlet-1/_settings
{
  "index.routing.allocation.include.data": null
}
```

#### Let's assume that the nodes have either a "large" or "small" local storage. Add the attribute `storage` to the nodes configuration, and set its value so that `node2` is the only with a "small" storage

Update configuration on `node1` and `node3`: 
```
node.attr.storage: large
```

Update configuration on `node2`: 
```
node.attr.storage: small
```

#### Configure the `hamlet-2` index to allocate its shards only to nodes with a large storage size

```
PUT hamlet-2/_settings
{
  "index.routing.allocation.include.storage": "large"
}
```

#### Verify the success of the last action by using the _cat API

```
GET _cat/shards/hamlet-2?v
```