## Cluster Administration

#### Download the exam version of Elasticsearch
#### Deploy the cluster `eoc-06-original-cluster`, with one node named `node-1`
#### Start the cluster

#### Create the index `hamlet` and add some documents by running the following _bulk command

```
PUT hamlet/_bulk
{"index":{"_index":"hamlet","_id":0}}
{"line_number":"1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet","_id":1}}
{"line_number":"2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet","_id":2}}
{"line_number":"3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet","_id":3}}
{"line_number":"4","speaker":"FRANCISCO","text_entry":"Bernardo?"}
{"index":{"_index":"hamlet","_id":4}}
{"line_number":"5","speaker":"BERNARDO","text_entry":"He."}
```

#### Configure `node-1` to support a shared file system repository for backups, which is located in (i) "[user_home_folder]/repo" and (ii) "[user_home_folder]/elastic/repo" - e.g., "glenacota/elastic/repo"

Update configuration on `node1`:
```
path.repo: ["${HOME}/repo", "${HOME}/elastic/repo"]
```

#### Create the `hamlet_backup` shared file system repository, with location "[user_home_folder]/elastic/repo"

```
PUT /_snapshot/hamlet_backup
{
  "type": "fs",
  "settings": {
    "location": "/Users/katerynaoliferchuk/elastic/repo"
  }
}
```

#### Create a snapshot of the `hamlet` index, so that the snapshot (i) is named `hamlet_snapshot_1`, (ii) is stored into `hamlet_backup`

```
PUT /_snapshot/hamlet_backup/hamlet_snapshot_1?wait_for_completion=true
{
  "indices": "hamlet"
}
```

#### Delete the index `hamlet`

```
DELETE hamlet
```

#### Restore the index `hamlet` using `hamlet_snapshot_1`

```
POST /_snapshot/hamlet_backup/hamlet_snapshot_1/_restore
```

#### Deploy a second cluster `eoc-06-adaptation-cluster`, with one node named `node-2`
#### Start the cluster

#### Create the index `hamlet-pirate` on `node-2` and add some documents by running the following _bulk command
PUT hamlet-pirate/bulk
{"index":{"_index":"hamlet-pirate","_id":5}}
{"line_number":"6","speaker":"FRANCISCO","text_entry":"Ahoy Matey! Ye come most carefully upon yer hour."}
{"index":{"_index":"hamlet-pirate","_id":6}}
{"line_number":"7","speaker":"BERNARDO","text_entry":"Aye! Tis now struck twelve; get ye to bed, Francisco."}
{"index":{"_index":"hamlet-pirate","_id":7}}
{"line_number":"8","speaker":"FRANCISCO","text_entry":"For this relief much thanks, son of a biscuit eater"}
{"index":{"_index":"hamlet-pirate","_id":8}}
{"line_number":"9","speaker":"BERNARDO","text_entry":"Arrrrrrrrh!"}


#### Enable cross-cluster search on `eoc-06-adaptation-cluster`, so that (i) the name of the remote cluster is `original`, (ii) the seed for the remote cluster is `node-1`, which is listening on the default transport port, (iii) the cross-cluster configuration persists across multiple restarts

Set up a remote cluster:
```
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "original": {
          "seeds": [
            "151.101.2.217:9300"
          ]
        }
      }
    }
  }
}
```

#### Run the following cross-cluster query to check that your setup is correct

```
GET /original:hamlet,hamlet-pirate/_search
{
  "query": {
    "match": {
      "speaker": "BERNARDO"
    }
  }
}
```