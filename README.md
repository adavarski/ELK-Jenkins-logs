1.Install and configure filebeat:
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.6.1-x86_64.rpm
sudo rpm -vi filebeat-6.6.1-x86_64.rpm

# cat /etc/filebeat/filebeat.yml 
filebeat:
  prospectors:
    -
      paths:
        - "/var/lib/jenkins/jobs/*/builds/*/log"
 
output:
  logstash:
        hosts: ["localhost:5044"]


#  systemctl start filebeat
#  systemctl enable filebeat
```
2.Dockerizing Jenkins build logs with ELK stack (Elasticsearch, Logstash and Kibana).
```
# cat docker-compose-elk.yml
version: "3.1"

services:

  logstash:
    image: logstash:2
    ports:
     - "5044:5044"
    volumes:
          - ./:/config
    command: logstash -f /config/logstash.conf
    links:
     - elasticsearch
    depends_on:
     - elasticsearch

  elasticsearch:
     image: elasticsearch:5.6.15
     ports:
      - "9200:9200"
     volumes:
      - "./es_data/es_data:/usr/share/elasticsearch/data/"


  kibana:
    image: kibana:5
    ports:
     - "5601:5601"
    links:
     - elasticsearch
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
    depends_on:
     - elasticsearch


# cat logstash.conf 
input { beats {      port => 5044    }  }
output {
  stdout { codec => rubydebug }
  elasticsearch { hosts => ["elasticsearch:9200"] }
}
```
#run elk stack
```
docker-compose -f docker-compose-elk.yml up 
docker-compose -f docker-compose-elk.yml up -d

# docker-compose -f docker-compose-elk.yml up -d
Creating elk_elasticsearch_1 ... done
Creating elk_kibana_1        ... done
Creating elk_logstash_1      ... done
```

Check Elasticsearch 

Elasticsearch: http://{JENKINS_SERVER_IP}:9200/_cat/health?v

Kibana setup:

http://{JENKINS_SERVER_IP}:5601/

Hit the “create” button to create the first index pattern. As the logs come from logstash, the index will have the pattern “logstash-*” where “*” will be any date like “logstash-2019.02.28 ”. Once done kibana will pick up logs and can hit “Discover” to see logs. 
