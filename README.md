# How to start
```bash
docker-compose build
docker-compose up -d
```


```
[optional commands in ES docker container]
bin/elasticsearch-plugin install analysis-icu
bin/elasticsearch-plugin install analysis-kuromoji
```

check your browser
* Elasticsearch
  * http://localhost:9200
* Kibana
  * http://localhost:5601/app/home#/
  ![Screen Shot 2023-10-25 at 11 51 17](https://github.com/Baplisca/elastic-search-sandbox/assets/65814732/030a92e1-ce54-4655-8624-355e06b1bc1e)


# Test in Kibana Dev tool
sample query
```
POST _analyze
{
  "tokenizer": {
    "type": "kuromoji_tokenizer",
    "mode": "search"
  },
  "text": ["アブラマシマシ"]
}
```
result
```
{
  "tokens" : [
    {
      "token" : "アブラマシマシ",
      "start_offset" : 0,
      "end_offset" : 7,
      "type" : "word",
      "position" : 0
    }
  ]
}
```

sample query2
```
POST _analyze
{
  "tokenizer": {
    "type": "kuromoji_tokenizer",
    "mode": "extended"
  },
  "text": ["東京スカイツリー"]
}
```

result
```
{
  "tokens" : [
    {
      "token" : "東京",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "スカイ",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "ツリー",
      "start_offset" : 5,
      "end_offset" : 8,
      "type" : "word",
      "position" : 2
    }
  ]
}
```

sample query3
```
POST /_analyze
{
  "char_filter": ["icu_normalizer", "kuromoji_iteration_mark"],
  "tokenizer": {
    "type": "kuromoji_tokenizer",
    "mode": "search"
  },
  "filter": [
    "kuromoji_baseform",
    "kuromoji_part_of_speech",
    "ja_stop",
    "kuromoji_number",
    "kuromoji_stemmer",
    "lowercase"
  ],
  "text": "東京ｽｶｲサーバー"
}
```

result
```
{
  "tokens" : [
    {
      "token" : "東京",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "スカイ",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "サーバ",
      "start_offset" : 5,
      "end_offset" : 9,
      "type" : "word",
      "position" : 2
    }
  ]
}
```
![Screen Shot 2023-10-25 at 11 53 45](https://github.com/Baplisca/elastic-search-sandbox/assets/65814732/fcbf11a6-dd2a-4c81-b401-685ea5192c5a)

sample query4 (部分一致検索)
```
[If kuromoji index was created, delete it]
DELETE /kuromoji

PUT /kuromoji
{
  "settings":{
    "analysis": {
      "tokenizer": {
        "ja_kuromoji_tokenizer": {
          "type": "kuromoji_tokenizer",
          "mode": "extended"
        }
      },
      "analyzer": {
        "ja_kuromoji_analyzer": {
          "type": "custom",
          "tokenizer": "ja_kuromoji_tokenizer",
          "char_filter": ["icu_normalizer"],
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "ja_kuromoji_analyzer"
      }
    }
  }
}

[check mapping setting]
GET kuromoji/_mapping

POST kuromoji/_doc/1
{
  "text": "日本酒は水"
}

GET kuromoji/_search
{
  "query": {
    "match_phrase": {
      "text": "酒"
    }
  }
}
```

result
```
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "kuromoji",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "text" : "日本酒は水"
        }
      }
    ]
  }
}
```

check term_vectors
```
GET kuromoji/_doc/1/_termvectors
{
  "fields": ["text"]
}
```
result
```
#! Deprecation: [types removal] Specifying types in term vector requests is deprecated.
{
  "_index" : "kuromoji",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "took" : 1,
  "term_vectors" : {
    "text" : {
      "field_statistics" : {
        "sum_doc_freq" : 4,
        "doc_count" : 1,
        "sum_ttf" : 4
      },
      "terms" : {
        "は" : {
          "term_freq" : 1,
          "tokens" : [
            {
              "position" : 2,
              "start_offset" : 3,
              "end_offset" : 4
            }
          ]
        },
        "日本" : {
          "term_freq" : 1,
          "tokens" : [
            {
              "position" : 0,
              "start_offset" : 0,
              "end_offset" : 2
            }
          ]
        },
        "水" : {
          "term_freq" : 1,
          "tokens" : [
            {
              "position" : 3,
              "start_offset" : 4,
              "end_offset" : 5
            }
          ]
        },
        "酒" : {
          "term_freq" : 1,
          "tokens" : [
            {
              "position" : 1,
              "start_offset" : 2,
              "end_offset" : 3
            }
          ]
        }
      }
    }
  }
}
```

# Ref
* https://github.com/elastic/elasticsearch
* https://note.com/lizefield/n/necadb7abf225
* https://qiita.com/tkani/items/fb3aa2e61d60ad9dc56c
* https://zenn.dev/fujimotoshinji/scraps/4fb4616976ee00