# FDC suite deployment

## Overview

This repository contains docker-compose files and configuration files for verifiers and blockchain nodes that are required to run a FDC client.

Bitcoin, Dogecoin and Ripple use an indexer that creates a local database with data from the blockchain. This is then exposed via api by verifier api server.

EVM based chains (Ethereum, Flare, Songbird) use a verifier api server that directly queries the rpc node.

- Blockchain nodes:
    - Bitcoin - [flarefoundation/bitcoin](https://hub.docker.com/r/flarefoundation/bitcoin)
    - Dogecoin - [flarefoundation/dogecoin](https://hub.docker.com/r/flarefoundation/dogecoin)
    - Ripple - [flarefoundation/rippled](https://hub.docker.com/r/flarefoundation/rippled)
    - Ethereum - [ethereum/client-go](https://hub.docker.com/r/ethereum/client-go) and [prysm](https://docs.prylabs.network/docs/install/install-with-docker)

- Indexers and verification servers for:
    - BTC - [flare-foundation/verifier-utxo-indexer](https://github.com/flare-foundation/verifier-utxo-indexer) and [flare-foundation/verifier-indexer-api](https://github.com/flare-foundation/verifier-indexer-api)
    - DOGE - [flare-foundation/verifier-utxo-indexer](https://github.com/flare-foundation/verifier-utxo-indexer) and [flare-foundation/verifier-indexer-api](https://github.com/flare-foundation/verifier-indexer-api)
    - XRP - [flare-foundation/verifier-xrp-indexer](https://github.com/flare-foundation/verifier-xrp-indexer) and [flare-foundation/verifier-indexer-api](https://github.com/flare-foundation/verifier-indexer-api)

Bitcoin, Dogecoin and Ripple use an indexer that creates a local database with data from the blockchain. This is then exposed via api by verifier api server.

- EVM verifier - [flare-foundation/evm-verifier](https://github.com/flare-foundation/evm-verifier)

## Hardware Requirements

The minimal hardware requirements for a complete `testnet` configuration are:

- CPU: 8 cores @ 2.2GHz
- DISK: 100 GB SSD disk
- MEMORY: 8 GB

The minimal hardware requirements for a complete `mainnet` configuration are:

- CPU: 16/32 cores/threads @ 2.2GHz
- DISK: 4 TB NVMe disk
- MEMORY: 64 GB

If you don't want to deploy everything on a single server, separate components can be deployed on different servers. In that case the requirements for a single server can be lower.

## Software Requirements

The Attestation Suite was tested on Debian 12 and Ubuntu 22.04.

Additional required software:

- *Docker* version 24.0.0 or higher
- *Docker Compose* version 2.18.0 or higher

## Prerequisites

- A machine(s) with `docker` and `docker compose` installed.
- A deployment user in the `docker` group.
- The Docker folder set to a mount point that has sufficient disk space for Docker volumes. The installation creates several Docker volumes.

## Step 1 Clone deployment Repository

``` bash
git clone https://github.com/flare-foundation/fdc-suite-deployment.git
cd fdc-suite-deployment

```

### 1.1 (Optional) Build docker images

Docker images are automatically built and published to github container registry. By default the deployment will download the images automatically. If you need to build them manually clone the required git repository (linked in the overview of this readme), and run:

``` bash
docker build -t <image-tag> .
```

replace image tag with the tag that is used `docker-compose.yaml` files that use this image.

## Step 2 Configuration

### 2.1 Configuring blockchain nodes

#### BTC

The only required configuration is setting the authentication for the node. To generate a password for admin user run:
``` bash
cd nodes-mainnet/btc
./generate-password.sh
```
example output:
```
password: c021cae645db6d3371b26ced94c8d17a5d9f3accbf3591d8b4c0be19623e5662
String to be appended to bitcoin.conf:
rpcauth=admin:a0956d81a2344f1602d9ed7b82ef3118$2caf19c9cf27937f728f600fc14e8db97f80218d727e331a57c3cfc55b3e17fe
Your password:
c021cae645db6d3371b26ced94c8d17a5d9f3accbf3591d8b4c0be19623e5662
```

or configure the username and password manually:

``` bash
./rpcauth.py <USERNAME> <PASSWORD>
```

#### DOGE

Configuration works like BTC.

For example, to generate a password for admin user run:
``` bash
cd nodes-mainnet/doge
./generate-password.sh
```

#### XRP

Default configuration doesn't need any additional configuration.

#### ETH

Configure the jwt.hex for authentication. Create the file `nodes-mainnet/eth/jwt.hex`. Or generate the password randomly:
``` bash
openssl rand -hex 32 > nodes-mainnet/eth/jwt.hex
```

### 3.2 Simple configuration for indexers and verifiers

For a simple configuration the only file that needs to be edited is `.env` file in the root of this repository. Copy `.env.example` file to `.env` and edit it.

For rpc nodes, fill in the authentication data you created in the previous step. If you run blockchain nodes and verifiers on the same server, you can use the ip `172.17.0.1` to reach the nodes.

Indxers will start indexing the blockchain with the block number configured in `_START_BLOCK_NUMBER` variables. This needs to be set the first time when you start the indexers to avoid indexing too much data. FDC requires at least 14 days of history, so pick a block number that was finalized 14 days ago. On later restarts indexers will start indexing from the latest block in the database.

Set `TESTNET` to `true` if you are running verifiers for testnets.

Set `VERIFIER_API_KEYS` to api keys that will have access to verifier api servers. One or multiple comma separated keys can be configured. You will likely need at least one key for FDC client that will call verifier api servers.

`*_DB_PASSWORD` variables are used internally for the indexer database. If you don't know you probably don't need to access the database, so set those passwords to a random string.

### 3.3 Generating configs

from the root of this repo, run `./generate-config.sh`

This script uses the values from `.env` and generates config files from `*.example` files in directories:

- verifiers/btc/
- verifiers/doge/
- verifiers/xrp/
- evm-verifier/

## Step 4 Running

### 4.1 Starting blockchain nodes

cd into correct directory (example `nodes-mainnet/btc`) and run `docker compose up -d`.

Do this for all blockchain nodes you plan to run on the current server.

### 4.2 Starting indexers and verifiers 

cd into correct directory (example `verifiers/btc`) and run `docker compose up -d`.

Do this for all verifiers you plan to run on the current server.
