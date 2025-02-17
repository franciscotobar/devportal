---
layout: rsk
title: RIF Name Service
tags: rif, rns, rif-name-service, node, sdk, libraries, infrastructure, protocols, mvp, design, rbtc, defi, decentralized, quick-start, guides, tutorial, networks, dapps, tools, rootstock, rsk, ethereum, smart-contracts, install, get-started, how-to, mainnet, testnet, contracts, wallets, web3, crypto
---

RNS provides an architecture which enables the identification of blockchain addresses by human-readable names.

{% include rns-register.html %}

<div class="container the-stack">
  <div class="row rif_blue_text">
    <div class="col">
      <div class="rns-index-box">
        <a href="try-rns">Try the service</a>
        <br />
        <br />
        <p>Register a domain in the Testnet, for free.</p>
      </div>
    </div>
    <div class="col">
      <div class="rns-index-box">
        <a href="integrate">Integrate with RNS</a>
        <br />
        <br />
        <p>Easy guides on how to integrate RNS in your solution.</p>
      </div>
    </div>
  </div>
  <div class="row rif_blue_text">
    <div class="col">
      <div class="rns-index-box">
        <a href="run-locally">Develop on top of RNS</a>
        <br />
        <br />
        <p>Deploy RNS suite in your local development environment</p>
      </div>
    </div>
    <div class="col">
      <div class="rns-index-box">
        <a href="libs">Use the libraries</a>
        <br />
        <br />
        <p>Use simple libraries to interact with RNS service.</p>
      </div>
    </div>
  </div>
</div>

## The stack

<div class="container the-stack">
  <div class="row has-unique-col">
    <div class="col">
      <div class="row rotate defi"><a href="/rif/rns">RNS</a></div>
      <div class="row rsk_blue dapps">
        <div class="col"><span><a href="/rif/rns/operations">dApp</a></span></div>
        <div class="col"><span><a href="/rif/rns/libs">Libraries</a></span></div>
        <div class="col"><span><a href="/rif/rns/architecture">Smart contracts</a></span></div>
        <div class="col"><span><a href="/rif/rns/specs">Specifications</a></span></div>
      </div>
    </div>
  </div>
</div>

## Motivation

By adding a name resolution service, also known as “alias”, the probability of errors is significantly reduced. In addition, centralizing the access to multiple resources associated with a human-readable name improves the blockchain platform user experience. As resource names may change over time, the system needs to be flexible to support frequent changes.

Currently over the World Wide Web, the Domain Name System (DNS) is responsible for mapping human-readable names to IP addresses. RNS is a decentralized and secure service that works over RSK's blockchain.

## Design

RNS is a hierarchical name space inspired by DNS, with the hierarchy roughly corresponding to organizational structure, and uses the "." character to delimit its levels.

The design goals of the RIF Name Service influence its structure.

- The primary goal is a consistent name space which will be used for referring to resources.
- All data associated with a name is tagged with a type, and queries can be limited to a single type.
- Because we want the name space to be useful in dissimilar networks and applications, we provide the ability to use the same name space with different protocol families or management. The RNS tags all data with a class as well as the type, so that we can allow parallel use of different formats for data of type address.
- There may be tradeoffs between the cost of acquiring data, the speed of updates, and the accuracy of caches. The owner of the domain, who is the source of the data, should weigh these tradeoffs, and decide what to store, and how to cache.

[RNS specs](specs)

## Elements of the RNS

RNS has two major components:

- **The RNS Registry**, which is specification for a tree structured name space and data associated with the names.

  Conceptually, each node and leaf of the domain name space tree names a set of information, and query operations are attempts to extract specific types of information from a particular set. A query names the domain name of interest and describes the type of resource information that is desired.

  [Specs](specs/registry)

  [Implementation](architecture/registry)

- **RNS Resolvers** are contracts that provide information from a name in response to client requests.

  Resolvers must be able to answer a query directly, or pursue the query using referrals to other resolvers. A resolver will typically be a contract's public function that is directly accessible to user programs or other contracts; hence no protocol is necessary between the resolver and the user program.

  [Specs](specs/resolvers)

These three components roughly correspond to the three layers or views of the domain system:
- From the user's point of view, the domain system is accessed through a simple [resolution operation](operations/resolve). The domain space consists of a single tree and the user can request information from any section of the tree.
- From the resolver's point of view, the domain system is composed of an unknown number of names. Each name has a corresponding resolver that provides information for a set of resolution types directly.
- From the registry's point of view, the domain system consists of a [hierarchical tree](architecture/registry) where each leaf has an owner (contract or account) and an associated resolver that provides information of the name.

## Guidelines on use

Before RNS can be used to hold naming information for some kind of object, two needs must be met:
- A convention for mapping between object names and domain names. This describes how information about an object is accessed. Find specs [here](specs#name-mapping-convention)
- Resource record types and data formats for describing the object. Find specs [here](specs/resolvers).

The guideline for finding a specific record for a name is as follows:
1. Calculate the name identifier with [`namehash` function](specs#name-mapping-convention).
2. Get the name's resolver address via [`resolver(bytes32)`](specs/registry#access).
3. Determine if resolver supports desired resource record via [ERC-165 interface detection](https://eips.ethereum.org/EIPS/eip-165).
4. Get the desired resource record. Find currently standardized resolvers [here](specs/resolvers).

> Guidelines on [integration](integrate)

### Resource records

A domain name identifies a node. Each node has a set of resource information, which may be empty. The set of resource information associated with a particular name is composed of separate resource records (RRs). The order of RRs in a set is not significant. Resource records associated with a name are found in the domain's resolver
