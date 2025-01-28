# Elastic-search

    docker run -d --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" -e "ELASTIC_PASSWORD=your_password" elasticsearch:8.0.0

    https://localhost:9200/
    
    powershell
    [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("elastic:Aa@123456"))

    PUT https://localhost:9200/my_index
    Authorization: Basic ZWxhc3RpYzpBYUAxMjM0NTY=
    Content-Type: application/json
    
    {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
      }
    }
    ###
    GET https://localhost:9200/_cat/indices?v
    Authorization: Basic ZWxhc3RpYzpBYUAxMjM0NTY=
    
    ###
    POST https://localhost:9200/my_index/_doc/2
    Authorization: Basic ZWxhc3RpYzpBYUAxMjM0NTY=
    Content-Type: application/json
    
    {
      "title": "2 Document",
      "filed3": "This is the content of the 2 document."
    }
    
    ###
    GET https://localhost:9200/my_index/_search
    Authorization: Basic ZWxhc3RpYzpBYUAxMjM0NTY=
    Content-Type: application/json
    
    {
      "query": {
        "match": {
          "title": "first"
        }
      }
    }
    
    ###
    GET https://localhost:9200/my_index/_search
    Authorization: Basic ZWxhc3RpYzpBYUAxMjM0NTY=
    Content-Type: application/json
    
    {
      "query": {
        "match_all": {}
      }
    }
