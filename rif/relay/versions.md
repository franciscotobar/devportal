---
layout: rsk
title: RIF Relay Versions
tags: rif, envelope, relay, gas station network, gsn, versions, changelog
permalink: /rif/relay/versions/
---

The first iteration of RIF Relay was based on the great work done by the [Gas Station Network team](https://www.opengsn.org/).

## Version 0.1

RIF Relay V0.1 started as a fork of GSN with two goals in mind:

- Be compatible with existing and future smart contracts without requiring such contracts to be adapted to work with RIF Relay.
- Be as cost effective as possible.

## Version 0.2

### Overview

RIF Relay V0.2 is a redesign of GSN. It reduces gas costs and simplifies the interaction between the different contracts that are part of the system. It achieves this by:

- Securely deploying counterfactual Smart Wallet proxies for each user account: this eliminates the need for relying on `_msgSender()` and `_msgData()` functions, making existing and future contracts compatible with RIF Relay without any modification.
- Allowing relayers to receive tokens in a worker address under their control and decide what to do with funds later on.
- Reducing gas costs by optimizing the GSN architecture.

Our main objective is to provide the RSK ecosystem with the means to enable blockchain applications and end-users (wallet-apps) to pay for transaction fees using tokens (e.g. RIF tokens), and thereby remove the need to acquire RBTC in advance.

It is important to recall that - as a security measure - version 1 contracts deployed on Mainnet have limits on the staked amounts to operate; these limits were removed in version 2.

### Details

* RelayHub contract no longer receives payments, the payment for the service (in tokens) is now sent directly to the worker relaying the transaction on behalf of the user.
* RelayHub contract now handles relay manager staking.
* Gas estimation improvements:
    * Gas overhead removed from RelayHub; there are no more validations against hardcoded values.
    * The gas and token gas fields from the request can now be left undefined, and in that case, they will automatically be estimated by the RIF Relay Client.
    * The maximum gas estimation in the RIF Relay Server is more precise now.
    * A new utility function is available to estimate the maximum gas a relay transaction would consume, based in a linear fit estimation. This can be used in applications that don't want to sign a payload each time they need an approximation of the cost of relaying the transaction.
* Paymaster verifications are done off-chain to optimize gas costs, thus the paymasters are now called Verifiers and they are not part of the on-chain relay flow nor do they handle payments at all.
* Gas cost optimization.
* Security issues.


## Version 1

### Overview

Including a revenue-sharing mechanism to the RIF Relay service should not introduce a price penalty to the RIF Relay users. With this restriction in mind, we proposed to modify the address used to pay (in tokens) for a successful transaction relay (or deploy). 

In its current implementation, the RelayRequest and the DeployRequest include a relayWorker attribute to identify which account paid for the gas. The RIF Relay SmartWallet pays directly to this account the number of tokens negotiated for the Relay (or Deploy) service. The relayWorker attribute can be replaced with the address of a Smart Contract containing the revenue sharing logic. 

This change will not alter the current relay flow, keeping its cost as it is today, and it will also introduce the flexibility to implement any revenue-sharing strategy needed.

* At the moment, there’s no enforced logic on what address the relayWorker attribute from a RelayRequest or DeployRequest can be. We propose for it to be a MultiSig contract address, with all the fee distribution logic. 
* The relayWorker attribue will be removed from the RelayRequest and DeployRequest. A new attribute called feesReceiver will be implemented and configured in the RelayServer:
    * The feesReceiver could be the worker or MultSig contract. 
* The SmartWallet will pay to the feesReceiver. The feesReceiver will hold the funds of each user payment.
* Upon payment from the SmartWallet, the feesReceiver will not perform any distribution logic to avoid increasing the cost of the relay service for the user.
* With this approach, no changes in the relay flow are required, and thus the introduction of a revenue-sharing mechanism will not impact the price of the relay service.
* The relevant participants will form part of the MultiSig contract.
    *  The MultiSig contract will specify how much of the funds collected go to each participant (e.g., the relayServer operator, the wallet provider, the liquidity provider, and IOVlabs could be the participants).
    *   At a later time, an off-chain process will trigger the distribution process from the contract. This process can invoke the distribution function once per week, month, or when the funds in the contract surpass a certain threshold.
    *  The participants can modify the sharing parameters (e.g., the percentages used for the distribution among the participants) if they agree on the particular changes.
    * Multiple Revenue Sharing Strategies can exist, ideally one per group of participants.

