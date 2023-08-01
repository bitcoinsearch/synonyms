# Authors Synonyms Logic

## Steps to implement Author Analyzer and Search Query
1. Assume you already have an index (_bs-06-23_) with docs in it.

	_OPTIONAL: you can also clone an index if necessary:_
	```
	POST /_reindex?pretty
	{
	  "source": {
		"index": "bitcoin-search-july-23"
	  },
	  "dest": {
		"index": "bs-06-23"
	  }
	}
	```

2. Close an index for read/write operations. [Close Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-close.html)
    ```  
    POST /bs-06-23/_close
    ```
	

3. Update the index with settings with your synonyms in `synonyms` field and define your analyzer in `analyzer` field. 
    ```
    PUT /bs-06-23/_settings?pretty
    {
      "settings": {
            "analysis": {
                "filter": {
                    "synonym_rule": {
                        "type": "synonym",
                        "lenient": "true",
                        "synonyms": [
                            "Greg Maxwell, gmaxwell, Gregory Maxwell, G. Maxwell, nullc",
                            "jgarzik, Jeff Garzik",
                            "Wladimir J. van der Laan, Wladimir, wumpus",
                            "Luke Dashjr, Luke-Jr",
                            "bitcoin-list at bluematt.me, Matt Corallo, lf-lists at mattcorallo.com",
                            "Johnson Lau, jl2012",
                            "David A. Harding, David Harding, harding, bitcoinops",
                            "cdecker, Christian Decker",
                            "eric at voskuil.org, Eric Voskuil",
                            "jonas.schnelli, Jonas Schnelli",
                            "Adam Back, adam3us, Dr Adam Back",
                            "jtimon, Jorge Timón, Jorge Timón",
                            "Warren Togami Jr., Warren Togami, wtogami",
                            "instagibbs, Greg Sanders",
                            "nopara73, Adam Ficsor",
                            "Achow101, Andrew Chow, achow101_alt, achow101",
                            "Vincenzo Palazzo, vincenzopalazzo",
                            "Sergi Delgado Segura, sr_gi",
                            "Nicolas DORIER, Nicolas Dorier",
                            "Rene Pickhardt, René Pickhardt",
                            "Mark Erhardt, Murch",
                            "sdaftuar, Suhas Daftuar",
                            "jnewbery, John Newbery",
                            "Antoine Poinsot, darosior",
                            "Jeremy, Jeremy Rubin",
                            "Kalle Alm, Karl-Johan Alm",
                            "gavinandresen, Gavin Andresen, Gavin",
                            "nickler, Jonas Nick",
                            "Sergio Lerner, Sergio Demian Lerner",
                            "Adam Gibson, AdamISZ",
                            "Btc Drak, Drak",
                            "Bastien TEINTURIER, Bastien Teinturier",
                            "Andreas M. Antonopoulos, Andreas Antonopoulos"
                        ]
                    },
                    "synonym_rule_q": {
                        "type": "synonym",
                        "synonyms": [
                            "Greg Maxwell, gmaxwell, Gregory Maxwell, G. Maxwell, nullc",
                            "jgarzik, Jeff Garzik",
                            "Wladimir J. van der Laan, Wladimir, wumpus",
                            "Luke Dashjr, Luke-Jr",
                            "bitcoin-list at bluematt.me, Matt Corallo, lf-lists at mattcorallo.com",
                            "Johnson Lau, jl2012",
                            "David A. Harding, David Harding, harding, bitcoinops",
                            "cdecker, Christian Decker",
                            "eric at voskuil.org, Eric Voskuil",
                            "jonas.schnelli, Jonas Schnelli",
                            "Adam Back, adam3us, Dr Adam Back",
                            "jtimon, Jorge Timón, Jorge Timón",
                            "Warren Togami Jr., Warren Togami, wtogami",
                            "instagibbs, Greg Sanders",
                            "nopara73, Adam Ficsor",
                            "Achow101, Andrew Chow, achow101_alt, achow101",
                            "Vincenzo Palazzo, vincenzopalazzo",
                            "Sergi Delgado Segura, sr_gi",
                            "Nicolas DORIER, Nicolas Dorier",
                            "Rene Pickhardt, René Pickhardt",
                            "Mark Erhardt, Murch",
                            "sdaftuar, Suhas Daftuar",
                            "jnewbery, John Newbery",
                            "Antoine Poinsot, darosior",
                            "Jeremy, Jeremy Rubin",
                            "Kalle Alm, Karl-Johan Alm",
                            "gavinandresen, Gavin Andresen, Gavin",
                            "nickler, Jonas Nick",
                            "Sergio Lerner, Sergio Demian Lerner",
                            "Adam Gibson, AdamISZ",
                            "Btc Drak, Drak",
                            "Bastien TEINTURIER, Bastien Teinturier",
                            "Andreas M. Antonopoulos, Andreas Antonopoulos"
                        ]
                    }
                },
                "analyzer": {
                    "analyzer_search": {
                        "filter": ["lowercase", "synonym_rule"],
                        "type": "custom",
                        "tokenizer": "iplexus_tokenizer"
                    },
                    "analyzer_q": {
                        "filter": ["lowercase", "synonym_rule_q"],
                        "type": "custom",
                        "tokenizer": "iplexus_tokenizer"
                    }
                },
                "tokenizer": {
                    "iplexus_tokenizer": {
                        "pattern": "[^a-zA-Z0-9\\p{InGreek}\\p{No}\\p{Lm}\\+\\-\\_]",
                        "type": "pattern",
                        "max_token_length": "256"
                    }
                }
            }
        }
    }
    ```


   _OPTIONAL: you can check an updated settings in your index:_ 
    ```
    GET /bs-06-23/_settings?pretty
    ```  


