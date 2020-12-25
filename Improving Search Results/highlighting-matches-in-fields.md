# Highlighting matches in fields

## Adding a test document

```
PUT /highlighting/_doc/1
{
  "description": "Let me tell you a story about Elasticsearch. It's a full-text search engine that is built on Apache Lucene. It's really easy to use, but also packs lots of advanced features that you can use to tweak its searching capabilities. Lots of well-known and established companies use Elasticsearch, and so should you!"
}
```

## Highlighting matches within the `description` field

```
GET /highlighting/_search
{
  "_source": false,
  "query": {
    "match": { "description": "Elasticsearch story" }
  },
  "highlight": {
    "fields": {
      "description" : {}
    }
  }
}
```

## Specifying a custom tag

```
GET /highlighting/_search
{
  "_source": false,
  "query": {
    "match": { "description": "Elasticsearch story" }
  },
  "highlight": {
    "pre_tags": [ "<strong>" ],
    "post_tags": [ "</strong>" ],
    "fields": {
      "description" : {}
    }
  }
}
```

## Highlighting matching part of words
We need to use the `ngram` or `edge_ngram` tokenizer in order to highlight part of word.

### Mapping
Below mapping is for type-ahead searches (prefix searches).
```
PUT /products
{
  "settings": {
    "index.max_ngram_diff": 14,
    "analysis": {
      "tokenizer": {
          "ngram_tokenizer": {
            "type": "edge_ngram",
            "min_gram": "3",
            "max_gram": "15",
            "token_chars": [ "letter", "digit" ]
          }
      },
      "analyzer": {
        "index_ngram_analyzer": {
          "type": "custom",
          "tokenizer": "ngram_tokenizer",
          "filter": [ "lowercase" ]
        },
        "search_term_analyzer": {
          "type": "custom",
          "tokenizer": "keyword",
          "filter": "lowercase" 
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name" : {
        "type" : "text",
        "analyzer": "index_ngram_analyzer",
        "search_analyzer": "search_term_analyzer",
        "term_vector":"with_positions_offsets",
        "fields" : {
          "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
          }
        }
      }
    }
  }
}
```

### Search query
```
GET products/_search
{
  "_source": "name", 
  "query": {
    "match": {
      "name": "kiw"
    }
  },
  "highlight": {
    "pre_tags": [ "<strong>" ],
    "post_tags": [ "</strong>" ],
    "fields": {
      "name" : {}
    }
  }
}
```