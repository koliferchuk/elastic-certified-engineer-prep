## Indexing Data

### Create the indices `hamlet-1` and `hamlet-2`, each with two primary shards and no replicas

```
PUT hamlet-1
{
  "settings": {
    "number_of_shards" : 2,
    "number_of_replicas" : 0
  }
}

PUT hamlet-2
{
  "settings": {
    "number_of_shards" : 2,
    "number_of_replicas" : 0
  }
}
```

### Add some documents to `hamlet-1` by running the following _bulk command

```
PUT hamlet_1/_bulk
{"index":{"_index":"hamlet-1","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet-1","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet-1","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet-1","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}
```

### Add some documents to `hamlet-2` by running the following _bulk command

```
PUT hamlet-2/_bulk
{"index":{"_index":"hamlet-2","_id":4}}
{"line_number":"2.1.1","speaker":"LORD POLONIUS","text_entry":"Give him this money and these notes, Reynaldo."}
{"index":{"_index":"hamlet-2","_id":5}}
{"line_number":"2.1.2","speaker":"REYNALDO","text_entry":"I will, my lord."}
{"index":{"_index":"hamlet-2","_id":6}}
{"line_number":"2.1.3","speaker":"LORD POLONIUS","text_entry":"You shall do marvellous wisely, good Reynaldo,"}
{"index":{"_index":"hamlet-2","_id":7}}
{"line_number":"2.1.4","speaker":"LORD POLONIUS","text_entry":"Before you visit him, to make inquire"}
```

### Create the alias `hamlet` that maps both `hamlet-1` and `hamlet-2`

```
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "hamlet-*", "alias" : "hamlet" } }
    ]
}
```

### Verify that the documents grouped by `hamlet` are 8

```
GET hamlet/_count
```

### Configure `hamlet-1` to be the write index of the `hamlet` alias

```
POST /_aliases
{
  "actions" : [
        {
            "add" : {
                 "index" : "hamlet-1",
                 "alias" : "hamlet",
                 "is_write_index" : true
            }
        }
    ]
}
```

### Add a document to `hamlet`, so that the document (i) has id "8", (ii) has "_doc" type, (iii) has a field `text_entry` with value "With turbulent and dangerous lunacy?", (iv) has a field `line_number` with value "3.1.4", (v) has a field `speaker` with value "KING CLAUDIUS"

```
PUT /hamlet/_doc/8
{
    "text_entry": "With turbulent and dangerous lunacy?",
    "line_number": "3.1.4",
    "speaker": "KING CLAUDIUS"
}
```

### Create a script named `control_reindex_batch` and save it into the cluster state. The script checks whether a document has the field `reindexBatch`, and (i) in the affirmative case, it increments the field value by a script parameter named `increment`, (ii) otherwise, the script adds the field to the document setting its value to "1"

```
POST _scripts/control_reindex_batch
{
  "script": {
    "lang": "painless",
    "source": """
      if(ctx._source.increment != null) 
        { ctx._source.reindexBatch += params.increment; } 
      else { ctx._source.reindexBatch = 1; }"""
  }
}
```

### Create the index `hamlet-new` with two primary shard and no replicas

```
PUT hamlet-new
{
  "settings": {
    "number_of_shards" : 2,
    "number_of_replicas" : 0
  }
}
```

### Reindex `hamlet` into `hamlet-new`, while satisfying the following criteria: (i) disable any refresh of `hamlet-new` during the reindexing, (ii) apply the `control_reindex_batch` script with the `increment` parameter set to "1", (iii) reindex using two parallel slices

```
POST _reindex
{
  "source": {
    "index": "hamlet"
  },
  "dest": {
    "index": "hamlet-new"
  },
  "script": {
    "id": "control_reindex_batch",
    "params": {
      "increment": "1"
    }
  }
}
```

### In one request, add `hamlet-new` to the alias `hamlet` and delete the `hamlet-1` and `hamlet-2` indices

```
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "hamlet-new",
        "alias": "hamlet"
      }
    },
    { "remove_index": { "index": "hamlet-1" } },
    { "remove_index": { "index": "hamlet-2" } }
  ]
}
```

### Create an ingest pipeline named `split_act_scene_line`. The pipeline splits the value of `line_number` using the dots as a separator, and stores the split values into three new fields named `number_act`, `number_scene`, and `number_line`, respectively

```
PUT _ingest/pipeline/split_act_scene_line
{
  "processors" : [
    {
        "script": {
          "lang": "painless",
          "source": """
            String[] lineSplit = ctx['line_number'].splitOnToken(params['delimiter']);
            ctx['number_act'] = lineSplit[0];
            ctx['number_scene'] = lineSplit[1];
            ctx['number_line'] = lineSplit[2];
          """,
          "params": {
            "delimiter": "."
          }
        }
      }
  ]
}
```

```
PUT _ingest/pipeline/split_act_scene_line
{
  "processors": [
    {
      "dissect": {
        "field": "line_number",
        "pattern": "%{number_act}.%{number_scene}.%{number_line}"
      }
    }
  ]
}

```

To verify created pipeline, use: 
```
POST _ingest/pipeline/split_act_scene_line/_simulate
{
  "docs": [
    {
      "_source": {
        "line_number": "1.2.3"
      }
    }
  ]
}
```

### Test the ingest pipeline on the following document

```
POST _ingest/pipeline/split_act_scene_line/_simulate
{
  "docs" : [
    { 
      "_source": {
        "line_number": "1.2.3"
      }
    }
  ]
}
```

### Update all documents in `hamlet-new` by using the `split_act_scene_line` pipeline

```
POST hamlet-new/_update_by_query?pipeline=split_act_scene_line
```
