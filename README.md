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
        }

For this example, explicit mappings are used which means the token on the lefthand side of => is replaced with the one on the right side. We will use equivalent synonyms later, which means the tokens provided are treated equivalently.

The synonym filter, it’s a bit special and may be surprising to many of us. In this example, even though the synonym_filter filter is put after the lowercase filter, the tokens returned by this filter are also passed to the lowercase filter and thus also get lowercased. Therefore, you don’t need to provide lowercase tokens in the synonym list or in the synonym file.

## search-time synonyms

The `search_analyzer` is specified for the name field explicitly. If it’s not specified, the same analyzer (`index_analyzer`) will be used for both indexing and searching.