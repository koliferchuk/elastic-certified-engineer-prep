## Indexing Data

### Create the index template `hamlet_template`, so that the template (i) matches any index that starts by "hamlet_" or "hamlet-", (ii) allocates one primary shard and no replicas for each matching index

```
PUT _template/hamlet_template
{
  "index_patterns": ["hamlet_*", "hamlet-*"],
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```

### Create the indices `hamlet2` and `hamlet_test`

```
PUT hamlet2
PUT hamlet_test
```

### Verify that only `hamlet_test` applies the settings defined in `hamlet_template`

```
GET hamlet2/_settings
GET hamlet_test/_settings
```

### In one request, delete both `hamlet2` and `hamlet_test`

```
DELETE hamlet*
```

### Update `hamlet_template` by defining a mapping for the type "_doc", so that (i) the type has three fields, named `speaker`, `line_number`, and `text_entry`, (ii) `speaker` and `line_number` map to a unanalysed string, (iii) `text_entry` uses an "english" analyzer

```
PUT _template/hamlet_template
{
  "index_patterns": ["hamlet_*", "hamlet-*"],
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "speaker": {
        "type": "keyword"
      },
      "line_number": {
        "type": "keyword"
      },
      "text_entry": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}
```

### Create the index `hamlet-1` and add some documents by running the following _bulk command

```
PUT hamlet-1/_doc/_bulk
{"index":{"_index":"hamlet-1","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet-1","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet-1","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet-1","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}
```

### Verify that the mapping of `hamlet-1` is consistent with what defined in `hamlet_template`

```
GET hamlet-1/_mapping
```

### Update `hamlet_template` so as to reject any document having a field that is not defined in the mapping

```
PUT _template/hamlet_template
{
  "index_patterns": ["hamlet_*", "hamlet-*"],
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "speaker": {
        "type": "keyword"
      },
      "line_number": {
        "type": "keyword"
      },
      "text_entry": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}
```

### Verify that you cannot index the following document in `hamlet-2`

```
POST hamlet-2/_doc/
{
  "author": "Shakespeare"
}
```

### Update `hamlet_template` so as to enable dynamic mapping again

```
PUT _template/hamlet_template
{
  "index_patterns": ["hamlet_*", "hamlet-*"],
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "dynamic": true,
    "properties": {
      "speaker": {
        "type": "keyword"
      },
      "line_number": {
        "type": "keyword"
      },
      "text_entry": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}
```

### Update `hamlet_template` so as to (i) dynamically map to an integer any field that starts by "number_", (ii) dynamically map to unanalysed text any string field

```
PUT _template/hamlet_template
{
  "index_patterns": ["hamlet_*", "hamlet-*"],
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "dynamic_templates": [
      {
        "numbers": {
          "match_mapping_type": "*",
          "match":   "number_*",
          "mapping": {
            "type": "integer"
          }
        }
      },
      {
        "strings": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ],
    "properties": {
      "speaker": {
        "type": "keyword"
      },
      "line_number": {
        "type": "keyword"
      },
      "text_entry": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}
```

### Create the index `hamlet-3` and add a document by running the following command

```
POST hamlet-3/_doc/4
{
  "text_entry": "With turbulent and dangerous lunacy?",
  "line_number": "3.1.4",
  "number_act": "3",
  "speaker": "KING CLAUDIUS"
}
```

### Verify that the mapping of `hamlet-3` is consistent with what defined in `hamlet_template`

```
GET hamlet-3/_mapping
```