4. Update an index with the mappings to specify the analyzer for the fields of your choice, here we have updated for `authors` and `body` field:  
	```
	PUT /bs-06-23/_mapping?pretty
	{
		"properties": {
			"accepted_answer_id": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"authors": {
				"type": "text",
				"fields": {
					"analyzed": {
						"type": "text",
						"analyzer": "analyzer_search",
						"fielddata": true
					},
					"keyword": {
						"type": "keyword"
					}
				}
			},
			"body": {
				"type": "text",
				"fields": {
					"search": {
						"type": "text",
						"analyzer": "analyzer_search",
						"search_analyzer": "analyzer_q"
					}
				}
			},
			"body_formatted": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"body_type": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"categories": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"created_at": {
				"type": "date"
			},
			"domain": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"id": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"indexed_at": {
				"type": "date"
			},
			"language": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"media": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"tags": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"thread_url": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"title": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"transcript_by": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"type": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"url": {
				"type": "text",
				"fields": {
					"keyword": {
						"type": "keyword",
						"ignore_above": 256
					}
				}
			},
			"primary_topics": {
				"type": "keyword"
			},
			"secondary_topics": {
				"type": "keyword"
			},
			"summary": {
				"type": "text"
			}
		}
	}
	```

   _OPTIONAL: Check an updated mapping in your index:_ 
     ```
     GET /bs-06-23/mapping?pretty
   ```

5. Reopen an index for indexing or searching documents. [Open index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-open-close.html)
    ```
    POST /bs-06-23/_open
    ```

6. Use an analyzer and perform a text search simultaneously in a single API call, using `match` query in Elasticsearch, which accepts an analyzer parameter to specify which analyzer to use during search.
	```
	GET /bs-06-23/_search?pretty
	{
	  "_source": ["Id", "authors", "domain", "title", "type"],
	  "query": {
		"bool": {
		  "must": [
			{
			  "match": {
				"authors": {
				  "query": "greg maxwell",
				  "analyzer": "analyzer_search"
				}
			  }
			},
			{
			  "match": {
				"domain": "https://lists.linuxfoundation.org/pipermail/bitcoin-dev/"
			  }
			}
		  ]
		}
	  }
	}
	```
	---
	### _Breakdown of STEP-6:_
	
	(i). Analyzer API to generate tokens based on your input text: 

	```
	POST /bs-06-23/_analyze
	{
	  "analyzer": "analyzer_search",
	  "text": "greg maxwell"
	}
	```  

	(ii). Search API to query the index by passing generated tokens in `authors.analyzed`:
		
	```
	GET /bs-06-23/_search
	{
	  "_source": ["Id", "authors", "domain", "title", "type"],
	  "query": {
		"bool": {
		  "must": [
			{
			  "terms": {
				"authors.analyzed": [
				  "greg",
				  "maxwell",
				  "gregory",
				  "gmaxwell",
				  "nullc",
				  "g"
				]
			  }
			},
			{
			  "term": {
				"domain.keyword": "https://lists.linuxfoundation.org/pipermail/bitcoin-dev/"
			  }
			}
		  ]
		}
	  }
	}
	```
---

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
The output of the above query in `Dev Tools` is as follows:
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