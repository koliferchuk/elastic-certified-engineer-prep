## Aggregations

Run the next queries on the `kibana_sample_data_flights` index

### Create an aggregation named "max_distance" that calculates the maximum value of the `DistanceKilometers` field

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" : {
    "max_distance" : { "max" : { "field" : "DistanceKilometers" } }
  }
}
```

### Create an aggregation named "stats_flight_time" that computes stats over the value of the `FlightTimeMin` field

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" : {
    "stats_flight_time" : { "stats" : { "field" : "FlightTimeMin" } }
  }
}
```

### Create two aggregations, named "cardinality_origin_cities" and "cardinality_dest_cities", that calculate an (approximate) count of distinct values of the `OriginCityName` and `DestCityName` fields, respectively

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" :
    {
      "cardinality_origin_cities" : { "cardinality" : { "field" : "OriginCityName" } },
      "cardinality_dest_cities" : { "cardinality" : { "field" : "DestCityName" } }
    }
}
```

### Create an aggregation named "popular_origin_cities" that calculates the number of flights grouped by the `OriginCityName` field

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs": {
    "popular_origin_cities": {
      "terms" : { "field" : "OriginCityName" }
    }
  }
}
```

### As above, but return only 5 buckets and in descending order

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs": {
    "popular_origin_cities": {
      "terms" : { 
        "field" : "OriginCityName",
        "order" : { "_count" : "desc" },
        "size" : 5
      }
    }
  }
}
```

### Create an aggregation named "avg_price_histogram" that groups the documents based on their `AvgTicketPrice` field by intervals of 250

```
POST kibana_sample_data_flights/_search?size=0
{
    "aggs" : {
        "avg_price_histogram" : {
            "histogram" : {
                "field" : "AvgTicketPrice",
                "interval" : 250
            }
        }
    }
}
```

### Create an aggregation named "popular_carriers" that calculates the number of flights grouped by the `Carrier` field

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs": {
    "popular_carriers": {
      "terms" : { "field" : "Carrier" }
    }
  }
}
```

### Add a sub-aggregation to "popular_carriers", named "carrier_stats_delay", that computes stats over the value of the `FlightDelayMin` field for the related bucket of carriers

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs": {
    "popular_carriers": {
      "terms" : { "field" : "Carrier" },
      "aggs": {
        "carrier_stats_delay": {
          "stats": { "field" : "FlightDelayMin" }
        }
      }
    }
  }
}
```

### Add a second sub-aggregation to "popular_carriers", named "carrier_max_delay", that shows the flight having the maximum value of the `FlightDelayMin` field for the related bucket of carriers

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs": {
    "popular_carriers": {
      "terms" : { "field" : "Carrier" },
      "aggs": {
        "carrier_stats_delay": {
          "stats": { "field" : "FlightDelayMin" }
        },
        "carrier_max_delay": {
          "top_hits": {
              "sort": [
                  {
                    "FlightDelayMin": {
                        "order": "desc"
                    }
                  }
              ],
              "size" : 1
          }
        }
      }
    }
  }
}
```

### Use the `timestamp` field to create an aggregation named "flights_every_10_days" that groups the number of flights by an interval of 10 days

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" : {
    "flights_every_10_days" : {
      "date_histogram" : {
        "field" : "timestamp",
        "fixed_interval" : "10d"
      }
    }
  }
}
```

### Use the `timestamp` field to create an aggregation named "flights_by_day" that groups  the number of flights by day

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" : {
    "flights_by_day" : {
      "date_histogram" : {
        "field" : "timestamp",
        "calendar_interval" : "day"
      }
    }
  }
}
```

### Add a sub-aggregation to “flights_by_day”, named “destinations_by_day”, that groups the day buckets by the value of the `DestCityName` field

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" : {
    "flights_by_day" : {
      "date_histogram" : {
        "field" : "timestamp",
        "calendar_interval" : "day"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          }
        }
      }
    }
  }
}
```

### Add a sub-aggregation to the sub-aggregation "destinations_by_day", named "popular_destinations_by_day", that returns the 3 most popular documents for each bucket (i.e., ordered by their score)

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" : {
    "flights_by_day" : {
      "date_histogram" : {
        "field" : "timestamp",
        "calendar_interval" : "day"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          },
          "aggs": {
            "popular_destinations_by_day": {
              "top_hits": {"size": 3}
            }
          }
        }
      }
    }
  }
}
```

### Update “popular_destinations_by_day” to display only the `DestCityName` field in for each top hit object

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" : {
    "flights_by_day" : {
      "date_histogram" : {
        "field" : "timestamp",
        "calendar_interval" : "day"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          },
          "aggs": {
            "popular_destinations_by_day": {
              "top_hits": {"size": 3, "_source": ["DestCityName"]}
            }
          }
        }
      }
    }
  }
}
```

### Remove the "popular_destinations_by_day” sub-sub-aggregation from “flights_by_day”

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" : {
    "flights_by_day" : {
      "date_histogram" : {
        "field" : "timestamp",
        "calendar_interval" : "day"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          }
        }
      }
    }
  }
}
```

### Add a pipeline aggregation to "flights_by_day", named "most_popular_destination_of_the_day", that identifies the "destinations_by_day” bucket with the most documents for each day

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" : {
    "flights_by_day" : {
      "date_histogram" : {
        "field" : "timestamp",
        "calendar_interval" : "day"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          }
        },
        "most_popular_destination_of_the_day": {
          "max_bucket": {
            "buckets_path": "destinations_by_day._count"
          }
        }
      }
    }
  }
}
```

### Add a pipeline aggregation named "day_with_most_flights" that identifies the “flights_by_day” bucket with the most documents

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" : {
    "flights_by_day" : {
      "date_histogram" : {
        "field" : "timestamp",
        "calendar_interval" : "day"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          }
        },
        "most_popular_destination_of_the_day": {
          "max_bucket": {
            "buckets_path": "destinations_by_day._count"
          }
        }
      }
    },
    "day_with_most_flights": {
      "max_bucket": {
        "buckets_path": "flights_by_day._count"
      }
    }
  }
}
```

### Add a pipeline aggregation named "day_with_the_most_popular_destination_over_all_days" that identifies the “flights_by_day” bucket with the largest “most_popular_destination_of_the_day” value

```
POST kibana_sample_data_flights/_search?size=0
{
  "aggs" : {
    "flights_by_day" : {
      "date_histogram" : {
        "field" : "timestamp",
        "calendar_interval" : "day"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          }
        },
        "most_popular_destination_of_the_day": {
          "max_bucket": {
            "buckets_path": "destinations_by_day._count"
          }
        }
      }
    },
    "day_with_most_flights": {
      "max_bucket": {
        "buckets_path": "flights_by_day._count"
      }
    },
    "day_with_the_most_popular_destination_over_all_days": {
      "max_bucket": {
        "buckets_path": "flights_by_day>most_popular_destination_of_the_day"
      }
    }
  }
}
```
