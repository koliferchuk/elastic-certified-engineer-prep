## Queries

Run the next queries on the `kibana_sample_data_logs` index


### Search for documents with the `message` field containing the string "Firefox"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "match": {
      "message": "Firefox"
    }
  }
}
```

### As above, but return up to 50 results

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "match": {
      "message": "Firefox"
    }
  },
  "size": 50
}
```

### As above, but return up to 50 results with an offset of 50 from the first

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "match": {
      "message": "Firefox"
    }
  },
  "from": 50,
  "size": 50
}
```

### Search for documents with the `message` field containing the strings "Firefox" or "Kibana"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "match": {
      "message": {
        "query": "Firefox Kibana",
        "operator" : "or"           // operator 'or' is default
      }
    }
  }
}
```

### Search for documents with the `message` field containing both the strings "Firefox" and "Kibana"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "match": {
      "message": {
        "query": "Firefox Kibana",
        "operator" : "and" 
      }
    }
  }
}
```

### Search for documents with the `message` field containing at least two of the following strings: "Firefox", "Kibana", "159.64.35.129"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "match": {
      "message": {
        "query": "Firefox Kibana 159.64.35.129",
        "operator" : "or",
        "minimum_should_match": 2
      }
    }
  }
}
```

### As above, but also return the highlights for the `message` field

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "match": {
      "message": {
        "query": "Firefox Kibana 159.64.35.129",
        "operator" : "or",
        "minimum_should_match": 2
      }
    }
  },
  "highlight" : {
    "fields" : {
        "message" : {}
    }
  }
}
```

### As above, but also wrap the highlights in "{{" and "}}"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "match": {
      "message": {
        "query": "Firefox Kibana 159.64.35.129",
        "operator" : "or",
        "minimum_should_match": 2
      }
    }
  },
  "highlight" : {
    "pre_tags" : ["{{"],
    "post_tags" : ["}}"],
    "fields" : {
        "message" : {}
    }
  }
}
```

### Search for documents with the `message` field containing the phrase "HTTP/1.1 200 51"

```
GET kibana_sample_data_logs/_search
{
  "query": {
        "match_phrase" : {
            "message" : "HTTP/1.1 200 51"
        }
    }
}
```

### As above, but sort the results by the `machine.os` field, in descending order

```
GET kibana_sample_data_logs/_search
{
  "query": {
        "match_phrase" : {
            "message" : "HTTP/1.1 200 51"
        }
    },
    "sort": [
      {
        "machine.os.keyword": {
          "order": "desc"
        }
      }
    ]
}
```

### As above, but also sort the results by the `timestamp` field, in ascending order

```
GET kibana_sample_data_logs/_search
{
  "query": {
        "match_phrase" : {
            "message" : "HTTP/1.1 200 51"
        }
    },
    "sort": [
      {
        "machine.os.keyword": {
          "order": "desc"
        }
      },
      {
        "timestamp": {
          "order": "asc"
        }
      }
    ]
}
```

Run the next queries on the `kibana_sample_data_ecommerce` index

### Search for documents with the `day_of_week` field containing the string "Monday"

```
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "term": {
      "day_of_week": "Monday"
    }
  }
}
```

### As above, but sort the results by the `products.base_price` field in descending order, picking the lowest value of the array

```
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "term": {
      "day_of_week": "Monday"
    }
  },
  "sort": [
    {
      "products.base_price": {
        "order": "desc",
        "mode" : "min"
      }
    }
  ]
}
```

Run the next queries on the `kibana_sample_data_logs` index

### Filter documents with the `response` field greater or equal to 400 and less than 500

```
GET kibana_sample_data_logs/_search
{
    "query": {
        "range" : {
            "response.keyword": {
                "gte" : 400,
                "lt" : 500
            }
        }
    }
}
```

