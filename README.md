# Elastic research
## Table of Contents

- [Elastic](#elastic)
  - [Concept](#concept)
- [Environment](#enviroment)
  - [Install](#install)
  - [Hello world](#hello-world)
- [Cluster](#cluster)
  - [Cluster health](#cluster-health)
  - 
- [Prepare data](#prepare-data)
- [Verify analytics](#verify-analytics)

## Elastic
Elastic search is a real time distributed search and analytics search engine. It is used for full text search, structured search, analytics.

Unfortunately, most databases are astonishingly inept at extracting actionable knowledge from your data. Sure, they can filter by timestamp or exact values, but can they perform full-text search, handle synonyms, and score documents by relevance? Can they generate analytics and aggregations from the same data? Most important, can they do this in real time without big batch-processing jobs?

This is what sets Elasticsearch apart: Elasticsearch encourages you to explore and utilize your data, rather than letting it rot in a warehouse because it is too difficult to query.

### Concept

- Relevance
- 
  
  
### Environment

- Install
  - Retrieved elasticsearch from docker
  
  ```
  docker pull docker.elastic.co/elasticsearch/elasticsearch:6.0.1
  ```
  
  - Run with develop mode
  
  ```
  docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.0.1
  ```

  - Docker cluster, please refer: https://www.elastic.co/guide/en/elasticsearch/reference/6.0/docker.html#docker-prod-cluster-composefile

  - Docker cluster
     - Configure the docker-compose.yml
     
     ```
       version: '2.2'
			services:
			  elasticsearch:
			    image: docker.elastic.co/elasticsearch/elasticsearch:6.0.1
			    container_name: elasticsearch
			    environment:
			      - cluster.name=docker-cluster
			      - bootstrap.memory_lock=true
			      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
			    ulimits:
			      memlock:
			        soft: -1
			        hard: -1
			    volumes:
			      - esdata1:/usr/share/elasticsearch/data
			    ports:
			      - 9200:9200
			    networks:
			      - esnet
			  elasticsearch2:
			    image: docker.elastic.co/elasticsearch/elasticsearch:6.0.1
			    container_name: elasticsearch2
			    environment:
			      - cluster.name=docker-cluster
			      - bootstrap.memory_lock=true
			      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
			      - "discovery.zen.ping.unicast.hosts=elasticsearch"
			    ulimits:
			      memlock:
			        soft: -1
			        hard: -1
			    volumes:
			      - esdata2:/usr/share/elasticsearch/data
			    networks:
			      - esnet
			
			volumes:
			  esdata1:
			    driver: local
			  esdata2:
			    driver: local
			
			networks:
			  esnet:
     ```
     - start up the docker cluster
     
	   ```
	     docker-compose up
	   ``` 
	   
	 - user docker ps to see the running container
	 
	   ```
	   Jesse-4:elasticresearch jessepi$ docker ps
		CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                              NAMES
		4d0dd2efcecf        docker.elastic.co/elasticsearch/elasticsearch:6.0.1   "/usr/local/bin/do..."   33 minutes ago      Up 33 minutes       0.0.0.0:9200->9200/tcp, 9300/tcp   elasticsearch
		f3049a024ed5        docker.elastic.co/elasticsearch/elasticsearch:6.0.1   "/usr/local/bin/do..."   33 minutes ago      Up 33 minutes       9200/tcp, 9300/tcp                 elasticsearch2
		```
		
	  - Enter into the docker container
	  
	  ```
	  Jesse-4:elasticresearch jessepi$ docker exec -it f3049a024ed5 /bin/bash
[root@f3049a024ed5 elasticsearch]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
elastic+     1     0  3 05:59 ?        00:01:11 /usr/lib/jvm/jre-1.8.0-openjdk/bin/java -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+
elastic+    98     1  0 05:59 ?        00:00:00 /usr/share/elasticsearch/plugins/x-pack/platform/linux-x86_64/bin/controller
root       232     0  0 06:34 pts/0    00:00:00 /bin/bash
root       247   232  0 06:35 pts/0    00:00:00 ps -ef
[root@f3049a024ed5 elasticsearch]#
```

### Hello world
- Put data into elastic search

```
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}

PUT /megacorp/employee/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}

```
- Retrieving data

```
GET /megacorp/employee/1
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}

```

- Search the data

```
GET /megacorp/employee/_search
{
   "took":      6,
   "timed_out": false,
   "_shards": { ... },
   "hits": {
      "total":      3,
      "max_score":  1,
      "hits": [
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "3",
            "_score":         1,
            "_source": {
               "first_name":  "Douglas",
               "last_name":   "Fir",
               "age":         35,
               "about":       "I like to build cabinets",
               "interests": [ "forestry" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "1",
            "_score":         1,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            "_index":         "megacorp",
            "_type":          "employee",
            "_id":            "2",
            "_score":         1,
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}

```

```
/megacorp/employee/_search?q=last_name:Smith
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

- Search with Query DSL(domain-specific language)

```
POST /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}

POST /megacorp/employee/_search
{
    "query" : {
        "bool" : {
            "must" : {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter" : {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}

```

- Full-Text search

```
POST /megacorp/employee/_search

{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}

{
  "took": 11,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "1",
        "_score": 0.5753642,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      },
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "2",
        "_score": 0.2876821,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      }
    ]
  }
}

```

- Phrase Search

```
POST /megacorp/employee/_search

{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}


 {
  "took": 20,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "1",
        "_score": 0.5753642,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      }
    ]
  }
}

```

- Highlighting Our Searches

```
POST /megacorp/employee/_search

{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}

{
  "took": 120,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "1",
        "_score": 0.5753642,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        },
        "highlight": {
          "about": [
            "I love to go <em>rock</em> <em>climbing</em>"
          ]
        }
      }
    ]
  }
}

```

- Analytics

```
POST /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "age"
      }
    }
  }
}

{
  "took": 86,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "2",
        "_score": 0.2876821,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      },
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      }
    ]
  },
  "aggregations": {
    "all_interests": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": 25,
          "doc_count": 1
        },
        {
          "key": 32,
          "doc_count": 1
        }
      ]
    }
  }
}

```

## Cluster