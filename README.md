# DEPRECATED

This was moved into https://github.com/cristian04/bank-statements-data a more extensive solution.

# util-bnz-data-extractor
A ~~simple~~ python ~~script~~ to consume the BNZ API and export it to ElasticSearch

Please vote for an API into https://community.bnz.co.nz/t/api-application-programming-interface/337

Disclaimer: Work in Progress

##### TODO's list 

- [x] Learn about elastic-search
- [x] ~~Convert Bank's csv files into JSON files~~ Not needed. CSV files must be ingested by using logstash. Json file must be ingested by elastic-search api
- [x] Inject these JSON files into elastic-search
- [ ] Create a Kibana dashboard to show `interesting data` <sup>1</sup>
- [ ] Create a Machine Learning Engine to analyze and categorize data <sup>1</sup>

<sup>1</sup> No idea of what Im doing


# Project requirements

- Python 2.7
- Virtualenv
- Pip
- Docker
- Docker-machine
- Docker-compose
- *Kubernetes support soon*

# Architecture

Use stackedit.io to see this.

```sequence
BNZ Extractor->BNZ API: Get accounts
BNZ API->BNZ Extractor: List of accounts
BNZ Extractor->Download Worker: Notify
BNZ Extractor->MongoDB: Initial Structure
Download Worker->BNZ API: Get transactions
BNZ API->Download Worker: transaction
Download Worker->MongoDB: Insert transactions
Download Worker->Process Worker: Notify
Process Worker->Elastic Search: Import transaction
Kibana->Elastic Search: Discovery
```

- Transactions are saved on mongodb as a backup ( I dont know when BNZ will stop allowing this)
- Transactions are imported on Elastic Search for analysis
- Workers are managed by RQ using REDIS

ELK stack is based on https://github.com/deviantony/docker-elk

# Getting started

- Install project dependencies by typing
  ```
  $ virtualenv .
  $ pip install -r requirements.txt
  ```

## Starting docker-machine and docker containers
Starting docker-machine
```
docker-machine start default
```

Setting up the docker context
```
eval (docker-machine env default)
```


We need more memory for ElasticSearch
```
docker-machine ssh

 sudo sysctl -w vm.max_map_count=262144
 exit

```

Starting the services that we will conect with
```
docker-compose up elasticsearch kibana database message-queue workers-dashboard node
```
Optional: Testing with nmap that every service is running

```
nmap 192.168.99.100 -p 27017,9181,5601,6379,9200,9300,3000 # Just to test that everything is on place
```

# Running the project (Magic)

Auth token comes from the BNZ App. Use chrome inspector to get it.

Starting a worker to import into elastic search. Doesn't need auth token.
```
rq worker -c processor-worker-settings # Turn on the worker to import into ES
```

Starting the download worker, who downloads the transactions and import it into MONGODB
```
env auth='' rq worker -c download-worker-settings # Start the worker to download from BNZ
```

Starting the main app, who get the accounts ids and builds the timeslots into the database
```
env auth='' start_date='2014-06-01' freq='w' python bnz-extractor.py # Start the app that creates the initial structure
```
# Additional info

- Kibana: [http://192.168.99.100:5601](http://192.168.99.100:5601)
- Elastic Search: [http://192.168.99.100:9200](http://192.168.99.100:9200)
- RQ dashboard: [http://192.168.99.100:9181](http://192.168.99.100:9181)


# Credits

[Cristian Russo](http://www.cristianmarquez.me)