### As above, but add a second filter for documents with the `referer` field matching the string "http://twitter.com/success/guion-bluford"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "response.keyword": {
                  "gte" : 400,
                  "lt" : 500
              }
          }
        },
        {
          "term": {
            "referer": "http://twitter.com/success/guion-bluford"
          }
        }
      ]
    }
  }
}
```

### Filter documents with the `referer` field that starts by "http://twitter.com/success"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "prefix" : { "referer" : "http://twitter.com/success" }
  }
}
```

### Filter documents with the `request` field that starts by "/people"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "match_bool_prefix" : { "request": "/people" }
  }
}
```

### Filter documents with the `memory` field containing any indexed value

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "exists": { "field": "memory" }
  }
}
```

### (opposite of above) Filter documents with the `memory` field not containing any indexed value

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "must_not":
        {
          "exists": { "field": "memory" }
        }
    }
  }
}
```

### Search for documents with the `agent` field containing the string "Windows" and the `url` field containing the string "name:john"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match" : {
            "agent" : {
              "query" : "Windows"
            }
          }
        },
        {
          "match_phrase" : {
            "url" : "name:john"
          }
        }
      ]
    }
  }
}
```

### As above, but also filter documents with the `phpmemory` field containing any indexed value

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match" : {
            "agent" : {
              "query" : "Windows"
            }
          }
        },
        {
          "match_phrase" : {
            "url" : "name:john"
          }
        }
      ],
      "filter": [
        { 
          "exists": { "field": "phpmemory" }
        }
      ]
    }
  }
}
```

### Search for documents that have either the `response` field greater or equal to 400 or the `tags` field having the string "error"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "range": {
            "response": {
              "gte": 400
            }
          }
        },
        {
          "term": {
            "tags": {
              "value": "error"
            }
          }
        }
      ],
      "minimum_should_match" : 1
    }
  }
}
```

### Search for documents with the `tags` field that does not contain any of the following strings: "warning", "error", "info"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "must_not": [
        { 
          "term": {
            "tags": { "value": "warning" }
          }
        },
        { 
          "term": {
            "tags": { "value": "error" }
          }
        },
        { 
          "term": {
            "tags": { "value": "info" }
          }
        }
      ]
    }
  }
}
```

### Filter documents with the `timestamp` field containing a date between today and one week ago

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "range": {
      "timestamp": {
        "lt": "now/d",
        "gte": "now-1w/d"
      }
    }
  }
}
```

