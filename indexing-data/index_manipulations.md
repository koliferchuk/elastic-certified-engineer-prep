# Indexing Data

###  Create the index `hamlet-raw` with one primary shard and four replicas

```
PUT hamlet-raw
{
  "settings" : {
        "number_of_shards" : 1,
        "number_of_replicas" : 4
    }
}
```

### Add a document to `hamlet-raw`, so that the document (i) has id "1", (ii) has default type, (iii) has a field named `line` with value "To be, or not to be: that is the question"

```
PUT /hamlet-raw/_doc/1
{
  "line": "To be, or not to be: that is the question"
}
```

### Update the document with id "1" by adding a field named `line_number` with value "3.1.64"

```
POST hamlet-raw/_update/1
{
    "doc" : {
        "line_number": "3.1.64"
    }
}
```

### Add a new document to `hamlet-raw`, so that the document (i) has the id automatically assigned by Elasticsearch, (ii) has default type, (iii) has a field named `text_entry` with value "Whether tis nobler in the mind to suffer", (iv) has a field named `line_number` with value "3.1.66"


```
POST hamlet-raw/_doc/
{
    "text_entry" : "Whether tis nobler in the mind to suffer",
    "line_number" : "3.1.66"
}
```

### Update the last document by setting the value of `line_number` to "3.1.65"

```
POST hamlet-raw/_update/GAQ1H3kBrPPqJuURrndm
{
    "doc" : {
        "line_number": "3.1.65"
    }
}
```

### In one request, update all documents in `hamlet-raw` by adding a new field named `speaker` with value "Hamlet"

```
POST _bulk
{ "update" : {"_id" : "1", "_index" : "hamlet-raw", "retry_on_conflict" : 3} }
{ "doc" : {"speaker" : "Hamlet"} }
{ "update" : {"_id" : "GAQ1H3kBrPPqJuURrndm", "_index" : "hamlet-raw", "retry_on_conflict" : 3} }
{ "doc" : {"speaker" : "Hamlet"} }
```

### Update the document with id "1" by renaming the field `line` into `text_entry`

```
POST _bulk # Is bulk okay to use here?
{ "update" : {"_id" : "1", "_index" : "hamlet-raw", "retry_on_conflict" : 3} }
{"script" : "ctx._source.text_entry = ctx._source.line"}
{ "update" : {"_id" : "1", "_index" : "hamlet-raw", "retry_on_conflict" : 3} }
{"script" : "ctx._source.remove('line')"}
```

```
POST hamlet-raw/_update/1
{
  "script" : "ctx._source.text_entry = ctx._source.line"
}

POST hamlet-raw/_update/1
{
  "script" : "ctx._source.remove('text_entry')"
}
```

### Create the index `hamlet` and add some documents by running the following _bulk command

```
PUT hamlet
```

```
PUT hamlet/_bulk
{"index":{"_index":"hamlet","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}
{"index":{"_index":"hamlet","_id":4}}
{"line_number":"1.2.2","speaker":"KING CLAUDIUS","text_entry":"The memory be green, and that it us befitted"}
{"index":{"_index":"hamlet","_id":5}}
{"line_number":"1.3.1","speaker":"LAERTES","text_entry":"My necessaries are embarkd: farewell:"}
{"index":{"_index":"hamlet","_id":6}}
{"line_number":"1.3.4","speaker":"LAERTES","text_entry":"But let me hear from you."}
{"index":{"_index":"hamlet","_id":7}}
{"line_number":"1.3.5","speaker":"OPHELIA","text_entry":"Do you doubt that?"}
{"index":{"_index":"hamlet","_id":8}}
{"line_number":"1.4.1","speaker":"HAMLET","text_entry":"The air bites shrewdly; it is very cold."}
{"index":{"_index":"hamlet","_id":9}}
{"line_number":"1.4.2","speaker":"HORATIO","text_entry":"It is a nipping and an eager air."}
{"index":{"_index":"hamlet","_id":10}}
{"line_number":"1.4.3","speaker":"HAMLET","text_entry":"What hour now?"}
{"index":{"_index":"hamlet","_id":11}}
{"line_number":"1.5.2","speaker":"Ghost","text_entry":"Mark me."}
{"index":{"_index":"hamlet","_id":12}}
{"line_number":"1.5.3","speaker":"HAMLET","text_entry":"I will."}
```

### Create a script named `set_is_hamlet` and save it into the cluster state. The script (i) adds a field named `is_hamlet` to each document, (ii) sets the field to "true" if the document has `speaker` equals to "HAMLET", (iii) sets the field to "false" otherwise

```
POST _scripts/set_is_hamlet
{
  "script": {
    "lang": "painless",
    "source": "ctx._source.is_hamlet = (ctx._source.speaker =='HAMLET')"
  }
}
```

### Update all documents in `hamlet` by running the `set_is_hamlet` script

```
POST hamlet/_update_by_query
{
  "script" : {
    "id": "set_is_hamlet"
  },
  "query": {
    "match_all": {}
  }
}
```

### Remove from `hamlet` the documents that have either "KING CLAUDIUS" or "LAERTES" as the value of `speaker`

```
POST hamlet/_delete_by_query
{
  "query": { 
    "bool": {
      "should" : [
        { "term" : { "speaker.keyword" : "KING CLAUDIUS" } },
        { "term" : { "speaker.keyword" : "LAERTES" } }
      ]
    }
  }
}
```

### Delete all documents that start by "ham"

```
DELETE ham*  # Are here indicies meant?
```
