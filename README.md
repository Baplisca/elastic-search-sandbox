# How to start
```bash
docker-compose build
docker-compose up -d
```

[optional in ES docker container]
```
bin/elasticsearch-plugin install analysis-icu
bin/elasticsearch-plugin install analysis-kuromoji
```

check
* Elasticsearch
  * http://localhost:9200
* Kibana
  * http://localhost:5601/app/home#/

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
PUT kuromoji_tokenizer
{
  "settings": {
    "index": {
      "analysis": {
        "tokenizer": {
          "kuromoji_user_dict": {
            "type": "kuromoji_tokenizer",
            "mode": "extended"
          }
        },
        "analyzer": {
          "kuromoji_normalize": {
            "type": "custom",
            "char_filter": ["icu_normalizer"],
            "tokenizer": "kuromoji_user_dict",
            "filter": [
              "kuromoji_baseform",
              "kuromoji_part_of_speech",
              "cjk_width",
              "ja_stop",
              "kuromoji_stemmer",
              "lowercase"
            ]
          }
        }
      }
    }
  }
}

GET kuromoji_tokenizer/_analyze
{
  "analyzer": "kuromoji_normalize",
  "text": "東京ｽｶｲツリー"
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

# Ref
* https://github.com/elastic/elasticsearch
* https://note.com/lizefield/n/necadb7abf225
* https://qiita.com/tkani/items/fb3aa2e61d60ad9dc56c
* https://zenn.dev/fujimotoshinji/scraps/4fb4616976ee00