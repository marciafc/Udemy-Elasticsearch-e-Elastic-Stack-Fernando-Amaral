# Controle de concorrência otimista

  - if_seq_no=362&if_primary_term=2

  - [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/optimistic-concurrency-control.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/optimistic-concurrency-control.html)
  
```
PUT products/_doc/1567?if_seq_no=362&if_primary_term=2
{
  "product": "r2d2",
  "details": "A resourceful astromech droid",
  "tags": [ "droid" ]
}
```

  - retry_on_conflict
  
    - [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/docs-update.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/docs-update.html)