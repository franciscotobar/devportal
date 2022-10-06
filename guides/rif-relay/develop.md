---
layout: rsk
title: RIF Relay Develop
tags: rif, envelope, relay, user, guide, develop
permalink: /guides/rif-relay/develop/
---


## Initializing the project

To use RIF Relay, follow these steps to build the project.

## Project structure

The project is divided in 4 modules that interact with each other. Each project has their own documentation on their repository. We also developed a SDK to simplify the integration process.

1. [RIF Relay Contracts](https://github.com/rsksmart/rif-relay-contracts)
2. [RIF Relay Common](https://github.com/rsksmart/rif-relay-common)
3. [RIF Relay Client](https://github.com/rsksmart/rif-relay-client)
4. [RIF Relay Server](https://github.com/rsksmart/rif-relay-server)
5. [RIF Relay SDK](https://github.com/rsksmart/rif-relay-sdk)



## Committing changes

To contribute to the project, create a branch with the name of the new feature you are implementing (e.g. `gas-optimization`). When you commit to git, a hook is executed. The hook executes a linter and all the tests.

## Troubleshooting

Common errors:

#### Unknown network "NA". See your Truffle configuration file for available networks.

Verify that the network that you are trying to connect is included in the truffle.js file in the RIF Relay Contracts module.
