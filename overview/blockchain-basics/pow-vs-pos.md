---
description: >-
  Proof of work (PoW) and proof of stake (PoS) are two different blockchain
  consensus mechanisms.
---

# PoW vs PoS

## Contents

* <mark style="color:blue;">****</mark>[<mark style="color:blue;">**Proof of Work**</mark>](pow-vs-pos.md#proof-of-work)<mark style="color:blue;">****</mark>
* <mark style="color:blue;">****</mark>[<mark style="color:blue;">**Proof of Stake**</mark>](pow-vs-pos.md#proof-of-stake)<mark style="color:blue;">****</mark>
* <mark style="color:blue;">****</mark>[<mark style="color:blue;">**Staking**</mark>](pow-vs-pos.md#staking)<mark style="color:blue;">****</mark>

## **Proof of Work**

{% hint style="info" %}
_**Bitcoin and Ethereum are two of the most well known blockchains that use a proof of work consensus protocol.**_
{% endhint %}

Proof of work (PoW) blockchains require network participants, also known as “miners,” to guess the solution to a complex mathematical problem in order to add a “block” to the decentralized public ledger. The miner that solves or comes closest to solving the problem earns a prize for their work, usually in the form of the blockchain’s native cryptocurrency.

{% hint style="success" %}
_**Proof of work blockchains are able to be extremely decentralized because anyone with the necessary hardware and skillset can participate in the consensus.**_
{% endhint %}

{% hint style="danger" %}
_**Proof of work blockchains often consume significant amounts of energy when operating at a large scale, and face scalability issues which can make basic transactions slower and more expensive for the average user.**_
{% endhint %}

## Proof of Stake

{% hint style="info" %}
_**Umee is a proof of stake blockchain. Other well known proof of stake blockchains include Cosmos, Solana, Polkadot, and Cardano.**_
{% endhint %}

Proof of stake (PoS) blockchains require participants, also known as “**validators**,” to lock up, or “**stake**,” coins/tokens on the network as a form of collateral. Validators are randomly selected to “validate” a block that will be added to the decentralized public ledger once a consensus has been reached. Block rewards are split between validators according to the amount of coins/tokens they have staked relative to the total amount staked.

On rare occasions validators can be “**slashed**.” Slashing occurs when a validator misbehaves, and a percentage of their total stake is redistributed or burnt as a punishment. There are two scenarios in which an Umee validator’s stake is slashed:

* When a validator has significant downtime, and is not actively contributing to the validation of blocks;&#x20;
* When a validator “double signs,” or signs the same block twice. Validators who double sign will also be "tombstone jailed," or kicked from the active set and unable to rejoin.

{% hint style="success" %}
_**Proof of stake blockchains become more decentralized as more validators participate.**_
{% endhint %}

{% hint style="success" %}
_**Proof of stake blockchains allow all coin/token holders to earn block rewards.**_
{% endhint %}

{% hint style="success" %}
_**Proof of stake blockchains consume a small fraction of the energy that proof of work blockchains demand.**_
{% endhint %}

{% hint style="success" %}
_**Proof of stake blockchains are more scalable by design, meaning transactions are usually fast and inexpensive for the average user, and more applications can be built on top.**_
{% endhint %}

## Staking

Coin/token holders have the ability to “**delegate**” their tokens to a validator to help the network remain secure and decentralized. In return, those who delegate their assets earn “**staking rewards,**” a proportional share of the block rewards relative to the amount of tokens they have staked. Additionally, those who delegate their assets are typically able to participate in [<mark style="color:blue;">protocol governance</mark>](../governance/) decisions to help determine the future of the network.

It’s incredibly important for any proof of stake network to have a significant percentage of the total supply of coins/tokens staked at all times in order to increase the cost associated with an attack so it’s not attainable and/or profitable for a bad actor.

In order to encourage coin/token holders to stake their tokens to help secure the network, most proof of stake blockchains initially use a strategically chosen token inflation rate so that coin/token holders who do not delegate/stake are penalized by having their market share diluted.

Validators with greater amounts of coins/tokens staked will have higher chances of being selected to validate a block, as well as greater influence over governance processes. It’s essential for coin/token holders to [<mark style="color:blue;">choose carefully when selecting a validator to stake with</mark>](../../using-umee/staking-umee/selecting-a-validator.md). If the majority of total tokens staked are staked by a handful of validators, the network becomes less decentralized and secure.

{% hint style="info" %}
_**Token holders should delegate to validators with smaller stakes in order to help support the decentralization and security of the network.**_
{% endhint %}

### Learn How To:

* <mark style="color:blue;"></mark>[<mark style="color:blue;">**Select a validator**</mark>](../../using-umee/staking-umee/selecting-a-validator.md)**;**
* ****[<mark style="color:blue;">**Stake UMEE**</mark>](../../using-umee/staking-umee/)**;**
* ****[<mark style="color:blue;">**Vote on a governance proposal**</mark>](../../using-umee/participating-in-governance.md)**.**
