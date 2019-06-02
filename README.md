## Elasticsearch Shingles Example

1. Make index

```
PUT /test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "analyzer_shingle": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "filter_shingle"
          ]
        }
      },
      "filter": {
        "filter_shingle": {
          "type": "shingle",
          "max_shingle_size": 2,
          "min_shingle_size": 2,
          "output_unigrams": "false"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "search_analyzer": "analyzer_shingle",
        "analyzer": "analyzer_shingle",
        "type": "text",
        "fielddata": true
      }
    }
  }
}
```

2. Add sample data
```
POST /test/_bulk?pretty
{ "index":{}}
{ "title":"Sample product title for shingles"}
{ "index":{}}
{ "title":"How To Clean A Microwave With Vinegar And Steam!"}
{ "index":{}}
{ "title":"How To Clean Grout With A Homemade Grout Cleaner"}
{ "index":{}}
{ "title":"How To do anything"}
{ "index":{}}
{ "title":"How To Make a Large Monogram Cutout The Easy Way"}
{ "index":{}}
{ "title":"18 Mouthwatering Breakfast Recipes to Try On Your Next Camping Trip"}
{ "index":{}}
{ "title":"25 Sewing Hacks You Won't Want to Forget"}
{ "index":{}}
{ "title":"9 Ways Apple Cider Vinegar Will Improve Your Life"}
{ "index":{}}
{ "title":"135 Easy Elf on the Shelf Ideas"}
{ "index":{}}
{ "title":"99 Things You Might be Thankful for about Your Husband"}
{ "index":{}}
{ "title":"The 50 Funniest Tumblr Posts Of All Time"}
{ "index":{}}
{ "title":"99 Of The Funniest Pinterest Pictures We've Ever Seen"}
{ "index":{}}
{ "title":"Stop Searching For A Magical Exercise Routine"}
{ "index":{}}
{ "title":"council welcomes ambulance levy decision"}
{ "index":{}}
{ "title":"council welcomes insurance breakthrough"}
{ "index":{}}
{ "title":"man to face court over alleged hijack"}
{ "index":{}}
{ "title":"man to face court over attempted armed robbery"}
{ "index":{}}
{ "title":"man who whacked thatcher gets 3 months jail"}

```

3. Query shingles
```
GET /test/_search
{
  "size" : 0,
  "query" : {
    "match_all" : {}
  },
  "aggs" : {
    "analyzer_shingle" : {
      "terms" : {
        "field" : "title",
        "size"  : 100  
      }
    }
  }
}
```

4. Kibana visualization (custom vega bar chart)
```
{
  "$schema": "https://vega.github.io/schema/vega/v4.json",
  "title": "Most frequent phrasses",
  "data": {
    "name": "table",
    "url": {
      "index": "test",
      "body": {
        "aggs": {
          "analyzer_shingle": {"terms": {"field": "title", "size": 20}}
        },
        "size": 0
      }
    },
    "format": {"property": "aggregations.analyzer_shingle.buckets"},
    "transform": [
      {
        "type": "window",
        "sort": {"field": "doc_count", "order": "descending"},
        "ops": ["row_number"],
        "as": ["rank"]
      }
    ]
  },
  "scales": [
    {
      "name": "xscale",
      "type": "linear",
      "domain": {"data": "table", "field": "doc_count"},
      "range": "width",
      "nice": true
    },
    {
      "name": "yscale",
      "type": "band",
      "domain": {
        "data": "table",
        "field": "key",
        "sort": {
          "op": "max",
          "field": "doc_count",
          "order": "descending"
        }
      },
      "range": "height",
      "padding": 0.1
    }
  ],
  "axes": [
    {"orient": "bottom", "scale": "xscale", "labelFontSize": 12},
    {"orient": "left", "scale": "yscale", "labelFontSize": 12}
  ],
  "marks": [
    {
      "type": "rect",
      "from": {"data": "table"},
      "encode": {
        "update": {
          "x": {"scale": "xscale", "value": 0},
          "x2": {"scale": "xscale", "field": "doc_count"},
          "y": {"scale": "yscale", "field": "key"},
          "height": {"scale": "yscale", "band": 1},
          "fill": {"value": "steelblue"}
        }
      }
    }
  ]
}
```
