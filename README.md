
Starts the [FlockData](http://FlockData.com) stack with Docker Compose

## Status
Docker and docker-compose is the only current pre-packaged distribution. Pre-built artifacts are available in the [MVNRepostitory](https://mvnrepository.com/artifact/org.flockdata) for alternative deployment scenarios 

You'll need to ensure you've installed [DockerToolbox](https://www.docker.com/products/docker-toolbox) before proceeding. Do not run this with the native docker osx/windows as underlying community libraries are still being upgraded to address the new architecture.

Check that your docker-machine is up and running

`docker-machine ip`

Hereon please replace references to `docker-ip` with your machines docker IP, or create an alias in your hosts file for convenience. 

# Installation

## Step 1 - Clone this repo
`git clone https://github.com/monowai/fd-demo`

`cd fd-demo`

## Step 2 Start the services
`docker-compose up -d` This will take time to pull images from the docker hub - be patient!

Congratulations - you've now installed started ElasticSearch, Neo4j, Riak, RabbitMQ, FdEngine, FdSearch, FdStore, FdDiscovery and a bunch of other useful apps 

# Testing the install
Start the FD interactive shell

`docker-compose run fd-client`

The fd-shell is a command line wrapper around the REST api exposed by FlockData. You can use `curl` to run commands against the endpoints if you prefer

Sample commands:

```
fd>ping
pong

fd>env
ClientConfiguration - org.fd.engine.api [http://fd-engine:8091], org.fd.client.http.user [demo], org.fd.client.batchsize [1]
**** FlockData RabbitAMQP configuration deployed. rabbit.host set to [rabbit:5672], rabbit.user [guest]

fd>health
{
  "eureka.client.serviceUrl.defaultZone" : "http://eureka:8761/eureka/",
  "fd-search" : {
    "health" : {
      "org.fd.search.es.settings" : "fd-default-settings.json",
      "elasticsearch" : {
        "Status" : "ok",
        "nodeName" : "Fasaud",
        "dataNodes" : 1,
        "nodes" : 1,
        "clusterName" : "es_flockdata",
        "health" : "YELLOW"
      },
      "es.nodes" : "172.19.0.3:9300",
      "org.fd.search.es.mapping" : "./",
      "fd.search.version" : "0.98.8-SNAPSHOT (geo-tools/1273d8e)"
    },
    "org.fd.search.api" : "http://fd-search:8092",
    "status" : "ok"
  },
  "fd-store" : {
    "org.fd.store.api" : "http://fd-store:8093",
    "fd.store.engine" : "RIAK",
    "fd.store.enabled" : "true",
    "status" : "OK - Riak"
  },
  "fd.version" : "0.98.8-SNAPSHOT (geo-tools/1273d8e)",
  "rabbit.host" : "rabbit",
  "rabbit.port" : "5672",
  "rabbit.user" : "guest",
  "spring.cloud.config.discovery.enabled" : "false"
}
```

## Use the UI
At this point you can login to [fd-view](http://docker-ip) use demo/123 for credentials. You will be asked to register your account as a data access user in order to write data to the service.

## Say hello 

`curl -u demo:123 http://docker-ip:8091/info`

Search and store have unsecured endpoints. fd-engine talks to them

fd-search

`curl http://docker-ip:8092/api/ping`

fd-store

`curl http://docker-ip:8093/api/ping`

Riak
`curl http://docker-ip:8093/api/v1/admin/ping/riak` (Verify fd-store connectivity to Riak)

## Packaged services
|Service   |Description   |Address   |
|---|---|---|
|[fd-view](https://github.com/monowai/fd-view) |FlockData's integrated browser and analysis tool   |[http://docker-ip](http://docker-ip)|
|[weavescope](https://www.weave.works/products/weave-scope/)|visual overview of the stack   |[http://docker-ip:4040](http://docker-ip:4040)|
|RabbitMQ |messaging admin|[http://docker-ip:15672](http://docker-ip:15672)|
|[ElasticSearch](https://www.elastic.co)|Rest based search API |[http://docker-ip:9200](http://docker-ip:9200)|
[neo4j-browser](http://neo4j.org)|HTML graph browser and REST access to Neo4j|[http://docker-ip:7474](http://docker-ip:7474)|
[Kibana](https://www.elastic.co/products/kibana)|ElasticSearch data viz|[http://docker-ip:5601](http://docker-ip:5601)|
|Spring/Netflix Eureaka|Monitoring|[http://docker-ip:8761](http://docker-ip:8761)|
|[PostMan](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en)|Documentation for the REST API|[Postman](https://github.com/monowai/flockdata.org/blob/master/fd.api-postman.json)|

## Security

FlockData uses configurable [security mechanisms](https://github.com/monowai/flockdata.org/tree/master/fd-security) backed by the Spring Security framework. We love [StormPath](http://stormpath.com); you might too.
This stack uses simple security credentials that are managed in the `docker-compose.yml` file. Namely a single set of credentials - `demo` / `123`

FlockData currently has two access roles:

|Role|Purpose|
|---|---|
|FD_ADMIN|Allows performing administrative functions and creating data access users|
|FD_USER|Allows user accounts access to the service. Should be registered to connect to a SystemUser account that authorises reading and writing of data|

Authorised users are accounts that exist in your security domain. Authorised users need to have a FlockData SystemUser identity established. 

Security roles are defined in your authorisation sub-system - LDAP/AD/StormPath etc. They are not stored in FlockData (except in the case of using the simpleauth security mechanism).    

SystemUser identify
    Authenticated accounts need to be connected to a system user account to read and write data. This is done via the RegistrationEP or using the flockdata/fd-client docker image

## Registering a SystemUser account
This example logs in as an FD_ADMIN account - `demo` - and creates a SystemUser identify for the login identifier `demo` i.e. it connects the auth account to a SystemUser data access account. You can do this in one of the two following ways:

Using the pre-configured fd-client container in the docker-compose stack

`docker-compose run fd-client fdregister -u=demo:123 -l=demo -c=flockdata`

With the data access account established, you can load some static data into the service. FlockData comes with a data set of countries and capitals. Load this data via the following command    

`docker-compose run fd-client fdcountries -u=demo:123`

You can now navigate to the [Neo4j browser](http://docker-ip:7474) and after logging in with default password of `neo4j` you can run the following Cypher

`match (c:Country) return c` 

## Tracking data in to the service

Other commands that track data in to the service can be found [here](https://github.com/monowai/flockdata.org/tree/master/fd-engine#interacting-with-flockdata)

