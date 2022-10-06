---
layout: rsk
title: RIF Relay Deployment
description: RIF Relay deployment process
tags: rif, envelope, relay, user, guide, deployment
render_features: 'collapsible tables-with-borders'
permalink: /guides/rif-relay/deployment/
---

[](#top "collapsible")
- Regtest
  * Below are the steps to start using RIF Relay locally. All the components will be interacting with the RSKj node that was configured to use Regtest. The executed scripts used the  **account[0]** from the local node. 

  [](#top "collapsible")
  - 1) Contracts Deployment
    * We will begin by deploying the on-chain components. All the necessary tools to do the deployments were included in [RIF Relay Contract](https://github.com/rsksmart/rif-relay-contracts). 

      To start the process just execute the following command:

      ```
      npx truffle migrate --network regtest
      ```

      > Note: The migrate command is using the Regtest configuration that was previously configured in the truffle.js file.

      After the deployment we will see the summary of the deployed contracts. Most of this contracts are the on-chain components that will be used by RIF Relay and some are for testing purposes.


      ```
      |===================================|============================================|
      | Contract                          | Address                                    |
      |===================================|============================================|
      | Penalizer                         | 0x83C5541A6c8D2dBAD642f385d8d06Ca9B6C731ee |
      | RelayHub                          | 0xE0825f57Dd05Ef62FF731c27222A86E104CC4Cad |
      | Smart Wallet Contracts ========================================================|
      | SmartWallet                       | 0x73ec81da0C72DD112e06c09A6ec03B5544d26F05 |
      | SmartWalletFactory                | 0x03F23ae1917722d5A27a2Ea0Bcc98725a2a2a49a |
      | SmartWalletDeployVerifier         | 0x1eD614cd3443EFd9c70F04b6d777aed947A4b0c4 |
      | SmartWalletRelayVerifier          | 0x5159345aaB821172e795d56274D0f5FDFdC6aBD9 |
      | Custom Smart Wallet Contracts =================================================|
      | CustomSmartWallet                 | 0x7557fcE0BbFAe81a9508FF469D481f2c72a8B5f3 |
      | CustomSmartWalletFactory          | 0x0e19674ebc2c2B6Df3e7a1417c49b50235c61924 |
      | CustomSmartWalletDeployVerifier   | 0x8901a2Bbf639bFD21A97004BA4D7aE2BD00B8DA8 |
      | CustomSmartWalletRelayVerifier    | 0x1Af2844A588759D0DE58abD568ADD96BB8B3B6D8 |
      | Testing Contracts =============================================================|
      | SampleRecipient                   | 0xCd5805d60Bbf9Afe69a394c2BDa10F6Dae2c39AF |
      | TestToken                         | 0x726ECC75d5D51356AA4d0a5B648790cC345985ED |
      |===================================|============================================|
      ```

      As we can see we have two sets of Smart Wallets with their own verifiers. This is because each verifier use the factory to do deploy and relay validations. For testing purposes we will be using the Smart Wallet Contracts. 

  - 2) Collector Contract
    * In principle, any number of Collector contracts can be deployed and used. This does not automatically occur alongside the deployment of the rest of the Relay contracts during the execution of migrations since usage of the Revenue Sharing feature is optional. For an overview of the Collector contract please see [the corresponding architecture document section](/rif/relay/architecture/#collector).

      Before deploying a Collector contract please make sure that:
      1. The token chosen for this contract matches the one used for relayed transaction fees. **Any tokens other than the one set at the time of the Collector deployment which are sent to the contract will be impossible to retrieve, including the native token**. 
      2. The owner of the Collector contract has been chosen (be it a multisig contract address, a regular contract address or an EOA). This does not need to match the address deploying the contract, but please note that if there are no means of signing Collector contract function calls with the owner address **all revenue sharing funds will be permanently locked in the contract**.
      3. Partners and their shares have been defined correctly. This means that beneficiary shares must collectively add up to 100. As with the previous point, any tokens sent to an address without its private key held by its beneficiary will be lost. For an example of a structurally valid revenue shares definition see [this deploy collector task input sample](https://github.com/rsksmart/rif-relay-contracts/blob/PP-94/revenue-sharing-model/deploy-collector.input.sample.json).

    * To deploy the Collector contract we will be using the [RIF Relay Contract](https://github.com/rsksmart/rif-relay-contracts) and [Safe contracts v1.2.0](https://github.com/gnosis/safe-contracts/tree/v1.2.0).

      In the safe contracts project we will deploy the contracts in the Regtest network. 
      ```
      yarn truffle deploy --network regtest
      ```
      > Take in consideration that the Regtest network needs to be added in the truffle.js file.

      After the deployment from the safe contracts, we need to switch back to the RIF Relay Contracts repository. In this repository there are some scripts that we will need to execute. Prior executing the scripts we need to update the tasks/utils.js with the contract addresses from the safe contracts deployment:

        | Name                    | Mapped                 |
        | ----------------------- | ---------------------- |
        | safeMasterCopyAddress   | GnosisSafe             |
        | safeProxyFactoryAddress | GnosisSafeProxyFactory |
        | multiSendAddress        | MultiSend              |
   
      Now we need to execute the following command, to deploy the multisig contract:

        ```
        npx truffle --network regtest exec tasks/create-safe.js --owners="0x7986b3DF570230288501EEa3D890bd66948C9B79,0x0a3aA774752ec2042c46548456c094A76C7F3a79,0xCF7CDBbB5F7BA79d3ffe74A0bBA13FC0295F6036,0x39B12C05E8503356E3a7DF0B7B33efA4c054C409"
        ```
        > All this accounts are the first 4 accounts from the regtest blockchain that are provided by default. 

      The last step is to deploy the collector contract. To do that we need to create a configuration file with the following structure and save it as deploy-collector.input.json:

      ```json
      {
        "collectorOwner": "0xfC6B663863722F563a9a3234E15493d2b7c365E8",
        "partners": [
            {
                "beneficiary": "0x7986b3DF570230288501EEa3D890bd66948C9B79",
                "share": 20
            },
            {
                "beneficiary": "0x0a3aA774752ec2042c46548456c094A76C7F3a79",
                "share": 35
            },
            {
                "beneficiary": "0xCF7CDBbB5F7BA79d3ffe74A0bBA13FC0295F6036",
                "share": 13
            },
            {
                "beneficiary": "0x39B12C05E8503356E3a7DF0B7B33efA4c054C409",
                "share": 32
            }
        ],
        "tokenAddress": "0x726ECC75d5D51356AA4d0a5B648790cC345985ED",
        "remainderAddress": "0xc354D97642FAa06781b76Ffb6786f72cd7746C97"
      }
      ```

      Now we only need to execute the following command:

      ```
      npx truffle --network regtest exec tasks/deploy-collector.js --collectorConfig="deploy-collector.input.json" --outputFile="output.json"
      ```

      The collector is ready and can start receiving fees. 

  - 3) Allow Tokens
    * RIF Relay only accepts whitelisted tokens. Locally there is a script that can be use to whitelist the tokens.

      ```
      npm run allowTokens 0x726ECC75d5D51356AA4d0a5B648790cC345985ED --network regtest
      ```
      > The allowTokens script assumes that you are in Regtest and uses the account[0] as the owner of the contracts and that’s important because only the owner can allow tokens on the contracts.

      There are multiple reasons to just allow whitelisted tokens but the main one is to ensure that RIF Relay is just accepting tokens that represent value for the sponsor.

  - 4) Run RIF Relay Server
    * Once we have all the on-chain components, we will continue with the off-chain components. This process is done with the [RIF Relay Server](https://github.com/rsksmart/rif-relay-server).

      Prior starting the Relay Server, we need to update the configuration file, we save the configuration file as **server-config.json**.  Below is an example using the RSKj node locally and  the contracts that we just deployed.


      ```json
      {
        "url": "localhost",
        "port": 8090,
        "relayHubAddress": "0xE0825f57Dd05Ef62FF731c27222A86E104CC4Cad",
        "relayVerifierAddress": "0x5159345aaB821172e795d56274D0f5FDFdC6aBD9",
        "deployVerifierAddress": "0x1eD614cd3443EFd9c70F04b6d777aed947A4b0c4",
        "gasPriceFactor": 1,
        "rskNodeUrl": "http://127.0.0.1:4444",
        "devMode": true,
        "customReplenish": false,
        "logLevel": 1,
        "workdir": "/home/user/workspace/relay"
      }
      ```

      > The Relay and Deploy verifier are using the contracts from the Smart Wallet section in the summary.

      Each key of the file have the following meaning:

      - **url**: is the URL where the relay server will be deployed, it could be localhost or the IP of the host machine.
      - **port**: the port where the relay server will be hosted.
      - **relayHubAddress**: is the relay hub contract address.
      - **relayVerifierAddress**: is the relay verifier contract address.
      - **deployVerifierAddress**: is the deploy verifier contract address.
      - **gasPriceFactor**: is the gas price factor used to calculate the gas on the server, you can leave it as 1.
      - **rskNodeUrl**: is the RSK node endpoint URL, where the RSK node is located.
      - **devMode**: it indicates to the server if we are in development mode or not.
      - **customReplenish**: set if the server uses a custom replenish function or not.
      - **logLevel**: is the log level for the relay server.
      - **workdir**: is the absolute path to the folder where the server will store the database and all its data.

      Now we can start the server with the following command:

      ```
      npm run start
      ```
      > By default the server will use the file server-config.json in the root directory.

      > The start command can receive the path of the configuration file as parameter, more information can be found in the [RIF Relay Server](https://github.com/rsksmart/rif-relay-server#system-usage) documentation.

      At this point the server should be running and ready to start processing transactions, however, we still need to register all the off-chain components in the Relay Hub. To do the registration process, the following command needs to be executed:

      ```
      npm run register
      ```
      > The command needs to be execute in a different terminal since it needs the server to be running to perform the registration.

      > The register command can receive multiple parameters to simplify the interaction, more information can be found in the [RIF Relay Server](https://github.com/rsksmart/rif-relay-server#server-registration) documentation.

      The register process performs the following actions:
      - Stake the Relay Manager
      - Add the Relay Worker
      - Register the Relay Server

      Now the server is ready to start processing transactions and a ready message will be on the console.

- Testnet
  * Below are the steps to start using RIF Relay on Testnet. All the components will be interacting with the RSK Testnet (https://public-node.testnet.rsk.co). 

  [](#top "collapsible")
  - 1) Contracts Deployment
    * On Testnet the [contracts](https://developers.rsk.co/rif/relay/contracts/) are already deployed or you can deploy your own contracts. 

    * All the necessary tools to do the deployments were included in [RIF Relay Contract](https://github.com/rsksmart/rif-relay-contracts). To start the process on Testnet we need an account with funds. The funds retreival con be done from the [tRBTC Faucet](https://faucet.rsk.co/).

      ```
      npx truffle migrate --network testnet
      ```
      > Take in consideration that the testnet network needs to be configured in the truffle.js file.

  - 2) Collector Contract
    * In principle, any number of Collector contracts can be deployed and used. This does not automatically occur alongside the deployment of the rest of the Relay contracts during the execution of migrations since usage of the Revenue Sharing feature is optional. For an overview of the Collector contract please see [the corresponding architecture document section](/rif/relay/architecture/#collector).

    Before deploying a Collector contract please make sure that:
      1. The token chosen for this contract matches the one used for relayed transaction fees. **Any tokens other than the one set at the time of the Collector deployment which are sent to the contract will be impossible to retrieve, including the native token**. 
      2. The owner of the Collector contract has been chosen (be it a multisig contract address, a regular contract address or an EOA). This does not need to match the address deploying the contract, but please note that if there are no means of signing Collector contract function calls with the owner address **all revenue sharing funds will be permanently locked in the contract**.
      3. Partners and their shares have been defined correctly. This means that beneficiary shares must collectively add up to 100. As with the previous point, any tokens sent to an address without its private key held by its beneficiary will be lost. For an example of a structurally valid revenue shares definition see [this deploy collector task input sample](https://github.com/rsksmart/rif-relay-contracts/blob/PP-94/revenue-sharing-model/deploy-collector.input.sample.json).

    * 



  - 3) Allow Tokens
    * RIF Relay only accepts whitelisted tokens. To whitelist a token on Testnet we need to execute the `acceptToken(address token)` on the Relay Verifiers:

      * `SmartWalletDeployVerifier`
      * `SmartWalletRelayVerifier`
      
      > The contract execution needs to be done by the owner that is the account that did the deployment.

      There are multiple reasons to just allow whitelisted tokens but the main one is to ensure that RIF Relay is just accepting tokens that represent value for the sponsor.

  - 4) Run RIF Relay Server
    * Once we have all the on-chain components, we will continue with the off-chain components. This process is done with the [RIF Relay Server](https://github.com/rsksmart/rif-relay-server).

      Prior starting the Relay Server, we need to update the configuration file, we save the configuration file as **server-config.json**.  Below is an example using the RSKj node locally and  the contracts that we just deployed.

      ```json
      {
        "url": "localhost",
        "port": 8090,
        "relayHubAddress": "0xe90592939fE8bb6017A8a533264a5894B41aF7d5",
        "relayVerifierAddress": "0xDe988dB9a901C29A9f04050eB7ab08f71868a8fc",
        "deployVerifierAddress": "0x345799D90aF318fd2d8CbA87cAD4894feF2f3518",
        "gasPriceFactor": 1,
        "rskNodeUrl": "https://public-node.testnet.rsk.co/",
        "devMode": true,
        "customReplenish": false,
        "logLevel": 1,
        "workdir": "/home/user/workspace/relay"
      }
      ```

      > The [contracts](https://developers.rsk.co/rif/relay/contracts/#primary-contracts) that are been used, are the primary one that does not have support for CustomSmartWallet.

      Each key of the file have the following meaning:

      - **url**: is the URL where the relay server will be deployed, it could be localhost or the IP of the host machine.
      - **port**: the port where the relay server will be hosted.
      - **relayHubAddress**: is the relay hub contract address.
      - **relayVerifierAddress**: is the relay verifier contract address.
      - **deployVerifierAddress**: is the deploy verifier contract address.
      - **gasPriceFactor**: is the gas price factor used to calculate the gas on the server, you can leave it as 1.
      - **rskNodeUrl**: is the RSK node endpoint URL, where the RSK node is located.
      - **devMode**: it indicates to the server if we are in development mode or not.
      - **customReplenish**: set if the server uses a custom replenish function or not.
      - **logLevel**: is the log level for the relay server.
      - **workdir**: is the absolute path to the folder where the server will store the database and all its data.

      Now we can start the server with the following command:

      ```
      npm run start
      ```
      > By default the server will use the file server-config.json in the root directory.

      > The start command can receive the path of the configuration file as parameter, more information can be found in the [RIF Relay Server](https://github.com/rsksmart/rif-relay-server#system-usage) documentation.

      At this point the server should be running and ready to start processing transactions, however, we still need to register all the off-chain components in the Relay Hub. To do the registration process, the Relay Manager and Worker must have funds.

      To obtain the addresses we need to execute the command:
      ```
      curl http://<SERVER_URL>/getaddr
      ```
      > The command needs to be execute in a different terminal since it needs the server to be running to perform the request.

      ```json
      {
        "relayWorkerAddress": "0xabf898bd73b746298462915ca91623f5630f2462",
        "relayManagerAddress": "0xa71d65cbe28689e9358407f87e0b4481161c7e57",
        "relayHubAddress": "0xe90592939fE8bb6017A8a533264a5894B41aF7d5",
        "minGasPrice": "6000000000",
        "chainId": "31",
        "networkId": "31",
        "ready": false,
        "version": "2.0.1"
      }
      ```

      Send at least 0.001 tRBTC to the Worker and Manager.

      Now execute the command the register command.


      ```
      npm run register --network <RSKJ_NODE_URL> --hub <RELAY_HUB_CONTRACT_ADDRESS> -m <secret_mnemonic> --from <ADDRESS>  --funds <FUNDS> --stake <STAKE> --relayUrl <RELAY_URL>
      ```
      > The register command can receive multiple parameters to simplify the interaction, more information can be found in the [RIF Relay Server](https://github.com/rsksmart/rif-relay-server#server-registration) documentation.


      The register process performs the following actions:
      - Add the Relay Worker
      - Register the Relay Server

      Now the server is ready to start processing transactions and a ready message will be on the console.

- Mainnet
  * TBD