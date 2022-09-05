
Dica COMO CRIAR BASH PARA FACILTAR ENVIO DOS COMANDOS

	a) Criar script bash com nome 'curl'

	b) Tornar arquivo executável: $ chmod a+x <nome-arquivo>

	  $ chmod a+x curl

	c) Executar os comandos

	  $ source .profile
	  $ which curl

	Retorno deve ser o local onde está apontando dentro do arquivo 'curl': 

	  /usr/bin/curl  

============================

localhost:9200   ->   https://site:d9d1ae58c56612f6a3c15f24b357cad5@gimli-eu-west-1.searchly.com

============================

1. Criar índice e os fields

- Mapping Explícito

curl --request PUT 'http://localhost:9200/microservice-logs' \
--data-raw '{
   "mappings": {
       "properties": {
           "timestamp": { "type": "date"  },
           "service": { "type": "keyword" },
           "host_ip": { "type": "ip" },
           "port": { "type": "integer" },
           "message": { "type": "text" }
       }
   }
}'
 
 
Verificando se foi criado o índice e os fields

  curl -XGET 'https://site:d9d1ae58c56612f6a3c15f24b357cad5@gimli-eu-west-1.searchly.com/microservice-logs/_mapping'
 
 
Resposta:

  {"microservice-logs":{"mappings":{"properties":{"host_ip":{"type":"ip"},"message":{"type":"text"},"port":{"type":"integer"},"service":{"type":"keyword"},"timestamp":{"type":"date"}}}}}
  
  
============================ 
 
2. "port": 12345 -> tipo inteiro
 
{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": "Started!" }
 
============================ 
 
3. "port": "15000" -> está inserindo como inteiro (e foi definido como tipo inteiro no mapping) -> não dá erro, faz o cast
 
curl --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "XYZ", "host_ip": "10.0.2.15", "port": "15000", "message": "Hello!" }'
 
 
============================

4. "port": "NONE" -> está inserindo com "NONE" (e foi definido como tipo inteiro no mapping) -> ERRO, não consegue fazer o cast
 
curl --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "XYZ", "host_ip": "10.0.2.15", "port": "NONE", "message": "I am not well!" }'
 
============================ 
 
5. Definir parâmetro para tratar esse tipo de erro de cast (irá ignorar)
 
### Fechando o índice -> _close
curl --request POST 'http://localhost:9200/microservice-logs/_close'
 
### Alterar o parâmetro para ignorar -> _settings  +  "index.mapping.ignore_malformed": true
curl --location --request PUT 'http://localhost:9200/microservice-logs/_settings' \
--data-raw '{
   "index.mapping.ignore_malformed": true
}'
 
### Abrindo o índice -> _open
curl --request POST 'http://localhost:9200/microservice-logs/_open'

============================

6. Agora inserindo novamente com "port": "NONE" -> agora funciona e o campo será omitido e não será usado para indexação
 
curl --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "XYZ", "host_ip": "10.0.2.15", "port": "NONE", "message": "I am not well!" }'
 
============================

7. "message": {"data": {"received":"here"}}  ->  inserindo um JSON onde deveria ser um texto

  - Dá **erro** mesmo com a configuração para ignorar erro de cast
 
curl --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": {"data": {"received":"here"}}}'
 
============================

8. payload": {"data": {"received":"here"}} -> definindo campo "payload" e atribuindo um JSON -> insere com sucesso, o mapping será alterado

  - Mapping Dinâmico: Pode levar a "mapping explosion"
 
curl --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": "Received...", "payload": {"data": {"received":"here"}}}'
 
 
curl --request GET  'http://localhost:9200/microservice-logs/_mappings?pretty'
 
============================ 
 
9. "payload": {"data": {"received": {"even": "more"}}} -> vai gerar **erro** porque o mapping foi definido de forma dinâmica, baseado no primeiro insert com esse campo
 
  - Dá **erro** mesmo com a configuração para ignorar erro de cast


curl --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": "Received...", "payload": {"data": {"received": {"even": "more"}}}}'
 
  - ES tem limite de 1000 campos no json
 
============================ 
 
10.

- Instalar jq

    - [Comando jq no Linux (manipula arquivos json)](https://www.certificacaolinux.com.br/comando-linux-jq/)

    - [Baeldung - Guide to Linux jq Command for JSON Processing](https://www.baeldung.com/linux/jq-command-json)

    - [Cookbook](https://github.com/stedolan/jq/wiki/Cookbook)

    - [jq - script bash](https://github.com/eugenp/tutorials/blob/master/linux-bash/json/src/main/bash/jq.sh)
  
sudo apt-get install jq

- Gerando json com 1001 campos
  
thousandone_fields_json=$(echo {1..1001..1} | jq -Rn '( input | split(" ") ) as $nums | $nums[] | . as $key | [{key:($key|tostring),value:($key|tonumber)}] | from_entries' | jq -cs 'add')
 
- Visualizando o json
  
echo "$thousandone_fields_json"
 
- Criar índice

curl --location --request PUT  'http://localhost:9200/big-objects'

- Carregar o arquivo -> vai dar erro porque estourou o limite de 1000 fields do ES

curl --request POST  'http://localhost:9200/big-objects/_doc?pretty' \
--data-raw "$thousandone_fields_json"
 
 
- Aumentar o limite para 1001 -> _settings

Pode gerar problema no cluster, cuidado ao aumentar esse limite!

curl --location --request PUT  'http://localhost:9200/big-objects/_settings' \
--data-raw '{
"index.mapping.total_fields.limit": 1001
}'
