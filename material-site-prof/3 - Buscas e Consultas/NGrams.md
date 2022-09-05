1. Deletar o índice

curl -XDELETE 127.0.0.1:9200/movies


2. Criar o analyzer

curl -XPUT '127.0.0.1:9200/movies?pretty' -d '
{
 "settings": {
 "analysis": {
 "filter": {
   "autocomplete_filter": {
     "type": "edge_ngram",
     "min_gram": 1,
     "max_gram": 20
   }
 },
"analyzer": {
 "autocomplete": {
   "type": "custom",
   "tokenizer": "standard",
   "filter": [
     "lowercase",
     "autocomplete_filter"
   ]
  }
 }
 }
 }
}'


3. Testando o analyser com o texto 'sta'  -> _analyze -> Vai mostrar os unigram, bigram, trigram que foram criados

curl -XGET '127.0.0.1:9200/movies/_analyze?pretty' -d '
{
"analyzer": "autocomplete",
"text": "sta"
}'


4. Mapeando o analyzer para o campo "title" -> _mapping

curl -XPUT '127.0.0.1:9200/movies/_mapping?pretty' -d '
{
"properties" : {
  "title": {
    "type" : "text",
    "analyzer": "autocomplete"
  }
 }
}'


5. Reindexar os dados -> _bulk

curl -XPUT 127.0.0.1:9200/_bulk --data-binary @movies.json


6. Testando analyzer -> vai trazer vários resultados, por exemplo usando o unigran 's', não serve

curl -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
 "query": {
   "match": {
     "title": "sta"
   }
  }
}'


7. Para resolver o problema anterior, usar 'query' e 'analyzer' -> agora sim

  - A consulta não vai ser dividida em n-grams, não vamos ter resultados com "s", "t", "a" etc.

curl -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match": {
      "title": {
        "query": "sta",
        "analyzer": "standard"
      }
   }
  }
}'


8. Irá retornar filmes que tenha 'star tr', não necessariamente igual (com score/relevância menor)

curl -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match": {
      "title": {
        "query": "star tr",
        "analyzer": "standard"
      }
    }
  }
}'


