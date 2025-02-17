---
layout: rsk
title: Troubleshooting
tags: rsk, rskj, node, faq, troubleshoot
description: "How to solve some known or frequently encountered issues when working with RSKj"
collection_order: 2900
---

This section explains how to solve some known or frequently encountered issues.

If what you need is not in this section, **please contact us** without hesitation through the [RSK Community on Slack](/slack/). We will be happy to help you!

## Index

- ['Discovery can't be started'](#discovery-cant-be-started)
- ['I don't see the logs'](#i-dont-see-the-logs)
- ['Plugin with id 'witness' not found'](#plugin-with-id-witness-not-found)
- ['Truffle doesn't seem to work connected to RSK'](#truffle-doesnt-seem-to-work-connected-to-rsk)
- ['Can't get public IP'](#cant-get-public-ip)
- ['Corrupted blocks after hard fork'](#rewind-blocks)

## Sections

### Discovery can't be started

On Windows, if you start the node and it doesn't do anything, there is a high chance you have a problem with the UDP port of the node.

The UDP port is configured in the node's configuration file, specifically with the value `peer.port`. By default this port is configured to `5050`.

To check if that port is already taken by other application you can follow these steps:

* Open a `cmd` console and run `netstat -ano -p UDP | findstr :5050` (or replace `5050` with the port of your preference).
* You will get a result with the process ID (if any) already using that port for UDP.
* With the process ID (the value at the far right), run this command `tasklist /FI "PID eq processId-you-got"`.

This will let you know which application/service is using this port.

Please make sure the port of your preference is not taken by other application. If so, you need to change the node configuration [as explained here](/rsk/node/configure/#setting-your-own-config-preferences), by overwriting the [peer](/rsk/node/configure/reference/#peer) port option.

> Note: we are considering the possibility of changing this port (5050) in future releases to avoid this problem.

### I don't see the logs

You can configure your own log level, following these [instructions](/rsk/node/configure/verbosity).

### Plugin with id witness not found

If you have this error it's possible that you have missed to run rskj's dependencies.
So please, follow the instructions depending on your operation system:

- [On Windows](/rsk/node/contribute/windows)
- [On Linux](/rsk/node/contribute/linux)
- [On Mac](/rsk/node/contribute/macos)

### Truffle doesn't seem to work connected to RSK

If you can not get `truffle migrate` complete, you will see something like:

```javascript
Writing artifacts to ./build/contracts
Using network 'development'.
Running migration: 1_initial_migration.js
  Deploying Migrations...
  ... 0xc82d661d579e40d22c732b2162734f97aeb13fa095946927cbb8cd896b26a7a3
```

Be sure you are using the right configuration in the `truffle.js` and `truffle-config.js` files.

Remember that you need: **node host**, **node port**, **network_id** and in some cases the **from** (by default Truffle uses the first account in the node). This last one should be an account with positive balance (because it's the one Truffle uses to deploy contract and run transactions) and it should be present between the node's accounts (you can know that by executing the `web3.eth.accounts` command).

So, your config file should be like this:

``` javascript
module.exports = {
    networks : {
        rsk: {
            from : "0xcd2a3d9f938e13cd947ec05abc7fe734df8dd826",
            host : "localhost",
            port : 4444,
            network_id : "*" // Match any network id
        }
    }
};
```

### Can't get public IP

If you get the error:
`Can't get public IP` when you're trying to run your rskj node, the reason is that rskj uses Amazon Check IP service to set the [`public.ip`](/rsk/node/configure/reference/#publicip) parameter.

To solve it, you need to change the `public.ip` key in config file with your IP address (if you don't know your IP, simply [search for it](https://www.google.com/search?q=what's+my+IP+address)).

[This page](/rsk/node/configure) will show you how to change a node's configuration file.

### Rewind Blocks

This tool should be used in a scenario where an RSK node processes blocks that are corrupted or invalid, for example after a hard fork. It allows one to remove such blocks and start from a previously known state. It does so by removing the blocks with block number higher than the block number parameter command line argument.

> Note: The node must be turned off before the rewind, and restarted after.

Example:

`java -cp rsk-core-<VERSION>.jar co.rsk.cli.tools.RewindBlocks 1000000`

The above command removes the blocks with number 1000001 or higher.

### DbMigrate: Migrate between databases

This tool allows the user to migrate between different supported databases such as `rocksdb` and `leveldb`. 

#### How to use

To use the `DbMigrate` tool to migrate between databases, we will need a tool class and CLI arguments.

The tool class is: `co.rsk.cli.tools.DbMigrate`

Required CLI arguments:

`args[0]` - database target where we are going to insert the information from the current selected database.
> Note: You cannot migrate to the same database or an error will be thrown. It is highly recommended to turn off the node in order to perform the migration since latest data could be lost.


Example migrating from `leveldb` to `rocksdb`:

`java -cp rsk-core-<VERSION>.jar co.rsk.cli.tools.DbMigrate rocksdb`