### Filter documents with the `clientip` field having an IP address in the range "1.0.0.0"-"1.255.255.255"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "range": {
      "clientip": {
        "lte": "1.255.255.255",
        "gte": "1.0.0.0"
      }
    }
  }
}
```

Run the next queries on the `kibana_sample_data_flights` index

### Filter documents with either the `OriginCityName` or the `DestCityName` fields matching the string "Sydney"

```
GET kibana_sample_data_flights/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "OriginCityName": {
              "value": "Sydney"
            }
          }
        },
        {
          "term": {
            "DestCityName": {
              "value": "Sydney"
            }
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}
```

### As above, but allow inexact fuzzy matching, with a maximum allowed "Levenshtein Edit Distance" set to 2. Test that the query strings "Sydney", "Sidney" and "Sidnei" always return the same number of results

```
GET kibana_sample_data_flights/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "fuzzy": {
            "OriginCityName": {
              "value": "Sidnei",
              "fuzziness": 2
            }
          }
        },
        {
          "fuzzy": {
            "DestCityName": {
              "value": "Sidnei",
              "fuzziness": 2
            }
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}
```

Search for all documents in all indices

```
GET _all/_search
{
  "query": {
    "match_all": {}
  }
}
```

### As above, but use the scroll API to return the first 100 results while keeping the search context alive for 2 minutes

```
GET _all/_search?scroll=2m
{
  "query": {
    "match_all": {}
  },
  "size": 100
}
```

### Use the scroll id included in the response to the previous query and retrieve the next batch of results

```
GET /_search/scroll
{
    "scroll" : "1m", 
    "scroll_id" : "DnF1ZXJ5VGhlbkZldGNoCAAAAAAAAYl7Fjd2UzRiRW80U1V5NHQzNWdwWmZpY1EAAAAAAAGJfBY3dlM0YkVvNFNVeTR0MzVncFpmaWNRAAAAAAABiXoWN3ZTNGJFbzRTVXk0dDM1Z3BaZmljUQAAAAAAAYl5Fjd2UzRiRW80U1V5NHQzNWdwWmZpY1EAAAAAAAGJfRY3dlM0YkVvNFNVeTR0MzVncFpmaWNRAAAAAAABiYAWN3ZTNGJFbzRTVXk0dDM1Z3BaZmljUQAAAAAAAYl-Fjd2UzRiRW80U1V5NHQzNWdwWmZpY1EAAAAAAAGJfxY3dlM0YkVvNFNVeTR0MzVncFpmaWNR" 
}
```

Run the next queries on the `kibana_sample_data_logs` index

### Filter documents with (i) the `response` field greater or equal to 400 and less than 500, and (ii) the `tags` field containing the value "security"

```
GET kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "response": {
              "gte": 400,
              "lt": 500
            }
          }
        },
        {
          "term": {
            "tags": {
              "value": "security"
            }
          }
        }
      ]
    }
  }
}
```

### Create a search template for the above query, so that the template (i) is named "with_response_and_tag", (ii) has a parameter "with_min_response" to represent the lower bound of the `response` field, (iii) has a parameter "with_max_response" to represent the upper bound of the `response` field, (iv) has a parameter "with_tag" to represent a possible value of the `tags` field

```
POST _scripts/with_response_and_tag
{
  "script": {
    "lang": "mustache",
    "source": {
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "response": {
                  "gte": "{{with_min_response}}",
                  "lt": "{{with_max_response}}"
                }
              }
            },
            {
              "term": {
                "tags": {
                  "value": "{{with_tag}}"
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

### Test the "with_response_and_tag" search template by setting the parameters as follows: (i) "with_min_response": 400, (ii) "with_max_response": 500 (iii) "with_tag": "security"

```
GET _search/template
{
    "id": "with_response_and_tag", 
    "params": {
        "with_min_response": 400,
        "with_max_response": 500,
        "with_tag": "security"
    }
}
```

### Update the "with_response_and_tag" search template, so that (i) if the "with_max_response" parameter is not set, then don't set an upper bound to the `response` value, and (ii) if the "with_tag" parameter is not set, then do not apply that second filter at all

```
POST _scripts/with_response_and_tag
{
  "script": {
    "lang": "mustache",
    "source": 
      """{
        "query": {
          "bool": {
            "must": [
              {
                "range": {
                  "response": {
                    "gte": "{{with_min_response}}"
                    {{#with_max_response}},"lt": "{{with_max_response}}"{{/with_max_response}}
                  }
                }
              }
              {{#with_tag}},
              {
                "term": {
                  "tags": {
                    "value": "{{with_tag}}"
                  }
                }
              }{{/with_tag}}
            ]
          }
        }
      }"""
  }
}
```

### Test the "with_response_and_tag" search template by setting only the "with_min_response" parameter to 500

```
GET _search/template
{
    "id": "with_response_and_tag", 
    "params": {
        "with_min_response": 500
    }
}
```

### Test the "with_response_and_tag" search template by setting the parameters as follows: (i) "with_min_response": 500, (ii) "with_tag": "security"

```
GET _search/template
{
    "id": "with_response_and_tag", 
    "params": {
        "with_min_response": 500,
        "with_tag": "security"
    }
}
```

Run the next queries on the `kibana_sample_data_flights` index

### Filter documents with the `OriginWeather` and `DestWeather` fields set to the same value

```
GET kibana_sample_data_flights/_search
{
    "query": {
        "bool" : {
            "filter" : {
                "script" : {
                    "script" : {
                        "source": "doc['OriginWeather'].value == doc['DestWeather'].value",
                        "lang": "painless"
                     }
                }
            }
        }
    }
}
```
