# Authors Synonyms Logic
## _Synonyms Filter_ (Elasticsearch)

[Solr synoynms](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-tokenfilter.html#_solr_synonyms) are used in this filter.

        "filter": {
          "synonym_filter": {
            "type": "synonym",
            "synonyms": [
              "PS => PlayStation",
              "Play Station => PlayStation"
            ]
          }

In this example, explicit mappings are used which means the token on the left side of => is replaced with the one on the right. We will use equivalent synonyms later, which means the tokens provided are treated equivalently.

The synonym filter, is a bit special and may be surprising to many of us. In this example, even though the synonym_filter filter is put after the lowercase filter, the tokens returned by this filter are also passed to the lowercase filter and thus also get lowercased. Therefore, you don’t need to provide lowercase tokens in the synonym list or in the synonym file.

## index-time synonyms
```
"Greg Maxwell,gmaxwell,Gregory Maxwell,G. Maxwell => AUTHOR_ID_1",
"cdecker,Christian Decker => AUTHOR_ID_2"
```
Here all the phrases that are part of one row are **contracted** into the token `AUTHOR_ID_1`.
This token is stored in the inverted index which is then use to perform the search. So, if you search for `Greg Maxwell` or `Gregory Maxwell` because it is contracted into the same token, the resultant documents are the same. This usecase works well if only search is needed, but if we need to remap the `AUTHOR_ID_1` to its original tokens. We need something known as **search-time synonyms**.

## search-time synonyms

The `search_analyzer` is specified for the name field explicitly. If it’s not specified, the same analyzer (`index_analyzer`) will be used for both indexing and searching.
```
"Greg Maxwell => Greg Maxwell,AUTHOR_ID_1",
"gmaxwell => gmaxwell,AUTHOR_ID_1",
"Gregory Maxwell => Gregory Maxwell,AUTHOR_ID_1",
"G. Maxwell => G. Maxwell,AUTHOR_ID_1",
"cdecker => cdecker,AUTHOR_ID_2",
"Christian Decker => Christian Decker,AUTHOR_ID_2"
```
Using the above synonyms rule, we **expand** the `AUTHOR_ID_1` to its original phrases. These can be then used in usecases like aggregations where we need count of documents where a particular author appears.

## Debugging
We can use the `_analyze` endpoint to debug what tokens are emitted by each analyzer.
```sh
GET bitcoin-search-index-revamped/_analyze
{
  "text": "Gregory Maxwell",
  "analyzer": "analyzer_search",
  "explain": true
}
```
The output of the above in `Dev Tools` is as follows:
```sh
{
  "detail": {
    "custom_analyzer": true,
    "charfilters": [],
    "tokenizer": {
      "name": "iplexus_tokenizer",
      "tokens": [
        {
          "token": "Gregory",
          "start_offset": 0,
          "end_offset": 7,
          "type": "word",
          "position": 0,
          "bytes": "[47 72 65 67 6f 72 79]",
          "positionLength": 1,
          "termFrequency": 1
        },
        {
          "token": "Maxwell",
          "start_offset": 8,
          "end_offset": 15,
          "type": "word",
          "position": 1,
          "bytes": "[4d 61 78 77 65 6c 6c]",
          "positionLength": 1,
          "termFrequency": 1
        }
      ]
    },
    "tokenfilters": [
      {
        "name": "lowercase",
        "tokens": [
          {
            "token": "gregory",
            "start_offset": 0,
            "end_offset": 7,
            "type": "word",
            "position": 0,
            "bytes": "[67 72 65 67 6f 72 79]",
            "positionLength": 1,
            "termFrequency": 1
          },
          {
            "token": "maxwell",
            "start_offset": 8,
            "end_offset": 15,
            "type": "word",
            "position": 1,
            "bytes": "[6d 61 78 77 65 6c 6c]",
            "positionLength": 1,
            "termFrequency": 1
          }
        ]
      },
      {
        "name": "synonym_rule",
        "tokens": [
          {
            "token": "author_id_1",
            "start_offset": 0,
            "end_offset": 15,
            "type": "SYNONYM",
            "position": 0,
            "bytes": "[61 75 74 68 6f 72 5f 69 64 5f 31]",
            "positionLength": 1,
            "termFrequency": 1
          }
        ]
      }
    ]
  }
}
```