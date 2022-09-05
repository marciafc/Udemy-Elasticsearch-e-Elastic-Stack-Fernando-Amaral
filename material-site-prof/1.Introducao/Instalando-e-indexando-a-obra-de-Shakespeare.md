Instalando e indexando a obra de Shakespeare

# Baixando arquivo shakes-mapping.json com as propriedades do índice 'shakespeare'
# Arquivo será usado no próximo comando
wget http://media.sundog-soft.com/es7/shakes-mapping.json

# PUT com arquivo shakes-mapping.json para criar índice 'shakespeare'
#  Local
  curl -H 'Content-Type: application/json' -XPUT 127.0.0.1:9200/shakespeare --data-binary @shakes-mapping.json
# Searchly 
  curl -H 'Content-Type: application/json' -XPUT https://site:f52d70026563ae22552e61078a46eb6a@gimli-eu-west-1.searchly.com/shakespeare --data-binary @shakes-mapping.json

# Baixando arquivo json contendo as obras do Shakespeare
# Arquivo será usado no próximo comando
wget http://media.sundog-soft.com/es7/shakespeare_7.0.json

# Inserção em lote _bulk
# POST com arquivo shakespeare_7.0.json - inserir várias obras do Shakespeare no ES 
# Pode demorar até indexar tudo
# Local
  curl -H 'Content-Type: application/json' -XPOST '127.0.0.1:9200/shakespeare/_bulk?pretty' --data-binary @shakespeare_7.0.json
# Searchly 
  curl -H 'Content-Type: application/json' -XPOST 'https://site:f52d70026563ae22552e61078a46eb6a@gimli-eu-west-1.searchly.com/shakespeare/_bulk?pretty' --data-binary @shakespeare_7.0.json

# GET query buscar por trecho de texto
# Local
  curl -H 'Content-Type: application/json' -XGET '127.0.0.1:9200/shakespeare/_search?pretty' -d '{"query" : {"match_phrase" : {"text_entry" : "to be or not to be"}}}'
# Searchly
  curl -H 'Content-Type: application/json' -XGET 'https://site:f52d70026563ae22552e61078a46eb6a@gimli-eu-west-1.searchly.com/shakespeare/_search?pretty' -d '{"query" : {"match_phrase" : {"text_entry" : "to be or not to be"}}}'
# Searchly - Pesquisando usando query parameter 'q'  
  curl -XGET 'https://site:f52d70026563ae22552e61078a46eb6a@gimli-eu-west-1.searchly.com/shakespeare/_search?pretty&q=to%20be%20or%20not%20to%20be'

# Buscando um documento pelo id (no caso, id = 2)

  curl -H 'Content-Type: application/json' -XGET 'https://site:f52d70026563ae22552e61078a46eb6a@gimli-eu-west-1.searchly.com/shakespeare/_doc/2?pretty' 
  
Resposta:

{
  "_index" : "shakespeare",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 1,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "type" : "line",
    "line_id" : 3,
    "play_name" : "Henry IV",
    "speech_number" : "",
    "line_number" : "",
    "speaker" : "",
    "text_entry" : "Enter KING HENRY, LORD JOHN OF LANCASTER, the EARL of WESTMORELAND, SIR WALTER BLUNT, and others"
  }
}

