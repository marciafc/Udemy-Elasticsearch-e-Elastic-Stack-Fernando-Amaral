# Analyzers

  - [Mapping Types](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/mapping-types.html)
  
  - [Match query](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-match-query.html)
  
  - "type": "text" -> busca qlq coisa, mesmo que tenha baixa relevância
  
  - "type": "keyword" -> busca exata pelo termo
  
  - Alterar de "text" para "keyword": recriar índice e criar novamente como keyword para ter busca exata, e por fim inserir em lote novamente:
  
    - XDELETE host:9200/movies
	
	- XPUT host:9200/movies -d 
```'{
	"mappings" : {
		"properties" : {
			"speaker" : {"type": "keyword" },
			"play_name" : {"type": "keyword" },
			"line_id" : { "type" : "integer" },
			"speech_number" : { "type" : "integer" }
		}
	}
}'
```
    - XPUT host:9200/_bulk
