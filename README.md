# Asteroid Rush Docker

Docker with nodeos, keos and history api plugin for participating in Asteroid Rush blockchain.

## Getting Started

Current repository contains dockerfiles for building services neccessary for using Asteroid Rush blockchain.  
Services:  
1. Nodeos  
2. Keos  
3. MongoDB API

### Prerequisites

#### Docker
docker version >= 18.03  
docker-compose version >= 1.21.0


#### SSL
If you want to provide public api to your node, we recommend to add ssl certificate for public api and use http api only locally. Most simple way to retrieve ssl certificate is using of certbot.  
To install it and get more instructions about installation process for your OS look at [official site](https://certbot.eff.org/)  
Then follow instruction
```
echo "HTTPS_CERTS_DIR=[path to your certificates directory]" >> .env
```

#### Skipping SSL
If you don't want to support SSL or test nodeos locally, you should comment or delete parameter below in dockerfiles/Dockerfile.Node and rebuild it
```
--https-server-address=0.0.0.0:8010
```

### Installing

Before building of images, you need to fix version of repository to build.  
For example
```
echo "VERSION=v0.8.0" >> .env
```

First configure your config.ini file, start with copying of example file.
```
cp config.ini.example config.ini
```

#### Configuration

You should change main settings, another configs are optional.
```
# Peers

# Uncomment p2p-peer-address or add another one addresses
p2p-peer-address = node1.asteroidrush.io:8212
p2p-peer-address = node2.asteroidrush.io:8212

# Memory management

# We recommend to set value below, but you may set any what you want
# Maximum size (in MiB) of the chain state database (eosio::chain_plugin)
chain-state-db-size-mb = 3072


# SSL configs
# If you want to enable SSL for your public api, you should set values of variables below with path related to /certs directory
# https-certificate-chain-file =
# https-private-key-file =

```

#### History plugin


For history we are using original mongo_db_plugin.  
More information about configurations you may find at [official docs](https://developers.eos.io/eosio-nodeos/docs/mongo_db_plugin)
```
plugin = eosio::mongo_db_plugin


# MongoDB configs

mongodb-uri = mongodb://mongodb:27017/EOS
mongodb-filter-on = *
mongodb-filter-out = eosio:onblock:
mongodb-queue-size = 2048
mongodb-block-start = 1
mongodb-store-block-states = false
mongodb-store-blocks = false
mongodb-store-transactions = false
mongodb-store-transaction-traces = true
mongodb-store-action-traces = true


```


### Build

Next you should build base images
```
docker build -f dockerfiles/Dockerfile.Builder -t asteroid_rush/builder .
docker build -f dockerfiles/Dockerfile.Base -t asteroid_rush/base:[version] .
```

After that you may build neccessary images for your services.
```
docker-compose build nodeos keos mongodb-api
```

### Run

docker-compose.yml contains next services:  
1. nodeos - starts node with existing state  
2. nodeos-clean - starts clean node without state with genesis.json  
3. keos - starts keos (wallet server)  
4. mongodb-api - starts mongodb-api  

To start any of that services you should use command
```
docker-compose up [service name] # To run service foreground
# or 
docker-compose start [service name] # To run service in background
```

### History API

MongoDB history api image based on [Cryptolions History API](https://github.com/CryptoLions/EOS-mongo-history-API)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details