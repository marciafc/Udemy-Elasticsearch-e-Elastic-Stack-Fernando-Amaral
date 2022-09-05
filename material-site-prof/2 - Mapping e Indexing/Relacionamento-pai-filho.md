# Relacionamento pai e filho entre documentos

  - [Mapping Types](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/mapping-types.html)

  - [Join field type](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/parent-join.html)

  - A parent/child relation can be defined as follows:
  
    - "type": "join"
	
	- "relations"
	
```
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "my_id": {
        "type": "keyword"
      },
      "my_join_field": { 
        "type": "join",
        "relations": {
          "question": "answer" 
        }
      }
    }
  }
}
```
  - Retorna os documentos filhos associados ao pai
  
    - [Has parent query](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-has-parent-query.html#query-dsl-has-parent-query)
	
	  - "has_parent"
```	  
GET /my-index-000001/_search
{
  "query": {
    "has_parent": {
      "parent_type": "parent",
      "query": {
        "term": {
          "tag": {
            "value": "Elasticsearch"
          }
        }
      }
    }
  }
}	  
```

  - Retorna o documento pai vinculado ao filho
  
    - [Has child query](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-has-child-query.html)
	
	  - "has_child"
```	
GET /_search
{
  "query": {
    "has_child": {
      "type": "child",
      "query": {
        "match_all": {}
      },
      "max_children": 10,
      "min_children": 2,
      "score_mode": "min"
    }
  }
}
```