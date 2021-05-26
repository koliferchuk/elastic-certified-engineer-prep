## Mappings and Text Analysis

### Create the index `hamlet_1` with one primary shard and no replicas

```
PUT hamlet_1
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 1
  }
}
```

### Add some documents to `hamlet_1` by running the following _bulk command

```
PUT hamlet_1/_bulk
{"index":{"_index":"hamlet_1","_id":"C0","routing":"abc"}}
{"name":"HAMLET","relationship":[{"name":"HORATIO","type":"friend"},{"name":"GERTRUDE","type":"mother"}]}
{"index":{"_index":"hamlet_1","_id":"C1","routing":"abc"}}
{"name":"KING CLAUDIUS","relationship":[{"name":"HAMLET","type":"nephew"}]}
```

### Verify that the items of the `relationship` array cannot be searched independently - e.g., searching for a friend named Gertrude will return 1 hit

```
GET hamlet_1/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "relationship.name": "gertrude" } },
        { "match": { "relationship.type": "friend" } }
      ]
    }
  }
}
```

### Create the index `hamlet_2` with one primary shard and no replicas

```
PUT hamlet_2
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```

### Define a mapping for the default type "_doc" of `hamlet_2`, so that the inner objects of the `relationship` field (i) can be searched independently, (ii) have only unanalyzed fields

```
POST hamlet_2/_mapping
{
  "properties": {
    "relationship": {
      "type": "nested"
    }
  },
  "dynamic_templates": [
    {
      "relationship_unanalyzed": {
        "path_match":   "relationship.*",
        "mapping": {
          "type": "keyword"
        }
      }
    }
  ]
}
```

### Reindex `hamlet_1` to `hamlet_2`

```
POST _reindex
{
  "source": {"index": "hamlet_1"},
  "dest": {"index": "hamlet_2"}
}
```

### Verify that the items of the `relationship` array can now be searched independently - e.g., searching for a friend named Gertrude will return no hits

```
GET hamlet_2/_search
{
  "query": {
    "nested": {
      "path": "relationship",
      "query": {
        "bool": {
          "must": [
            { "match": { "relationship.name": "GERTRUDE" } },
            { "match": { "relationship.type": "friend" } }
          ]
        }
      }
    }
  }
}
```

### Add more documents to `hamlet_2` by running the following _bulk command

```
PUT hamlet_2/_bulk
{"index":{"_index":"hamlet_2","_id":"L0","routing":"abc"}}
{"line_number":"1.4.1","speaker":"HAMLET","text_entry":"The air bites shrewdly; it is very cold."}
{"index":{"_index":"hamlet_2","_id":"L1","routing":"abc"}}
{"line_number":"1.4.2","speaker":"HORATIO","text_entry":"It is a nipping and an eager air."}
{"index":{"_index":"hamlet_2","_id":"L2","routing":"abc"}}
{"line_number":"1.4.3","speaker":"HAMLET","text_entry":"What hour now?"}
```

So far, we have indexed two kinds of documents, either related to characters (ids: C0, C1) or to a line of the dialogue (ids: L0, L1, L2). Notice that a character can have many lines. We will model this one-to-many relation in the next step

### Create the index `hamlet_3` with only one primary shard and no replicas

```
PUT hamlet_3
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```

### Copy the mapping of `hamlet_2` into `hamlet_3`, but also add a join field to define a relation between a `character` (the parent) and a `line` (the child). The name of such field is "character_or_line"

```
POST hamlet_3/_mapping
{
  "properties": {
    "relationship": {
      "type": "nested"
    },
    "character_or_line": {
      "type": "join",
      "relations": {
        "character": "line"
      }
    }
  },
  "dynamic_templates": [
    {
      "relationship_unanalyzed": {
        "path_match":   "relationship.*",
        "mapping": {
          "type": "keyword"
        }
      }
    }
  ]
}
```

### Reindex `hamlet_2` to `hamlet_3`

```
POST _reindex
{
  "source": {"index": "hamlet_2"},
  "dest": {"index": "hamlet_3"}
}
```

### Update the document with id `C0` (i.e., the character document of Hamlet) by adding the field `character_or_line` and setting its `character_or_line.name` value to "character"

```
POST hamlet_3/_update/C0
{
  "doc" : {
      "character_or_line" : {
        "name": "character"
      }
  }
}
```

### Create a script named `init_lines` and save it into the cluster state. The script (i) has a parameter named `characterId`, (ii) adds the field `character_or_line` to the document, (iii) sets the value of `character_or_line.name` to "line" , (iv) sets the value of `character_or_line.parent` to the value of the `characterId` parameter

```
POST _scripts/init_lines
{
  "script": {
    "lang": "painless",
    "source": """
      ctx._source.character_or_line = ['name':'line','parent':params.characterId];
    """
  }
}
```

### Update the documents in `hamlet_3` that have "HAMLET" as a `speaker`, by running the `init_lines` script with `characterId` set to "C0"

```
POST hamlet_3/_update_by_query
{
  "script": {
    "id": "init_lines",
    "params": {
      "characterId": "C0"
    }
  },
  "query": { 
    "term": {
      "speaker.keyword": "HAMLET"
    }
  }
}
```

### Verify that the last operation was successful by running the query below

```
GET hamlet_3/_search
{
  "query": {
    "has_parent": {
      "parent_type": "character",
      "query": {
        "match": {
          "name": "HAMLET"
        }
      }
    }
  }
}
```
