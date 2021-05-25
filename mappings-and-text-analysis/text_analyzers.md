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

### Define a mapping for the default type "_doc" of `hamlet_1`, so that (i) the type has three fields, named `speaker`, `line_number`, and `text_entry`, (ii) `text_entry` is associated with the built-in "english" analyzer

```
PUT hamlet_1/_mapping
{
  "properties": {
    "speaker": {
      "type": "text"
    },
    "line_number": {
      "type": "text"
    },
    "text_entry": {
      "type": "text",
      "analyzer": "english"
    }
  }
}
```

### Add some documents to `hamlet_1` by running the following _bulk command

```
PUT hamlet_1/_bulk
{"index":{"_index":"hamlet_1","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet_1","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet_1","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet_1","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}
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

### Add to `hamlet_2` a custom analyzer named `shy_hamlet_analyzer`, which is composed of (i) a char filter to replace the characters "Hamlet" with "[censored]", (ii) a tokenizer to split tokens on whitespaces and columns, (iii) a token filter to ignore any token with less than 5 characters

```
PUT hamlet_2
{
  "settings": {
    "analysis": {
      "analyzer": {
        "shy_hamlet_analyzer": {
          "type":      "custom", 
          "tokenizer": "shy_hamlet_tokenizer",
          "char_filter": ["shy_hamlet_char_filter"],
          "filter": ["shy_hamlet_filter"]
        }
      },
      "tokenizer": {
        "shy_hamlet_tokenizer": {
          "type": "char_group",
          "tokenize_on_chars": [
            "whitespace",
            ":"
          ]
        }
      },
      "char_filter": {
        "shy_hamlet_char_filter": {
          "type": "mapping",
          "mappings": [
            "Hamlet => [censored]"
          ]
        }
      },
      "filter": {
        "shy_hamlet_filter": {
          "type": "length",
          "min": "5"
        }
      }
    }
  }
}
```

### Define a mapping for the default type "_doc" of `hamlet_2`, so that (i) the type has one field named `text_entry`, (ii) `text_entry` is associated with the `shy_hamlet_analyzer` created in the previous step

```
PUT hamlet_2/_mapping
{
  "properties": {
    "text_entry": {
      "type": "text",
      "analyzer": "shy_hamlet_analyzer"
    }
  }
}
```

### Reindex the `text_entry` field of `hamlet_1` into `hamlet_2`

```
POST _reindex
{
  "source": {
    "index": "hamlet_1",
    "_source": ["text_entry"]
  },
  "dest": {
    "index": "hamlet_2"
  }
}
```

### Verify that documents have been reindexed to `hamlet_2` as expected - e.g., by searching for "censored" into the `text_entry` field

```
GET hamlet_2/_search
{
  "query": {
    "match": {
      "text_entry": "[censored]"
    }
  }
}
```