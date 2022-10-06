---
layout: rsk
title: RIF Relay Installation Requirements
tags: rif, envelope, relay, user, guide, install
permalink: /guides/rif-relay/install/
---

## Requirements

To have the RIF Relay System running locally there are some tools that are required. All of these tools are open source and have their own support page. The functionality of RIF Relay does not depends on these technologies and could be updated or replaced if its necessary. 

### RSK Node

You need to have a running RSK node version [RSKj](https://github.com/rsksmart/rskj/releases). It is recommended to use any version above IRIS-3.2.0.

> The node can be run locally or using docker.

When using a local RSKj, we can configure the node to connect to a specific network; on this guide we will use Regtest and Testnet. 


### Node & NPM

We use `Node` version `v16.4.0`. 

We recommend the use of [`nvm`](https://github.com/nvm-sh/nvm) and the `.nvmrc` file included at the root of this project, so you can install the correct node version through:

```
nvm use
nvm install
```

If you choose to use node without `nvm`, you can find installation instructions at Node's [site](https://nodejs.org/en/). 

You can check the installation by running `node -v`.

### Truffle

An important tool we use for interacting with the blockchain is `Truffle` version `v5.0.33`.

You can follow the installation guide in the official [site](https://www.trufflesuite.com/truffle).

We run all truffle commands with the prefix `npx`. This is to execute node packages using the project's version.

Checking the install by running `npx truffle version`.

The configuration file is `truffle.js`. Please see Truffle's documentation for details about this file and how to use it.



### Docker

We recommend following the official [documentation](https://docs.docker.com/get-docker/) for installing Docker and keeping it updated.

You'll need to install both `docker` as well as `docker-compose`.

The RIF Relay components can be deployed using Docker or locally, its up to the developer to choose.

#### Running on macOS

To run the project using Docker on a Mac, please follow these steps or the scripts and web apps will not work. 

- Patch `readlink`
The startup scripts assume that GNU's `readlink` command is available. But MacOS ships with BSD's `readlink`, which is incompatible with GNU's version. So we must patch `readlink`. This can be done using [Homebrew](https://brew.sh/) as follows:

```
brew install coreutils
ln -s /usr/local/bin/greadlink /usr/local/bin/readlink
```

After this step, you must make sure that your `PATH` variable gives priority to `/usr/local/bin` over `/usr/bin`. You can check this with `which readlink`, which should output `/usr/local/bin/readlink`. Alternatively try executing `readlink -f .`, if it works you're ok.

### Web3
