1.

## Antes de executar, criar o índice 'demo-default' no Searchly
 
curl -XPUT "https://site:aa15b92b712684113f8e94547dace84c@gimli-eu-west-1.searchly.com/demo-default/_doc/1" -d'{
  "message": "[5592:1:0309/123054.737712:ERROR:child_process_sandbox_support_impl_linux.cc(79)] FontService unique font name matching request did not receive a response.",
  "fileset": {
    "name": "syslog"
  },
  "process": {
    "name": "org.gnome.Shell.desktop",
    "pid": 3383
  },
  "@timestamp": "2020-03-09T18:00:54.000+05:30",
  "host": {
    "hostname": "bionic",
    "name": "bionic"
  }
}' 

## Local

curl -XPUT "http://127.0.0.1:9200/demo-default/_doc/1" -d'{
  "message": "[5592:1:0309/123054.737712:ERROR:child_process_sandbox_support_impl_linux.cc(79)] FontService unique font name matching request did not receive a response.",
  "fileset": {
    "name": "syslog"
  },
  "process": {
    "name": "org.gnome.Shell.desktop",
    "pid": 3383
  },
  "@timestamp": "2020-03-09T18:00:54.000+05:30",
  "host": {
    "hostname": "bionic",
    "name": "bionic"
  }
}'
 
 
==========================  
 
2.

curl -XGET "https://site:aa15b92b712684113f8e94547dace84c@gimli-eu-west-1.searchly.com/demo-default/_mapping?pretty=true"
 
curl -XGET "http://127.0.0.1:9200/demo-default/_mapping?pretty=true"
 
========================== 

3.
 
## Estado do cluster para um arquivo json -> _cluster

curl -XGET "http://127.0.0.1:9200/_cluster/state?pretty=true" >> es-cluster-state.json
 
========================== 

4.
 
curl -XPUT "https://site:a8c4425d01cd73d33bacf3bd0935742e@gimli-eu-west-1.searchly.com/demo-flattened"

curl -XPUT "http://127.0.0.1:9200/demo-flattened"
 
========================== 
 
 
5. "type": "flattened" -> The flattened type provides an alternative approach, where the entire object is mapped as a single field.

  - [Mapping Types](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/mapping-types.html)

  - [Flattened](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/flattened.html)
  
  - Flattened são keyword
  
  - The flattened mapping type should not be used for indexing all document content, as it treats all values as keywords and does not provide full search functionality.
 
curl -XPUT "http://127.0.0.1:9200/demo-flattened/_mapping" -d'{
  "properties": {
    "host": {
      "type": "flattened"
    }
  }
}'

========================== 

 
6.
 
curl -XPUT "http://127.0.0.1:9200/demo-flattened/_doc/1" -d'{
  "message": "[5592:1:0309/123054.737712:ERROR:child_process_sandbox_support_impl_linux.cc(79)] FontService unique font name matching request did not receive a response.",
  "fileset": {
    "name": "syslog"
  },
  "process": {
    "name": "org.gnome.Shell.desktop",
    "pid": 3383
  },
  "@timestamp": "2020-03-09T18:00:54.000+05:30",
  "host": {
    "hostname": "bionic",
    "name": "bionic"
  }
}'

==========================
 
7.
 
curl -XGET "http://127.0.0.1:9200/demo-flattened/_mapping?pretty=true"
 
==========================


8. Atualizando com os campos "osVersion" e "osArchitecture" que não existiam
 
curl -XPOST "http://127.0.0.1:9200/demo-flattened/_update/1" -d'{
    "doc" : {
        "host" : {
          "osVersion": "Bionic Beaver",
          "osArchitecture":"x86_64"
        }
    }
}'
 
==========================

9. Busca será pelo valor exato, pois flattened == keyword
 
curl -XGET "http://127.0.0.1:9200/demo-flattened/_search?pretty=true" -d'{
  "query": {
    "term": {
      "host": "Bionic Beaver"
    }
  }
}'
 
==========================

10. Consulta aninhada ("osVersion" é um campo de "host")
 
curl -XGET "http://127.0.0.1:9200/demo-flattened/_search?pretty=true" -d'{
  "query": {
    "term": {
      "host.osVersion": "Bionic Beaver"
    }
  }
}'

==========================
 
11. Busca parcial com "host.osVersion": "Beaver" -> Não irá encontrar (apenas busca exact match com flattened)
 
curl -XGET "http://127.0.0.1:9200/demo-flattened/_search?pretty=true" -d'{
  "query": {
    "term": {
      "host.osVersion": "Beaver"
    }
  }
}'
