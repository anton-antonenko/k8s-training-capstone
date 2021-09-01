# K8s training capstone project

### The task for the project is the following:

You need to migrate a project from docker-compose to Kubernetes. With provided docker-compose.yaml you have to create all necessary specifications for Kubernetes deployment.

Requirements:
1. Every single application in this project has to have at least one Service.
2. Elasticsearch is a single-clustered application.
3. Elasticsearch should be deployed as a StatefulSet, Persistent Volumes would be a plus.
4. All communications have to be implemented through Services.
5. Kibana and web-server have to be accessible from outside of the cluster.
6. All shared configs should be managed from one place.
7. In Kubernetes specifications you should use as many different kinds of Kubernetes objects as possible.

```
version: '3'
services:
  web-server:
    image: 'quay.io/myafk/interactive:stable'
    command: interactive ingress
    environment:
      REDIS_SLAVE_HOST: redis-slave
      REDIS_MASTER_HOST: redis-master
      KUBERNETES_SERVICE_HOST: any string here
      KUBERNETES_SERVICE_PORT: any thing here
      ELASTICSEARCH_HOSTS: 'http://es01:9200'
    ports:
      - '8085:8085'
  redis-master:
    image: redis
    ports:
      - '6379:6379'
  redis-slave:
    image: redis
    command: bash -c "redis-server --slaveof $$REDIS_MASTER_HOST 6379"
    environment:
      REDIS_MASTER_HOST: redis-master
    ports:
      - 6379
  es01:
    image: 'docker.elastic.co/elasticsearch/elasticsearch:7.1.1'
    container_name: es01
    environment:
      - node.name=es01
      - discovery.seed_hosts=es02
      - 'cluster.initial_master_nodes=es01,es02'
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=false
    ports:
      - '9200:9200'
  es02:
    image: 'docker.elastic.co/elasticsearch/elasticsearch:7.1.1'
    container_name: es02
    environment:
      - node.name=es02
      - discovery.seed_hosts=es01
      - 'cluster.initial_master_nodes=es01,es02'
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=false
    ports:
      - 9200
  kibana:
    image: 'docker.elastic.co/kibana/kibana:7.1.1'
    environment:
      ELASTICSEARCH_HOSTS: 'http://es01:9200'
    ports:
      - '5601:5601'
```

To deploy execute: 
```
helm install capstone ./project
```