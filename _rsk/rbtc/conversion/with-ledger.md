---
layout: rsk
title: Conversion using a Ledger hardware wallet
tags: rsk, rbtc, conversion, peg, 2-way, peg-in, peg-out, federation, ledger
description: 'How to perform the Powpeg mechanism using Ledger'
collection_order: 3140
---

In this section we will go over the steps of converting BTC to RBTC using Ledger hardware wallet, and vice versa on the Bitcoin and RSK networks.

- [General requirements](#general-requirements)
- [BTC to RBTC conversion](#btc-to-rbtc-conversion)
- [RBTC to BTC conversion](#rbtc-to-btc-conversion)

## General Requirements

- You need a [Ledger](https://www.ledger.com/) with Bitcoin and
  RSK Apps installed. We recommend you to have
  [Ledger Live](https://www.ledger.com/pages/ledger-live)
  and review this tutorial:
  [Use the Manager](https://support.ledgerwallet.com/hc/en-us/articles/360006523674-Use-the-Manager).
- You need to have [Electrum](https://electrum.org/).
  Install it and [configure it to be used with Ledger](https://support.ledgerwallet.com/hc/en-us/articles/115005161925-Set-up-and-use-Electrum).
- node >= 10.16.0

## BTC to RBTC conversion

Instructions on how to do a Mainnet peg-in.

**1 Get a BTC address with balance**

We recommend to use Electrum BTC wallet for connecting to
BTC Mainnet using Ledger hardware wallet.

- Download the wallet from
  [Electrum Website](https://bitzuma.com/posts/a-beginners-guide-to-the-electrum-bitcoin-wallet/)
- Install Electrum
- Connect and unlock your Ledger device.
- Open the Bitcoin app
- Start Electrum
- Once Electrum starts, create or import a wallet
- At the keystore screen, select Use a hardware device and click Next.
- Select your Ledger device and click next.
- Choose the right derivation path for your account and click Next:
  - Legacy for an account that has addresses starting with a 1
- Go to the third tab "Receive". You will see a Bitcoin address.

> Note: Check out this article
> [configure it to be used with Ledger](https://support.ledgerwallet.com/hc/en-us/articles/)

> Note: The Bitcoin wallet needs to be legacy (not Segwit)
> whose public key starts with either `m` or `n`,
> and private key starting with `p2pkh:`

**2 Find a BTC address with balance**

You will need to find the corresponding BTC address derived
from the BTC derivation path in Electrum "Receive" tab.

- Check the derivation path for BTC to be used:
  - Mainnet: `44'/0'/0'/0/0`
    [BIP 44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) **Legacy**
- Unlock Ledger and open the **Bitcoin App**
- To get the BTC address derived from the derivation path that you have specified. Run the following script:
  ```
  const Transport = require("@ledgerhq/hw-transport-node-hid").default;
  const AppBtc = require("@ledgerhq/hw-app-btc").default;

  const getBtcAddress = async (derivationPath = "44'/0'/0'/0/0") => {
    try{
        const transport = await Transport.create();
        const btc = new AppBtc(transport);
        const result = await btc.getWalletPublicKey(derivationPath);

        console.log('BTC Address');
        console.log(result.bitcoinAddress);
        console.log('Derivation Path: ' + derivationPath);
    }
    catch(err){
      console.log(err);
    }
  };

  (async () => {
    await getBtcAddress("44'/0'/0'/0/0");
  })();
  ```
- After that you should get a result similar to:
  ```
  BTC Address
  12dAR91ji1xqimzdTQYHDtY....ppSR
  Derivation Path: 44'/0'/0'/0/0
  ```

**This is the address that you have to use in order to do the transfer to the federation.**

**3 Send Bitcoin to RSK Federation address**

<div class="fade alert alert-warning show">
Note: You need to send a minimum amount of 0.01 BTC and
not more than 10 BTC for conversion.
</div>

To get the RSK Federation address you can run the following script:

```
const Web3 = require('web3');
const precompiled = require('@rsksmart/rsk-precompiled-abis');

const getFederationAddress = async function(){
    const bridge = precompiled.bridge.build(new Web3('https://public-node.rsk.co'));
    const address = await bridge.methods.getFederationAddress().call();
    console.log('Federation Address:');
    console.log(address);
}

(async () => {
    await getFederationAddress();
})();
```

Once you have the RSK Federation address, you can send Bitcoin to it from your Bitcoin address.

Use Electrum to send BTCs to the RSK Federation Address. To do that:

- Open Electrum
- Go to Addresses Tab
- Right click over it
- Select the option "Spend From":
  ![Spend from](/assets/img/rsk/peg-ledger/electrumSpendFromOption.png)
- Finally make a payment to the RSK Federation Address
  ![Sending Payment](/assets/img/rsk/peg-ledger/electrumSpendFrom.png)

**4 Wait for BTC confirmations**

To ensure the transaction, we need to wait 100 BTC confirmations, be patient :)

> 100 blocks \* 10 minutes/block = 1000 minutes = 16.667 hours approx.

**5 Get RBTC address from Ledger harware wallet**

Get the corresponding RBTC address from your Ledger harware wallet, following these steps:

- Connect and unlock your Ledger device.
- Open the RSK app.
- Get RSK derived address running this scripts:
  ```
    const Transport = require("@ledgerhq/hw-transport-node-hid").default;
    const AppEth = require("@ledgerhq/hw-app-eth").default;

    const getRskAddress = async (derivationPath = "44'/0'/0'/0/0") => {
      try{
          const transport = await Transport.create();
          const eth = new AppEth(transport);
          const result = await eth.getAddress(derivationPath);

          console.log('RSK Address');
          console.log(result.address);
          console.log('Derivation Path: ' + derivationPath);
      }
      catch(err){
          console.log(err);
      }
    };

    (async () => {
      await getRskAddress("44'/0'/0'/0/0");
    })();

  ```
- Go to MyCrypto and connect to Ledger harware wallet.
- Select **Custom** Address and put the derivation path `m/44'/0'/0'/0`.
  Then choose the address that you got from the previous step.

**6 Check RBTC balance**

You can check balance of RBTC address on MyCrypto or MEW setting the corresponding derivation path and selection the address.

> Note: You have to wait a minimum of 100 confirmations + a minimum of 5 minutes for checking your RBTC balance

## RBTC to BTC conversion

Instructions on how to do a Mainnet peg-out.

**1 Get BTC address with Ledger hardware wallet**

If you forgot your BTC public address, you can check section **1**.
The important thing is that the receiving is BTC address will be
the same that it was used to send to the federation.

**2 Send RBTC to RSK Bridge Contract**

Open MyCrypto or MEW.
Set the corresponding derivation path and selection the address. \
This address has to be the same as that from section **6**.
Then do a transaction to the Bridge Contract.

Bridge Contract address: `0x0000000000000000000000000000000001000006`

> Note: The minimum amount to send in a peg-out transaction must be greater than or equals to 0.004 **RBTC** for Mainnet and the minimum amount to send in a peg-in transaction must be greater than or equals to 0.005 **BTC** for Mainnet
> Gas Limit of the transaction needs to be manually set at 100,000 gas;
> otherwise the transaction will fail.
> Gas Price can be set to 0.06 gwei.

![Customize Gas in Metamask before send transaction on RSK](/dist/images/metamask-gas-limit.png)

**3 Check balance of BTC address**

You can either use Electrum wallet downloaded earlier or
any Bitcoin explorer to check the balance.

> Note: The release process on Bitcoin network takes
> 4000 RSK block confirmations and at least 10 more minutes.
