## Mappings and Text Analysis

### Create the index `hamlet_1` with one primary shard and no replicas

```
PUT hamlet_1
{
  "settings": {
    "number_of_shards" : 1,
    "number_of_replicas" : 0
  }
}
```

### Define a mapping for the default type "_doc" of `hamlet_1`, so that (i) the type has three fields, named `speaker`, `line_number`, and `text_entry`, (ii) `speaker` and `line_number` are unanalysed string, (iii) aggregations are not allowed for `line_number`

```
PUT hamlet_1/_mapping
{
  "properties": {
    "speaker": {
      "type": "keyword"
    },
    "line_number": {
      "type": "keyword",
      "doc_values": false
    },
    "text_entry": {
      "type": "text"
    }
  }
}
```

### Add some documents to `hamlet_1` by running the following _bulk command

```
PUT hamlet-1/_bulk
{"index":{"_index":"hamlet_1","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet_1","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet_1","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet_1","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}
{"index":{"_index":"hamlet_1","_id":4}}
{"line_number":"1.2.2","speaker":"KING CLAUDIUS","text_entry":"The memory be green, and that it us befitted"}
{"index":{"_index":"hamlet_1","_id":5}}
{"line_number":"1.3.1","speaker":"LAERTES","text_entry":"My necessaries are embarkd: farewell:"}
{"index":{"_index":"hamlet_1","_id":6}}
{"line_number":"1.3.4","speaker":"LAERTES","text_entry":"But let me hear from you."}
{"index":{"_index":"hamlet_1","_id":7}}
{"line_number":"1.3.5","speaker":"OPHELIA","text_entry":"Do you doubt that?"}
{"index":{"_index":"hamlet_1","_id":8}}
{"line_number":"1.4.1","speaker":"HAMLET","text_entry":"The air bites shrewdly; it is very cold."}
```

### Create the index `hamlet_2` with one primary shard and no replicas

```
PUT hamlet_2
{
  "settings": {
    "number_of_shards" : 1,
    "number_of_replicas" : 0
  }
}
```

### Copy the mapping of `hamlet_1` into `hamlet_2`, but also define a multi-field for `speaker`. The name of such multi-field is `tokens` and its data type is the (default) analysed string

```
PUT hamlet_2/_mapping
{
  "properties": {
    "speaker": {
      "type": "keyword",
      "fields": {
          "tokens": { 
            "type":  "text",
            "analyzer": "default"
          }
        }
    },
    "line_number": {
      "type": "keyword",
      "doc_values": false
    },
    "text_entry": {
      "type": "text"
    }
  }
}
```

### Reindex `hamlet_1` to `hamlet_2`

```
POST _reindex
{
  "source": {
    "index": "hamlet_1"
  },
  "dest": {
    "index": "hamlet_2"
  }
}
```

### Verify that full-text queries on "speaker.tokens" are enabled on `hamlet_2`

```
GET hamlet_2/_search
{
    "query": {
        "match" : {
            "speaker.tokens" : {
                "query" : "bernardo"
            }
        }
    }
}
```