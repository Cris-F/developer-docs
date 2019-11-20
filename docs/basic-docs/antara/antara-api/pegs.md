# Pegs 

::: tip

The Pegs Antara Module is in the final stages of production. Please reach out to the Komodo team for consultation before attempting to use this module in a production environment.

:::

## Introduction

The Pegs Antara Module is a mechanism for creating a decentralized [stablecoin.](https://en.wikipedia.org/wiki/Stablecoin) 

Stablecoin assets can be used for buying, selling, and trading activities on the associated Smart Chain. 

The Pegs Module allows for stablecoins to be pegged to most common trading assets, such as fiat currency, stocks, and external cryptocurrencies.

#### Module Methodology

##### Associated Modules

The Pegs Antara Module requires interactivity with several additional Antara Modules, including the following:

- The [Gateways](./gateways.html) Module
  - This module acts as a bridge between the Smart Chain where Pegs is active, and an external Komodo-related Smart Chain that features the valuable cryptocurrency that supports the functionality of the Pegs Module
  - On the external chain, a user sends cryptocurrency coins to the Gateways Module, which locks the coins against further spending
  - On the Pegs-related Smart Chain, the Gateways Module then issues to the user [tokens](./tokens.html) that represent the value of the external cryptocurrency
- The [Tokens](./tokens.html) Module
  - This module provides the functionality necessary to create (and burn) tokens in coordination with the Gateways Module
  - The tokens created by the Tokens Module serve as the means of value exchange on the Pegs-related Smart Chain
- The [Oracles](./oracles.html) Module
  - This module uses an [oraclefeed](https://github.com/KomodoPlatform/komodo/blob/master/src/cc/dapps/oraclefeed.c) app to provide information to the Gateways Module about tokens a user deposits
- The [Prices](./prices.html) Module
  - The <!-- What does DTO stand for? --> DTO from the [Prices](./prices.html) Module adds the required price data to the Smart Chain from a range of external sources
  - Data from this module becomes active on the Pegs chain after a 24 hour delay

##### A Deposit and Loan Style System

There are several technical elements within the Pegs Module technology that collaborate to create a stablecoin. 

The first element is a system wherein users of the module lock KMD-based [tokens](../../../basic-docs/antara/antara-api/tokens.html) that represent external assets, such as foreign cryptocurrencies. After locking the tokens to the module, at any time the token contributor may withdraw up to `80%` of their tokens' value in the form of the Smart Chain’s native COIN. (This creates a collateralized "deposit and loan" style system.)

For example, consider a Smart Chain with the name `USDK` where the chain's native coins are pegged to the US Dollar (USD), and the coins are backed by the main coin in the Komodo ecosystem, `KMD`. Users on the `USDK` Smart Chain first create tokens (on the `USDK` Smart Chain) that represent and are locked to their `KMD` cryptocurrency counterparts. Users then lock the tokens to the Pegs Module (on the `USDK` Smart Chain). With the `KMD` tokens locked to the Pegs `USDK` Module, users may now withdraw up to `80%` of the value of the tokens in the form of `USDK` native coins. 

The value of the tokens with respect to the `USDK` coins is calculated according to the current value of `USD` to `KMD`. For example, if the current price ratio between `USD` and `KMD` is `1:1`, then a user that locks `100 KMD` tokens may withdraw `80 USDK` coins. These `USDK` coins can now be used as a representation of USD on the global market, allowing the user to buy, sell, and trade within the `USDK` Smart Chain's ecosystem. When desired, the user that holds `USDK` may exchange these coins for the associated `KMD` tokens.

##### Managing Price Volatility

As time progress, the difference in price between `KMD` and `USD` will change. 

If the value of `KMD` increases relative to `USD`, the user who locked the initial `KMD` tokens may withdraw additional `USDK` coins from the module until the total withdrawn amount is equal to `80%` of the total value of the deposited `KMD` tokens. 

On the other hand, if the value of `KMD` decreases relative to `USD`, the remaining amount of `KMD` tokens in the user’s account is subject to liquidation when the ratio (also called "debt") exceeds `90%` of the total value. 

When this happens, a third party can gain a `5%` rate of return by paying the debt of the account. This is achieved by sending `USDK` coins to Pegs Antara Module to relieve the debt of the associated user account. Here, the `USDK` coins sent are burned. <!-- How does the third party gain a 5% rate of return here? --> The remainder <!-- What specifically do we mean by "remainder"?  Are we talking about the 10% of the user's account that is still above balance? Or is there a remainder leftover from transaction burning the USDK coins? --> is added to the <!-- ?? global address of the ?? --> Pegs Antara Module on the `USDK` chain. This improves the global debt ratio on the `USDK` chain and prevents the underlying tokenized `KMD` assets from falling below the value of total `USDK` coins issued. 

<!-- What about the 20% of the user's value that is held in the account? I assume that the user must always have a 20% locked to 80% available ratio for their fund, and this is where a debt can arise if the value of the underlying cryptocurrency falls relative to the pegged value? Need to explain how a user can extract the full value of their tokens, back into the original cryptocurrency. --> 

To prevent account liquiditation, when a user detects that their account is approaching the `90%` debt-ratio threshold, a user can either return a portion of their `USDK` coins or deposit more tokenized `KMD` at the current price ratio to maintain a good debt ratio.

## Pegs Antara Module Flow

- The Smart Chain creator creates an instance of the Pegs Antara Module, called a "Peg", using the [pegscreate](#pegscreate) API method. 
  - Once created, the creator add this new peg to the Smart Chain's launch parameters using the <!-- Where is this earlytxid parameter in the docs? Need to document the parameter, and then insert a hyperlink here --> `-earlytxid` parameter
- With the peg active on the Smart Chain, a user locks tokenized external cryptocurrency to the Pegs Module using [pegsfund](#pegsfund)
- This user can withdraw up to `80%` of the value of their locked tokens in form of the Smart Chain's coins using [pegsget](#pegsget)
- This user can redeem the locked tokenized external cryptocurrency by repaying the Smart Chain's coins using [pegsredeem](#pegsredeem)
- A user can retrieve the current information about their Pegs account using [pegsaccountinfo](#pegsaccountinfo)
- A user can retrieve all past actions related to their Pegs account using [pegsaccounthistory](#pegsaccounthistory)
- A user can retrieve current information about the Peg using [pegsinfo](#pegsinfo)
- A user that has not yet created an account can exchange native coins of the Smart Chain for deposited tokens belonging to another user's account where the debt ratio is in the "yellow zone", `80%` to `90%`, using [pegsexchange](#pegsexchange)
- A user can retrieve information about accounts that have a debt ratio in the "red zone", `+90%`, using [pegsworstaccounts](#pegsworstaccounts)
- A user can liquidate the "red zone" account of another user, receiving `KMD` tokens in return according to the current price, as well as a `5%` additional profit, using [pegsliquidate](#pegsliquidate)

## Tutorial Availability

The Antara Tutorials section features a full walkthrough for the user side of the Pegs module.

[<b>Link to the user side of the Pegs Module Tutorial</b>](../../../basic-docs/antara/antara-tutorials/pegs-module-user-tutorial.html)

[<b>Link to the creator side of the Pegs Module Tutorial</b>](../../../basic-docs/antara/antara-tutorials/pegs-module-creator-tutorial.html)

## pegsaccounthistory

**pegsaccounthistory pegstxid**

The `pegsaccounthistory` method returns all the past actions related to the Pegs account that belongs to the owner of the pubkey used to launch the daemon.

### Arguments

| Name     | Type     | Description                                                                                    |
| -------- | -------- | ---------------------------------------------------------------------------------------------- |
| pegstxid | (string) | the transaction id of the [pegscreate](#pegscreate) transction used to create the on-chain Peg |

### Response

| Name              | Type            | Description                                                                                                                                           |
| ----------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| "result"          | (string)        | whether the command executed successfully                                                                                                             |
| "name"            | (string)        | name of the method                                                                                                                                    |
| "account history" | (array of json) | an array containing json that describe the past actions related to the Pegs account that belongs to the owner of the pubkey used to launch the daemon |
| "action"          | (string)        | name of the past action                                                                                                                               |
| "amount"          | (number)        | the amount of satoshis involved                                                                                                                       |
| "accounttxid"     | (string)        | transaction id of the action                                                                                                                          |
| "token"           | (string)        | name of the token involved                                                                                                                            |
| "deposit"         | (number)        | the amount of initial satoshis deposited                                                                                                              |
| "debt"            | (string)        | total amount of debt after this action                                                                                                                |

#### :pushpin: Examples

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegsaccounthistory a9539ec8db34ee44ff213cda59f412a02795821cf05844b0bc184660711371f7
```

<collapse-text hidden title="Response">

```json
{
  "result": "success",
  "name": "pegsaccounthistory",
  "account history": [
    {
      "action": "fund",
      "amount": 1000000,
      "accounttxid": "298cfa125e1a38a7aa2a8da8282b017a45cd0c1dc70935712692c00abf48ba3f",
      "token": "KMD",
      "deposit": 1000000,
      "debt": 0
    },
    {
      "action": "get",
      "amount": 100000,
      "accounttxid": "8f18ab623cb91c94ea27b16c455d98df1e057dd120341e54b453090a2c9c9adf",
      "token": "KMD",
      "deposit": 1000000,
      "debt": 100000
    },
    {
      "action": "get",
      "amount": 600000,
      "accounttxid": "8e79ed0a76f359ba048563a0fa2a29ab88ab86b1895e8eff9718f348a04dd1f3",
      "token": "KMD",
      "deposit": 1000000,
      "debt": 700000
    },
    {
      "action": "get",
      "amount": 30000,
      "accounttxid": "9a10581ae047fabe91495cfe558961e3b7362a19b700b2fcea216d06c44d9720",
      "token": "KMD",
      "deposit": 1000000,
      "debt": 730000
    },
    {
      "action": "redeem",
      "amount": 1000000,
      "accounttxid": "ff95e70f0d551748f460e61b7e75b279b8de355cb37df6c5895dfb8a49ee348a",
      "token": "KMD",
      "deposit": 0,
      "debt": 0
    }
  ]
}
```

</collapse-text>

## pegsaccountinfo

**pegsaccountinfo pegstxid**

The `pegsaccountinfo` method returns the current information the Pegs account that belongs to the owner of the pubkey used to launch the daemon.

### Arguments

| Name     | Type     | Description                                                                                    |
| -------- | -------- | ---------------------------------------------------------------------------------------------- |
| pegstxid | (string) | the transaction id of the [pegscreate](#pegscreate) transction used to create the on-chain Peg |

### Response

| Name           | Type            | Description                                                                                                                                       |
| -------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| "result"       | (string)        | whether the command executed successfully                                                                                                         |
| "name"         | (string)        | name of the method                                                                                                                                |
| "account info" | (array of json) | an array containing a json that describes the current state of the Pegs account that belongs to the owner of the pubkey used to launch the daemon |
| "token"        | (string)        | name of the token involved                                                                                                                        |
| "deposit"      | (number)        | the amount of initial satoshis deposited                                                                                                          |
| "debt"         | (string)        | total amount of current debt                                                                                                                      |
| "ratio"        | (string)        | the debt ratio based on the current price                                                                                                         |

#### :pushpin: Examples

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegsaccountinfo a9539ec8db34ee44ff213cda59f412a02795821cf05844b0bc184660711371f7
```

<collapse-text hidden title="Response">

```json
{
  "result": "success",
  "name": "pegsaccountinfo",
  "account info": [
    {
      "token": "KMD",
      "deposit": 1000000,
      "debt": 0,
      "ratio": 0
    }
  ]
}
```

</collapse-text>

## pegsaddress

**pegsaddress [pubkey]**

The `pegsaddress` method returns information about the Pegs module and associated addresses. Optionally, if a pubkey is supplied, this method also gives the Pegs CC address and its balance corresponding to the pubkey.

### Arguments

| Name   | Type              | Description                           |
| ------ | ----------------- | ------------------------------------- |
| pubkey | (string,optional) | pubkey of another user on the network |

### Response

| Name                    | Type     | Description                                                                                                          |
| ----------------------- | -------- | -------------------------------------------------------------------------------------------------------------------- |
| "result"                | (string) | whether the command executed successfully                                                                            |
| "PegsCCAddress"         | (string) | taking the contract's EVAL code as a modifier, this is the public address that corresponds to the contract's privkey |
| "PegsCCBalance"         | (number) | the amount of funds in the `PegsCCAddress`                                                                           |
| "PegsNormalAddress"     | (string) | the unmodified public address generated from the contract's privkey                                                  |
| "PegsNormalBalance"     | (number) | the amount of funds in the `PegsNormalBalance`                                                                       |
| "PegsCCTokensAddress"   | (string) | the public address where Tokens are locked in the Pegs module                                                        |
| "PubkeyCCaddress(Pegs)" | (string) | taking the module's EVAL code as a modifier, this is the Antara address from the pubkey supplied as an argument      |
| "PubkeyCCbalance(Pegs)" | (number) | the amount of funds in the `PubkeyCCaddress(Pegs)`                                                                   |
| "myCCAddress(Pegs)"     | (string) | taking the module's EVAL code as a modifier, this is the Antara address from the pubkey of the user                  |
| "myCCbalance(Pegs)"     | (number) | the amount of funds in the `myCCAddress(Pegs)`                                                                       |
| "myaddress"             | (string) | the public address of the pubkey used to launch the chain                                                            |
| "mybalance"             | (number) | the amount of funds in the `myaddress`                                                                               |

#### :pushpin: Examples

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegsaddress
```

<collapse-text hidden title="Response">

```json
{
  "result": "success",
  "PegsCCAddress": "RHnkVb7vHuHnjEjhkCF1bS6xxLLNZPv5fd",
  "PegsCCBalance": 999.9993,
  "PegsNormalAddress": "RMcCZtX6dHf1fz3gpLQhUEMQ8cVZ6Rzaro",
  "PegsNormalBalance": 0.0,
  "PegsCCTokensAddress": "RHG4K84bPP9h9KKqvpYbUzocaZ3LSUHxLa",
  "myCCAddress(Pegs)": "RBZ4AsnyhD3pZPasDmHXnbwNvQWy1CWK5H",
  "myCCbalance(Pegs)": 0.0,
  "myaddress": "RFmQiF4Zbzxchv9AG6dw6ZaX8PbrA8FXAb",
  "mybalance": 0.1025
}
```

</collapse-text>

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegsaddress 0257e1074b542c47cd6f603e3d78400045df0781875f698138e92cb03055286634
```

<collapse-text hidden title="Response">

```json
{
  "result": "success",
  "PegsCCAddress": "RHnkVb7vHuHnjEjhkCF1bS6xxLLNZPv5fd",
  "PegsCCBalance": 999.9992,
  "PegsNormalAddress": "RMcCZtX6dHf1fz3gpLQhUEMQ8cVZ6Rzaro",
  "PegsNormalBalance": 0.0,
  "PegsCCTokensAddress": "RHG4K84bPP9h9KKqvpYbUzocaZ3LSUHxLa",
  "PubkeyCCaddress(Pegs)": "RRvD19SwUo12Wmozrf8UyuewK1v5kNRJfc",
  "PubkeyCCbalance(Pegs)": 0.0,
  "myCCAddress(Pegs)": "RBZ4AsnyhD3pZPasDmHXnbwNvQWy1CWK5H",
  "myCCbalance(Pegs)": 0.0,
  "myaddress": "RFmQiF4Zbzxchv9AG6dw6ZaX8PbrA8FXAb",
  "mybalance": 0.226
}
```

</collapse-text>

## pegscreate

**pegscreate amount N bindtxid1 [bindtxid2 ...]**

The `pegscreate` method allows the creation of a an on-chain Peg where the Smart Chain's coin is pegged to a real world asset like USD and backed by tokenized Cryptocurrencies like KMD,BTC etc., The pegged asset created can be backed by more than one cryptocurrency at once.

The `amount` parameter is the number of coins to be added to the Pegs module. The coins will be used for transaction fees and markers for Pegs transactions.

The `N` parameter is the number of gateways we are associating with the Pegs module. We need as many Gateways as there are the assets that back the Peg.

The `bindtxidN` parameter is the `bindtxid` of a gateway we are using to tokenize external cryptocurrencies.

The transction id of the `pegscreate` transaction is called the `pegstxid`. Once it is confirmed, the Smart Chain daemon should be stopped and started again with the parameter `-earlytxid=pegstxid` added to the launch command. This ensures that it is the only Peg active on the Smart Chain. The `-earlytxid` parameter can be added to the launch parameters only before the 100th block.

### Arguments

| Name            | Type                | Description                                                                                  |
| --------------- | ------------------- | -------------------------------------------------------------------------------------------- |
| amount          | (number)            | the number of coins to be added                                                              |
| N               | (number)            | the number of gateways we are associating with the Pegs module                               |
| bindtxid1       | (string)            | the `bindtxid` of a gateway we are using to tokenize external cryptocurrencies               |
| [bindtxid2 ...] | (strings, optional) | same as above; used when more than one external cryptocurrency is being used to back the Peg |

### Response

| Name     | Type     | Description                               |
| -------- | -------- | ----------------------------------------- |
| "hex"    | (string) | the hex to be broadcasted                 |
| "result" | (string) | whether the command executed successfully |

#### :pushpin: Examples

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegscreate 100000 1 0b5716554e523aa4678112a8ac3d15039e0aae6f4812b9d4c631cc9cfbf48786
```

<collapse-text hidden title="Response">

```json
{
  "hex": "0400008085202f890197c74ae1999260bb3c7342dd25f3f68a0991d4b50a40f4a6e37838b6e3ca112e02000000484730440220466a45514b5eb24a3a14886dbca204997db498e1509928700a3e0ac63645be3402204164ce5c8f8eb82785158b1f79000721be4db45bb3726fb8043fb089aa19886601ffffffff669ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc2f96465cc2480000232102d3431950c2f0f9654217b6ce3d44468d3a9ca7255741767fdeee7c5ec6b47567ac0000000000000000256a23ee43018687f4fb9ccc31c6d4b912486fae0a9e03153daca8128167a43a524e5516570b000000005f0100000000000000000000000000",
  "result": "success"
}
```

</collapse-text>

Broadcast the hex:

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD sendrawtransaction 0400008085202f890197c74ae1999260bb3c7342dd25f3f68a0991d4b50a40f4a6e37838b6e3ca112e02000000484730440220466a45514b5eb24a3a14886dbca204997db498e1509928700a3e0ac63645be3402204164ce5c8f8eb82785158b1f79000721be4db45bb3726fb8043fb089aa19886601ffffffff669ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc2f96465cc2480000232102d3431950c2f0f9654217b6ce3d44468d3a9ca7255741767fdeee7c5ec6b47567ac0000000000000000256a23ee43018687f4fb9ccc31c6d4b912486fae0a9e03153daca8128167a43a524e5516570b000000005f01000000000000000000000000000400008085202f890197c74ae1999260bb3c7342dd25f3f68a0991d4b50a40f4a6e37838b6e3ca112e02000000484730440220466a45514b5eb24a3a14886dbca204997db498e1509928700a3e0ac63645be3402204164ce5c8f8eb82785158b1f79000721be4db45bb3726fb8043fb089aa19886601ffffffff669ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc9ce7764817000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc2f96465cc2480000232102d3431950c2f0f9654217b6ce3d44468d3a9ca7255741767fdeee7c5ec6b47567ac0000000000000000256a23ee43018687f4fb9ccc31c6d4b912486fae0a9e03153daca8128167a43a524e5516570b000000005f0100000000000000000000000000
```

<collapse-text hidden title="Response">

```
a9539ec8db34ee44ff213cda59f412a02795821cf05844b0bc184660711371f7
```

</collapse-text>

The above string is the `pegstxid` that represents the Peg.

## pegsexchange

**pegsexchange pegstxid tokenid amount**

The `pegsexchange` method allows the exchange of a given amount of the Smart Chain's coins for the deposited tokens. This method is used when the user does not have an account, but wants to exchange the Smart Chain's coins for the tokenized Cryptocurrencies. This method repays the debt of another user whose account is in the "yellow zone" (debt ratio is about `80%` to `90%` based on current prices) and improves their debt ratio and prevent their account from being liquidated.

### Arguments

| Name     | Type     | Description                                                                          |
| -------- | -------- | ------------------------------------------------------------------------------------ |
| pegstxid | (string) | the transaction id of the [pegscreate](#pegscreate) transaction that created the Peg |
| tokenid  | (string) | the tokenid of the tokenized cryptocurrency backing the peg                          |
| amount   | (amount) | the amount of coins to exchange                                                      |

### Response

| Name     | Type     | Description                               |
| -------- | -------- | ----------------------------------------- |
| "hex"    | (string) | the hex to be broadcasted                 |
| "result" | (string) | whether the command executed successfully |

#### :pushpin: Examples

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegsexchange a9539ec8db34ee44ff213cda59f412a02795821cf05844b0bc184660711371f7 1a459712f1e79a544efdf55cfb3e111c5df3300a4da4d16cb3b963bbb50aebf1 0.01
```

<!--
FIXME

add response

<collapse-text hidden title="Response">

```json

```

</collapse-text>
-->

## pegsfund

**pegsfund pegstxid tokenid amount**

The `pegsfund` method allows a user to lock the given `amount` of a tokenized cryptocurrency in the Pegs module.

### Arguments

| Name     | Type     | Description                                                                          |
| -------- | -------- | ------------------------------------------------------------------------------------ |
| pegstxid | (string) | the transaction id of the [pegscreate](#pegscreate) transaction that created the Peg |
| tokenid  | (string) | the tokenid of the tokenized cryptocurrency backing the peg                          |
| amount   | (amount) | the amount of the tokenized cryptocurrency to be locked in the Pegs account          |

### Response

| Name     | Type     | Description                               |
| -------- | -------- | ----------------------------------------- |
| "hex"    | (string) | the hex to be broadcasted                 |
| "result" | (string) | whether the command executed successfully |

#### :pushpin: Examples

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegsfund a9539ec8db34ee44ff213cda59f412a02795821cf05844b0bc184660711371f7 1a459712f1e79a544efdf55cfb3e111c5df3300a4da4d16cb3b963bbb50aebf1 0.01
```

<collapse-text hidden title="Response">

```json
{
  "hex": "0400008085202f89024c386ef60103e74f339867bd7c38b7f187ceee842b8e57ab9a42a16b0721be23040000007b4c79a276a072a26ba067a565802103c75c1de29a35e41606363b430c08be1c2dd93cf7a468229a082cc79c7b77eece8140213fbd380d7905f770709725fe462e5f0a731eca4e56435c4b2f7c96d5b6201433e7c780d313795ddf9ae1d0b689b4f950af6e0e07dd9bcbc293b46f2a349ef4a100af038001eea10001ffffffff9c29a09fb0f1f178ed83e7064297a3d0ba892d9b0eb6b9f0b04c3622203f8be3000000007b4c79a276a072a26ba067a56580210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e0681409a7fa29976f948038b77573fb1103836340ebbcdfbbba8dc852186be1666811c034adf2ec9d9515eb58f14df81925f46fb152b2964e8f392b70bafdfe3a8af83a100af038001f2a10001ffffffff051027000000000000302ea22c8020e39343ebe1b40dcc747d145f140983b38f230dcf7963f2b58051265c765f2efa81031210008203000401cc1027000000000000302ea22c8020d77058bfd93eebe366e1c82fc1475690fd290214ed43b8c9dd25374077b35cbe81031210008203000401cc40420f0000000000302ea22c802002bc3497bdabeac3d0c40ac845fa105685724d1b70c84bd6c5cef2ff4c353e7881032214008203000401cc3cdf993b00000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc0000000000000000c86a4cc5f2741a459712f1e79a544efdf55cfb3e111c5df3300a4da4d16cb3b963bbb50aebf102210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e062103c75c1de29a35e41606363b430c08be1c2dd93cf7a468229a082cc79c7b77eece175cee46f7711371604618bcb04458f01c829527a012f459da3c21ff44ee34dbc89e53a9210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e0640420f000000000040420f0000000000000000000000000000000000a79900000000000000000000000000",
  "result": "success"
}
```

</collapse-text>

Broadcast the hex:

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD sendrawtransaction 0400008085202f89024c386ef60103e74f339867bd7c38b7f187ceee842b8e57ab9a42a16b0721be23040000007b4c79a276a072a26ba067a565802103c75c1de29a35e41606363b430c08be1c2dd93cf7a468229a082cc79c7b77eece8140213fbd380d7905f770709725fe462e5f0a731eca4e56435c4b2f7c96d5b6201433e7c780d313795ddf9ae1d0b689b4f950af6e0e07dd9bcbc293b46f2a349ef4a100af038001eea10001ffffffff9c29a09fb0f1f178ed83e7064297a3d0ba892d9b0eb6b9f0b04c3622203f8be3000000007b4c79a276a072a26ba067a56580210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e0681409a7fa29976f948038b77573fb1103836340ebbcdfbbba8dc852186be1666811c034adf2ec9d9515eb58f14df81925f46fb152b2964e8f392b70bafdfe3a8af83a100af038001f2a10001ffffffff051027000000000000302ea22c8020e39343ebe1b40dcc747d145f140983b38f230dcf7963f2b58051265c765f2efa81031210008203000401cc1027000000000000302ea22c8020d77058bfd93eebe366e1c82fc1475690fd290214ed43b8c9dd25374077b35cbe81031210008203000401cc40420f0000000000302ea22c802002bc3497bdabeac3d0c40ac845fa105685724d1b70c84bd6c5cef2ff4c353e7881032214008203000401cc3cdf993b00000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc0000000000000000c86a4cc5f2741a459712f1e79a544efdf55cfb3e111c5df3300a4da4d16cb3b963bbb50aebf102210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e062103c75c1de29a35e41606363b430c08be1c2dd93cf7a468229a082cc79c7b77eece175cee46f7711371604618bcb04458f01c829527a012f459da3c21ff44ee34dbc89e53a9210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e0640420f000000000040420f0000000000000000000000000000000000a79900000000000000000000000000
```

<collapse-text hidden title="Response">

```
298cfa125e1a38a7aa2a8da8282b017a45cd0c1dc70935712692c00abf48ba3f
```

</collapse-text>

The above string is the `accounttxid` of the user.

## pegsget

**pegsget pegstxid tokenid amount**

The `pegsget` method allows a user to take a debt of upto 80% value(based on the current price) of the locked(collateral) tokenized cryptocurrency in the form of the Smart chain's coins.

### Arguments

| Name     | Type     | Description                                                                          |
| -------- | -------- | ------------------------------------------------------------------------------------ |
| pegstxid | (string) | the transaction id of the [pegscreate](#pegscreate) transaction that created the Peg |
| tokenid  | (string) | the tokenid of the tokenized cryptocurrency backing the peg                          |
| amount   | (amount) | the amount of the SmartChain coins to receive from the Pegs module                   |

### Response

| Name     | Type     | Description                               |
| -------- | -------- | ----------------------------------------- |
| "hex"    | (string) | the hex to be broadcasted                 |
| "result" | (string) | whether the command executed successfully |

#### :pushpin: Examples

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegsget a9539ec8db34ee44ff213cda59f412a02795821cf05844b0bc184660711371f7 1a459712f1e79a544efdf55cfb3e111c5df3300a4da4d16cb3b963bbb50aebf1 0.001
```

<collapse-text hidden title="Response">

```json
{
  "hex": "0400008085202f8903b68353f713e1d70cf2fe4fa538de32a8723e603d507c8ee2d42277f3fd7334ad00ca9a3b0201e2ffffffff3fba48bf0ac09226713509c71d0ccd457a012b28a88d2aaaa7381a5e12fa8c2900000000a74ca5a281a1a0819ca28194a067a565802103c75c1de29a35e41606363b430c08be1c2dd93cf7a468229a082cc79c7b77eece81402e06d560cac761ee7d7720f6ed0f307520fda7c66b2ac0d16cd3c34a33f004393f735429a7ad445a1c970723a7a9d173e1fcc62b54ed164dfb40f23542c9627da129a52780209de43b2cf09dcb2107822237e0afba0b0917b1592457b6fbfddb34fff3d302ae8103020000af038001eea10001ffffffff3fba48bf0ac09226713509c71d0ccd457a012b28a88d2aaaa7381a5e12fa8c2901000000a74ca5a281a1a0819ca28194a067a56580210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e068140152ff019e1e4346d450ab149f1ef9115ea84c9ff7c3bb60444a7e48eda0784a46ecefbf1a35f0062513033d9c3fce45a083f70b7ccc41103763c267598bcadcca129a52780209de43b2cf09dcb2107822237e0afba0b0917b1592457b6fbfddb34fff3d302ae8103020000af038001eea10001ffffffff041027000000000000302ea22c8020e39343ebe1b40dcc747d145f140983b38f230dcf7963f2b58051265c765f2efa81031210008203000401cc1027000000000000302ea22c8020d77058bfd93eebe366e1c82fc1475690fd290214ed43b8c9dd25374077b35cbe81031210008203000401cca08601000000000023210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e06ac0000000000000000fd22016a4d1e01e211b68353f713e1d70cf2fe4fa538de32a8723e603d507c8ee2d42277f3fd7334ad00000400008085202f89013fba48bf0ac09226713509c71d0ccd457a012b28a88d2aaaa7381a5e12fa8c290000000000ffffffff01a086010000000000ab6a4ca8e28efefefe7f065045475343433e140a06d1f028a3abce516bd5d08ebfd860da116dfb0c3a22003b57f0809b5400f7711371604618bcb04458f01c829527a012f459da3c21ff44ee34dbc89e53a9f1eb0ab5bb63b9b36cd1a44d0a30f35d1c113efb5cf5fd4e549ae7f11297451a210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e06a08601000000000040420f0000000000a08601000000000000000000b0990000000000000000000000000000000000000000000000000000000000000000",
  "result": "success"
}
```

</collapse-text>

## pegsinfo

**pegsinfo pegstxid**

The `pegsinfo` method returns the current information about the given Peg.

### Arguments

| Name     | Type     | Description                                                                          |
| -------- | -------- | ------------------------------------------------------------------------------------ |
| pegstxid | (string) | the transaction id of the [pegscreate](#pegscreate) transaction that created the Peg |

### Response

| Name            | Type         | Description                                                      |
| --------------- | ------------ | ---------------------------------------------------------------- |
| "result"        | (string)     | whether the command executed successfully                        |
| "name"          | (string)     | name of the method                                               |
| "info"          | (json array) | the current information about the given Peg                      |
| "token"         | (string)     | name of the token                                                |
| "total deposit" | (amount)     | total number of tokens deposited                                 |
| "total debt"    | (amount)     | total number of satoshis of Smart Chain's coin withdrawn         |
| "total ratio"   | (string)     | total debt ratio for the above token based on the current price  |
| "global ratio"  | (string)     | global debt ratio for all the tokens based on the current prices |

#### :pushpin: Examples

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegsinfo a9539ec8db34ee44ff213cda59f412a02795821cf05844b0bc184660711371f7
```

<collapse-text hidden title="Response">

```json
{
  "result": "success",
  "name": "pegsinfo",
  "info": [
    {
      "token": "KMD",
      "total deposit": 2000000,
      "total debt": 300000,
      "total ratio": "16.67%"
    }
  ],
  "global ratio": "16.67%"
}
```

</collapse-text>

## pegsliquidate

**pegsliquidate pegstxid tokenid accounttxid**

The `pegsliquidate` method allows a user to liquidate the account of another user if their debt ratio is in "red zone" (greater than "90%") for a given token specified by the `tokenid` by repaying the Smart Chain's coins for a profit of `5%`

### Arguments

| Name        | Type     | Description                                                                          |
| ----------- | -------- | ------------------------------------------------------------------------------------ |
| pegstxid    | (string) | the transaction id of the [pegscreate](#pegscreate) transaction that created the Peg |
| tokenid     | (string) | the tokenid of the tokenized cryptocurrency backing the peg                          |
| accounttxid | (string) | the `accounttxid` of another user whose account is in the "red zone"                 |

### Response

| Name     | Type     | Description                               |
| -------- | -------- | ----------------------------------------- |
| "hex"    | (string) | the hex to be broadcasted                 |
| "result" | (string) | whether the command executed successfully |

#### :pushpin: Examples

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegsliquidate a9539ec8db34ee44ff213cda59f412a02795821cf05844b0bc184660711371f7 1a459712f1e79a544efdf55cfb3e111c5df3300a4da4d16cb3b963bbb50aebf1 298cfa125e1a38a7aa2a8da8282b017a45cd0c1dc70935712692c00abf48ba3f
```

<!-- FIXME

<collapse-text hidden title="Response">

```json

```

</collapse-text>

-->

## pegsredeem

**pegsredeem pegstxid tokenid**

The `pegsredeem` method allows a user to withdraw their deposited tokenized cryptocurrency by repaying the whole debt in the Smart Cahin's coins.

### Arguments

| Name     | Type     | Description                                                                          |
| -------- | -------- | ------------------------------------------------------------------------------------ |
| pegstxid | (string) | the transaction id of the [pegscreate](#pegscreate) transaction that created the Peg |
| tokenid  | (string) | the tokenid of the tokenized cryptocurrency backing the peg                          |

### Response

| Name     | Type     | Description                               |
| -------- | -------- | ----------------------------------------- |
| "hex"    | (string) | the hex to be broadcasted                 |
| "result" | (string) | whether the command executed successfully |

#### :pushpin: Examples

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegsredeem a9539ec8db34ee44ff213cda59f412a02795821cf05844b0bc184660711371f7 1a459712f1e79a544efdf55cfb3e111c5df3300a4da4d16cb3b963bbb50aebf1
```

<collapse-text hidden title="Response">

```json
{
  "hex": "0400008085202f891e045ffbf7d56aba87d1527a8bd50c7cbfb44f376b1c0a6815c741a5f6e107f02d00000000484730440220064f36926ed7d6975ad1ecb358df866897554d29d623447b95d0f9d1f575ff0a02203ca4d9ea407c2310cd49be1315887cde4e54619de93d6882f0c2fe871820ef3201ffffffff3fba48bf0ac09226713509c71d0ccd457a012b28a88d2aaaa7381a5e12fa8c29030000007b4c79a276a072a26ba067a565802103c75c1de29a35e41606363b430c08be1c2dd93cf7a468229a082cc79c7b77eece81400301711d3ddb1753c3a004c250616ea580f0bf75c1c03f791f76327a6efa69792f19eae5fa3c73ab29a4b6754a91111935a46fa1c28d4cdaacdcb24ec571a45fa100af038001eea10001ffffffff20974dc4066d21eafcb200b7192a36b7e3618955fe5c4991befa47e01a58109a00000000a74ca5a281a1a0819ca28194a067a565802103c75c1de29a35e41606363b430c08be1c2dd93cf7a468229a082cc79c7b77eece81409ffd2bb29f40dbf7b3e741e00e3a97c7e66f49526ea9dd33ec724eb30a81b21b16cdac1ea840cbf902f36ef17b9ac81014a735e97377d6d5f07a6e8ac04556f8a129a52780209de43b2cf09dcb2107822237e0afba0b0917b1592457b6fbfddb34fff3d302ae8103020000af038001eea10001ffffffff20974dc4066d21eafcb200b7192a36b7e3618955fe5c4991befa47e01a58109a01000000a74ca5a281a1a0819ca28194a067a56580210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e068140bbfdfb024c50111ed7184b3fd4c7c462ea5093ab9969bab386d54965e248184a00a5cb2a1f625116d3bd02b470f4b33dab11a38ffd4d050fcdbe01e8ae754e70a129a52780209de43b2cf09dcb2107822237e0afba0b0917b1592457b6fbfddb34fff3d302ae8103020000af038001eea10001ffffffff3fba48bf0ac09226713509c71d0ccd457a012b28a88d2aaaa7381a5e12fa8c2902000000ac4caaa281a6a081a1a28194a067a56580210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e068140bfb34bd8abc720bef5c13708ad9280893df2b9fb1e70eb6063c05614245f5d3538d3048b9f6e4fbfc3053e594df7e638dd22e42c833c75aa174edf010fb45419a129a52780209de43b2cf09dcb2107822237e0afba0b0917b1592457b6fbfddb34fff3d302ae8103020000af038001eeaf038001f2a10001ffffffff0dfe7aecdaed2498f6a3c29bcd53f15913ae2b36bf7643efcb8d3eb93354fbad00000000484730440220403e6284a585ed11b58207279786396101ea442d79b9df44688ebdaa7467e04d0220474f38ecc4ff53f9d685538805d0f0b7eaf008c9736ddf677a4bb88919e5680e01ffffffff0491bd559806afc9638763c8fb8e550b5e13df6d18ed0eeb7209f8e7e6ccb90f0000000048473044022078b520bbaa03883e03321a98cfca3282f7f1d70d91a977ed81b50d0f91440b3402205d1f346312ffbe4b76c614fb8d9dfcfd38069eca55a8f7d2ae84d4cae6fb481901ffffffff0d5e3529d01acc37c5cfbac0a3321870a5097134029ed474db42023bcbf18bb50000000049483045022100e6ccd2403781c60d928fa090d78d3f42cfa812dcd37ca5c65c91969a81a6d8ab022067f917bd432c69a44fd338a4897462e294b40304ea44dfe544a826bdb75232b901ffffffff07c9ea3dcb2164a8b5e839b23f39e6d2e0d3b8a7848ea188dca6e61a9833a8d00000000048473044022061ffebbf638055097c9bfaf654135e65925e7107cdd86d553264df3c16eefd3f0220401de8c419a842ad1f72eab0eaeca0f63ff95419e222a94141b04b5eaa3e341701ffffffff0ac35ee065ed0090ef129c389f0b84c91feaf9bf8a4ae6c12480a4d435cd550400000000484730440220710f15cebed5631fdaee51b651456b2b56e1d83968526745241a4920c6719b2b022000abc38a4b9ae4f6a7c53c1e80049c7d9109f97e26daf76a27021ee805a21a1d01ffffffff0aee5de22398ff1a61bca8d95d3304337be9cb00d401afa088cf95d31f121b1d000000004847304402207e4c1793bb91c550e462d38531734dd9fece17bab40ad0d2e8c2e9cd741f342302202c4973b14c9a5768a72aa5a4ab752de8fb388fd208280b6e5ef529e918dcca4101ffffffff0bd4eeed3d68d4f361e4e65a06f1e437739d209c5b593adfb9c908a11e5c4b1e0000000049483045022100a3e1e8f18bdd72f3794d3e2bfef1d807f904c93c511d396d33f66a9a84ea988b02203954987e94629cf77982568f115a148d64b44f1619c285d9ea229ac81268a16c01ffffffff023035ddb47d6d6ba1a1e760a750325f912f66934130059313bd2460dd3626a80000000049483045022100be9bc941d1afcc3e816a70e2e49e0c0b5279d8c704b14e8ea6f0a5b8e644f999022016ac0c28af028149a3026eb3cbe266604809643b303ff3ae4c27f4a11ed7059901ffffffff0d0db11a4bfaeb4293d26d4f540f8ae7b8ac648998d32ee4902eb6445bf3b4f4000000004948304502210093896862f4c679d6db6eb57951b5117a6201eda285b257dd60c57a45a3254cbb02205a7d32b4ffbf0f8d90cf1fca8a0f4f09e36da21c536e32f7002a28d6d56d132b01ffffffff0623be2e2bd36d04b45df4fcb8349f0b10095ec1aac30613fd21ca40054361300000000049483045022100d24bb6d6720e876a57607c8e29348f29357181d4369e41b044be9a4ac699272c022046e536c70a5ae151c99cff8795b9a1a6fbf159cfdac0c36bbceced6f6c7cec6b01ffffffff0cc55d115db43ac735fef6a662896b24f288b2fbb7b793c1f96948f6820338ed00000000494830450221009d6ffacd7e7c167c254ad79e03184d30f3598a75baf5422f62a74fbee20f8d35022029c7348297580926f64d6f3dd69f54aaec45b428594fae57af252ccf2fb804e401ffffffff08230c57a73ae1167c7c99c5e135af4e52b21468c9ef6de6528e51700d611fcd00000000484730440220645c65d17a44532e80bec8c40fd683dfbf614a6d64acec64eb2567788bbf9ea402205976d7399e33d20e588230ff6d65cef4dde327ffbb42a15e2191400896b34a4a01ffffffff083ae311d476e71cbd214e90aa7065bc3f8e3735fccdb951e84be2f35dbe86e90000000049483045022100aedf51f885b813acbce5799f976dff5200658396d3c33dd36f5768d5225215e3022070faaccac58b8aee064a312d6ed01301ac947c9e5cfd6be3d81d5cabb48ddf0901ffffffff012f6500ac7ff9cf0a6a624ca7a49e684fc6d28b5646a47c1f65d693965b1c430000000049483045022100ea4eca020c79db8fe46ae1a970c15deda516180c875bbbdb7c43a73e884a60f202207278af9cd50ee2806f6e4f16a5f2e4454881deacbcade363289239fdd54a982d01ffffffff08359b7d0efb9eede36cf0a5d9fdc5f4a83308c8a11099a3fa4cbb1fadbfcfde0000000049483045022100d1c1ed48608d61ae88cd957281e9320311861dbdc3596136533f058313883b6a02200ba029992e819196fd7c83b51a8d108c1b3c9b1f03a7faf2290acfced66c113a01ffffffff083f857139d0711209eef17e2f6177cb4ed0f68c00f520038a587744262445b90000000049483045022100cd94b3dd563ad7a37bd531a89cb0d988f4ec92949df7302ba43e13bdb88082c1022003be0f619ae4b3de597f8716a9374d63becf81d3d3943b3c741ee636b7e5cd8401ffffffff09cb8b189715b655a0c07456648951c3c178a6c6270c484dd3cfad5b18f859880000000048473044022025b7fc3db1a0b33471e5eccd3cb54a71fec8c5366837388489f3d9c90ad80d070220485c4c79790e2cfac55566869a52dbd86426f4d1db9fa23314d82b26ffc8f06d01ffffffff070d10ba3a3bb2f50fbc88549ab14aea8424bfa5cf4932653a82b2af987a82200000000048473044022047ac9d547dbf96cebd1f6dc29aad99d752c60b316df54987b577b7cc04cb6a9302204ae6a4707e6f270a4bfdc1d55e31bbbaf95e233a6c26ae6823b3f592cb39656301ffffffff0be472c9362d31a026a7f22d0fa14f294c95b0457ae66cd7179bb29a1f30278d0000000049483045022100e54335c0b652f35a3ae2eda351638b3b5d1ff76bf256bfcfe6a3e8810fd131ad02202865894c90cdbea6e1f052a83a085236606da96afc03e47a79159c5f9ee1398401ffffffff051c297d636cc51d90a5996a23ffc7c5374a4e3df397f770b5113585749bb5480000000049483045022100d67d9b0c34aa8f8d6c4ed261af543ce4cd9b95fcae7bbbb94c0f2521982037be02207fa0b75e74f2c2e9a5009d160addb3b191cf751e0887f0186526023f7f79b80401ffffffff0b19234b25da56339d70722c5845e39a56513ce6b70c7c26486e7d3b8c5bd02c000000004948304502210086f6f6dc5005207cc4206ff43d29e1ca42fc6cf5ce30e870d6bfa39002264a9c02200be78b519d6d2f7448ded31e96077b00960532c7d7b1484c37fe6a1d6990237e01ffffffff0d628dfaece395d197c0b810668b7af212d083e5f441a4c729b164ab3df91b510000000049483045022100c6ca324eeaafc9bacae5c260981baf9ae8f5c0eb3f83533c8631ecbe592d470c022014d44bc921909246002d64e4c029af2a4abb26812a6902cf61f69782630ce9c901ffffffff039d71284b8643c45e14b5bfc8c9439a17474376a78cec7be3d6b688a40bc4fc000000004847304402206b8320f3ea3cdd73fa9e4ee43a62144076b6ffc6cf58fdb06233b542d6924db402205a2310f7fc382407c8466b5677691988a6dc2a0fe3f3b4c6bc1f1a01d1820e7901ffffffff02803fb66f0c03fa8211d4086ad2262e50ea5dd99f30603e9999603cdfac367c0000000049483045022100837fec21e7ba574c2283ec160bc418485f2dd6fbb5f5dd575156c41450e7deb6022058aefedcc08274f523c52981b69f124197a431580d5692c2a5f6e8af6bef594801ffffffff0af2f9ced5de00310114f70d6756db7df65205aeac26b5607bd71333812f68e40000000049483045022100d5fdd26ad4682b6aa2d522c1b75097f9cb3df94632e5cfc2454d79b76f9401660220193d741cc875e75cf04ad70de582915b14b41a6283967527e931c50565cdb62f01ffffffff071027000000000000302ea22c8020e39343ebe1b40dcc747d145f140983b38f230dcf7963f2b58051265c765f2efa81031210008203000401cc1027000000000000302ea22c8020d77058bfd93eebe366e1c82fc1475690fd290214ed43b8c9dd25374077b35cbe81031210008203000401cc40420f0000000000302ea22c8020d62369afdb9fbe16bae7a6ac394333e9337ecc0b7671cda491df2fa82e48a20d8103120c008203000401cc90230b0000000000232102f7711371604618bcb04458f01c829527a012f459da3c21ff44ee34dbc89e53a9ac2cb8993b00000000302ea22c802039452b774825750cd9390c3f05c96e486ecf2f21779466efbcd214220a7f288a8103120c008203000401cc102700000000000023210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e06ac0000000000000000a66a4ca3f2741a459712f1e79a544efdf55cfb3e111c5df3300a4da4d16cb3b963bbb50aebf101210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e06175cee52f7711371604618bcb04458f01c829527a012f459da3c21ff44ee34dbc89e53a9210217a6aa6c0fe017f9e469c3c00de5b3aa164ca410e632d1c04169fd7040e20e0640420f00000000000000000000000000000000000000000000000000209d00000000000000000000000000",
  "result": "success"
}
```

</collapse-text>

## pegsworstaccounts

**pegsworstaccounts pegstxid**

The `pegsworstaccounts` method returns the information on the accounts that can be liquidated based on the current prices (Accounts whose debt ratio is in the red zone i.e., exceeds `90%`).

### Arguments

| Name     | Type     | Description                                                                          |
| -------- | -------- | ------------------------------------------------------------------------------------ |
| pegstxid | (string) | the transaction id of the [pegscreate](#pegscreate) transaction that created the Peg |

<!-- FIXME

### Response

| Name     | Type     | Description                               |
| -------- | -------- | ----------------------------------------- |
| "result" | (string) | whether the command executed successfully |
| "name"   | (string) | name of the method                        |
|          |          |                                           |


#### :pushpin: Examples

Command:

```bash
./komodo-cli -ac_name=HELLOWORLD pegsworstaccounts a9539ec8db34ee44ff213cda59f412a02795821cf05844b0bc184660711371f7
```


<collapse-text hidden title="Response">

```json

```

</collapse-text>
-->