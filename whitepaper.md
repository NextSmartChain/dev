# NEXT Smart Chain
**NEXT Smart Chain is an ultrafast blockchain, ideal for NFT, DeFi, Gaming, and Metaverse applications. It's also 100% backward-compatible with Ethereum and Binance Smart Chain tokens and dApps.**

## Table of Contents
- [Motivation](#motivation)
- [Design Principles](#design-principles)
- [Consensus and Validators](#consensus-and-validator)
  * [Proof of Staked Authority](#proof-of-staked-authority)
  * [Validators](#validator)
  * [Security and Finality](#security-and-finality)
  * [Reward](#reward)
- [Token Economy](#token-economy)
  * [Native Token](#native-token)
  * [Other Tokens](#other-tokens)
- [Cross-Chain Transfer and Communication](#cross-chain-transfer-and-communication)
  * [Cross-Chain Transfer](#cross-chain-transfer)
  * [NEXT Architecture](#next-architecture)
  * [Timeout and Error Handling](#timeout-and-error-handling)
  * [Cross-Chain User Experience](#cross-chain-user-experience)
  * [Cross-Chain Contract Event](#cross-chain-contract-event)
- [Staking and Governance](#staking-and-governance)
  * [Staking on NSC](#staking-on-nsc)
  * [Rewarding](#rewarding)
  * [Slashing](#slashing)
- [Relayers](#relayers)
  * [NSC Relayers](#nsc-relayers)
  * [Oracle Relayers](#oracle-relayers)
- [Outlook](#outlook)
# Motivation

After its mainnet community [launch] on Februari 22, [NEXT Smart Chain](https://nextsmartchain.com) has exhibited its high speed and large throughput design. NEXT’s primary focus, its native [decentralized application](https://en.wikipedia.org/wiki/Decentralized_application) (“dApp”) [NEXT DEX](https://dex.nextsmartchain), has demonstrated its low-latency matching with large capacity headroom by handling millions of trading volume in a short time.

Flexibility and usability are often in an inverse relationship with performance. The concentration on providing a convenient digital asset issuing and trading venue also brings limitations. NEXT's most requested feature is the programmable extendibility, or simply the [Smart Contract](https://en.wikipedia.org/wiki/Smart_contract) and Virtual Machine functions. Digital asset issuers and owners struggle to add new decentralized features for their assets or introduce any sort of community governance and activities.

Despite this high demand for adding the Smart Contract feature onto Next Smart Chain, it is a hard decision to make. The execution of a Smart Contract may slow down the exchange function and add non-deterministic factors to trading. If that compromise could be tolerated, it might be a straightforward idea to introduce a new Virtual Machine specification based on [Ethereum](https://ethereum.com), based on the current underlying consensus protocol and major [RPC](https://docs.binance.org/api-reference/node-rpc.html) implementation of NEXT Smart Chain. But all these will increase the learning requirements for all existing dApp communities, and will not be very welcomed.

We propose a parallel blockchain of the current NEXT Smart Chain Chain to retain the high performance of the native DEX blockchain and to support a friendly Smart Contract function at the same time.

# Design Principles

Here are the design principles of **NSC**:

1. **Decentralized Blockchain**: technically, NSC is a decentralized blockchain. Most NSC fundamental  technical and business functions should be self-contained.
2. **Ethereum and Binance Smart Chain Compatibility**: The first practical and widely-used Smart Contract platform is Ethereum and BSC. To take advantage of the relatively mature applications and community, NSC chooses to be compatible with the existing Ethereum and Binance Smart Chain mainnet. This means most of the **dApps**, ecosystem components, and toolings will work with NSC and require zero or minimum changes; NSC node will require similar (or a bit higher) hardware specification and skills to run and operate. The implementation should leave room for NSC to catch up with further Ethereum and Binance Smart Chain upgrades.
3. **Staking Involved  Consensus and Governance**: Staking-based consensus is more environmentally friendly and leaves more flexible option to the community governance. Expectedly, this consensus should enable better network performance over [proof-of-work](https://en.wikipedia.org/wiki/Proof_of_work) blockchain system, i.e., faster blocking time and higher transaction capacity.

# Consensus and Validator

Based on the above design principles, the consensus protocol of NSC is to fulfill the following goals:

1. Blocking time should be shorter than Ethereum network, e.g. 1 second or even shorter.
2. It requires limited time to confirm the finality of transactions, e.g. around 1-min level or shorter.
3. There is no inflation of native token: NEXT, the block reward is collected from transaction fees, and it will be paid in NEXT.
4. It is compatible with Ethereum system as much as possible.
5. It allows modern [proof-of-stake](https://en.wikipedia.org/wiki/Proof_of_stake) blockchain network governance.

## Proof of Staked Authority

Although Proof-of-Work (PoW) has been recognized as a practical mechanism to implement a decentralized network, it is not friendly to the environment and also requires a large size of participants to maintain the security.

Ethereum and some other blockchain networks, such as [MATIC](https://github.com/maticnetwork/bor), [TOMOChain](https://tomochain.com/), [GoChain](https://gochain.io/), [xDAI](https://xdai.io/), do use [Proof-of-Authority(PoA)](https://en.wikipedia.org/wiki/Proof_of_authority) or its variants in different scenarios, including both testnet and mainnet. PoA provides some defense to 51% attack, with improved efficiency and tolerance to certain levels of Byzantine players (malicious or hacked). It serves as an easy choice to pick as the fundamentals.

Meanwhile, the PoA protocol is most criticized for being not as decentralized as PoW, as the validators, i.e. the nodes that take turns to produce blocks, have all the authorities and are prone to corruption and security attacks. Other blockchains, such as EOS and Lisk both, introduce different types of [Delegated Proof of Stake (DPoS)](https://en.bitcoinwiki.org/wiki/DPoS) to allow the token holders to vote and elect the validator set. It increases the decentralization and favors community governance.

NSC here proposes to combine DPoS and PoA for consensus, so that:
1. Blocks are produced by a limited set of validators
2. Validators take turns to produce blocks in a PoA manner, similar to [Ethereum’s Clique](https://eips.ethereum.org/EIPS/eip-225) consensus design
3. Validator set are elected in and out based on a staking based governance

## Validators

In the genesis stage, 3 nodes will run as the initial Validator Set. After the blocking starts, anyone can compete to join as candidates to elect as a validator.

**NEXT** is the token used to stake for NSC.

In order to remain as compatible as Ethereum and upgradeable to future consensus protocols to be developed, NSC chooses to rely on the **** for staking management (Please refer to the below “[Staking and Governance](#staking-and-governance)” section). There is a **dedicated staking module for NSC**. It will accept NEXT staking from NEXT validators. 

While producing further blocks, the existing NAC validators check whether there is a `ValidatorSetUpdate` message relayed onto NSC periodically. If there is, they will update the validator set after an **epoch period**, i.e. a predefined number of blocking time. For example, if NSC produces an event every milliseconds, and the epoch period is 240 blocks, then the current validator set will check and update the validator set for the next epoch in 4 hours.

## Security and Finality

Given there are more than ½\*N+1 validators are honest, PoA based networks usually work securely and properly. However, there are still cases where certain amount Byzantine validators may still manage to attack the network, e.g. through the “[Clone Attack](https://arxiv.org/pdf/1902.10244.pdf)”. To secure as much as NSC users are encouraged to wait until receiving blocks sealed by more than ⅔\*N+1 different validators. In that way, the NSC can be trusted at a similar security level and can tolerate less than ⅓\*N Byzantine validators.

## Reward

All the NSC validators in the current validator set will be rewarded with block reward **fees in NEXT**. As NEXT is not an inflationary token, there will be no mining rewards as what Bitcoin and Ethereum network generate, and the gas fee is the major reward for validators. As NEXT is also an utility token with other use cases, delegators and validators will still enjoy other benefits of holding NEXT.

The reward for validators is the fees collected from transactions in each block. Validators can decide how much to give back to the delegators who stake their NEXT to them, in order to attract more staking. Every validator will take turns to produce the blocks in the same probability (if they stick to 100% liveness), thus, in the long run, all the stable validators may get a similar size of the reward. Meanwhile, the stakes on each validator may be different, so this brings a counter-intuitive situation that more users trust and delegate to one validator, they potentially get less reward. So rational delegators will tend to delegate to the one with fewer stakes as long as the validator is still trustful (insecure validator may bring slashable risk). In the end, the stakes on all the validators will have less variation. This will actually prevent the stake concentration and “winner wins forever” problem seen on some other networks.

Some parts of the gas fee will also be rewarded to relayers for Cross-Chain communication. Please refer to the “[Relayers](#relayers)” section below.

# Token Economy (NRC20 and NRC722)

Next Smart Chain share the same token universe for NRC20 tokens. This defines:

1. The same token can circulate on Ethereum-compatible blockchain, and flow between them bi-directionally via a cross-chain communication mechanism.
2. The total circulation of the same token should be managed across the two networks, i.e. the total effective supply of a token should be the sum of the token’s total effective supply.
3. The tokens can be initially created on NSC in a similar format as ERC20 token standard, then created on the other. There are native ways on both networks to link the two and secure the total supply of the token.

## Native Token

NEXT will run on NSC in the same way as ETH runs on Ethereum so that it remains as “native token” for NSC. NEXT will be also used to:

1. pay “fees“ to deploy smart contracts on NSC
2. stake on selected NSC validators, and get corresponding rewards
3. perform cross-chain operations, such as transfer token assets across NSC, BSC and ETH

### Seed Fund

Certain amounts of NEXT will be minted during its genesis stage. This amount is called “Circulation Pool” which is the same as the combined circulated supply on other blockchains.

The  cross-chain transfer is discussed in a later section, but for NEXT to NEXT transfer, it is generally to lock NEXT on other networks and release NEXT from the 'Circulation Pool' on NSC. On NSC from the source address of the transfer to a system-controlled address and unlock the corresponding amount from special contract to the target address of the transfer on NSC, or reversely, when transferring from NSC to ETH, it is to lock NEXT from the source address on NSC into the pool and release locked amount on BC from the system address to the target address. The logic is related to native code and a series of smart contracts on NSC.

## Other Tokens

NSC is Ethereum compatible, it is natural to support ERC20 tokens and Smart Contracts on NSC, which here is called “**NRC20**” (with the real name to be introduced by the future NRCs). NRC20 may be “Enhanced” by adding a few more methods to expose more information, such as token denomination, decimal precision definition and the owner address who can decide the Token Binding across the chains. 

# Cross-Chain Transfer and Communication
Cross-chain communication is the key foundation to allow the community to take advantage of the dual chain structure:

* users are free to create any tokenization, financial products, and digital assets on NSC as they wish
* the items on NSC can be manually and programmingly traded and circulated in a stable, high throughput, lighting fast and friendly environment
* users can operate these in one UI and tooling ecosystem.

## Cross-Chain Transfer

The cross-chain transfer is the key communication between the two blockchains. Essentially the logic is:

1. the `transfer-out` blockchain will lock the amount from source owner addresses into a system controlled address/contracts;
2. the `transfer-in` blockchain will unlock the amount from the system controlled address/contracts and send it to target addresses.

The cross-chain transfer package message should allow the NEXT Relayers and NEXT **Oracle Relayers** to verify:

1. Enough amount of token assets are removed from the source address and locked into a system controlled addresses/contracts on the source blockchain. And this can be confirmed on the target blockchain.
2. Proper amounts of token assets are released from a system controlled addresses/contracts and allocated into target addresses on the target blockchain. If this fails, it can be confirmed on source blockchain, so that the locked token can be released back (may deduct fees).
3. The sum of the total circulation of the token assets across the 2 blockchains are not changed after this transfer action completes, no matter if the transfer succeeds or not.

![cross-chain](./assets/cross-chain.png)

The architecture of cross-chain communication is as in the above diagram. To accommodate the 2 heteroid systems, communication handling is different in each direction.

## Cross-Chain User Experience

Ideally, users expect to use two parallel chains in the same way as they use one single chain. It requires more aggregated transaction types to be added onto the cross-chain communication to enable this, which will add great complexity, tight coupling, and maintenance burden. Here NSC only implement the basic operations to enable the value flow in the initial launch and leave most of the user experience work to client side UI, such as wallets. E.g. a great wallet may allow users to sell a token directly from NSC onto NEXT’s DEX order book, in a secure way.

# Staking and Governance

Proof of Staked Authority brings in decentralization and community involvement. Its core logic can be summarized as the below. You may see similar ideas from other networks, especially Cosmos and EOS.

1. Token holders, including the validators, can put their tokens “**bonded**” into the stake. Token holders can **delegate** their tokens onto any validator or validator candidate, to expect it can become an actual validator, and later they can choose a different validator or candidate to **re-delegate** their tokens<sup>1</sup>.
2. All validator candidates will be ranked by the number of bonded tokens on them, and the top ones will become the real validators.
3. Validators can share (part of) their blocking reward with their delegators.
4. Validators can suffer from “**Slashing**”, a punishment for their bad behaviors, such as double sign and/or instability.
5. There is an “**unbonding period**” for validators and delegators so that the system makes sure the tokens remain bonded when bad behaviors are caught, the responsible will get slashed during this period.

## Staking on NEXT

Ideally, such staking and reward logic should be built into the blockchain, and automatically executed as the blocking happens.

NSC has been preparing to enable staking logic since the design days. On the other side, as NSC wants to remain compatible with Ethereum as much as possible, it is a great challenge and efforts to implement such logic on it. This is especially true when Ethereum itself may move into a different Proof of Stake consensus protocol in a short (or longer) time. In order to keep the compatibility and reuse the good foundation of BC, the staking logic of NSC:

1. The staking token is NEXT, as it is a native token on both blockchains anyway
2. The staking, i.e. token bond and delegation actions and records happens on NSC
3. The NSC validator set is determined by its staking and delegation logic, via a staking module built-in the blockchain.
4. The reward distribution happens with every EPOCH around every 4 hours.

## Rewarding

Both the validator update and reward distribution happen every 4 hours. This is to save the cost of frequent staking updates and block reward distribution. This cost can be significant, as the blocking reward is collected on NSC and distributed on validators and delegators.

1. The blocking reward will not be sent to validator right away, instead, they will be distributed and accumulated on a contract function called 'ClaimRewards';
2. Upon receiving the validator set update into NSC, it will trigger a few cross-chain transfers to transfer the reward to custody addresses on the corresponding validators. The custody addresses are owned by the system so that the reward cannot be spent until the promised distribution to delegators happens.
3. In order to make the synchronization simpler and allocate time to accommodate slashing, the reward for N day will be only distributed in N+2 days. After the delegators get the reward, the left will be transferred to validators’ own reward addresses.

###  Governance Parameters

There are many system parameters to control the behavior of the NSC, e.g. cross-chain transfer fees. All these parameters will be determined by NSC Validator Set together through a proposal-vote process based on their staking. Such the process will be carried on NSC, and the new parameter values will be picked up by corresponding system contracts via a cross-chain communication.

# Relayers

Relayers are responsible to submit Cross-Chain Communication Packages between the two blockchains. Due to the heterogeneous parallel chain structure, two different types of Relayers are created.

## NEXT Relayers

Relayers for NSC communication referred to as “**NSC Relayers**”, or just simply “Relayers”. Relayer is a standalone process that can be run by anyone, and anywhere, except that Relayers must register themselves onto NSC and deposit a certain refundable amount of NEXT. Only relaying requests from the registered Relayers will be accepted by NSC. 

The package they relay will be verified by the on-chain light client on NSC. The successful relay needs to pass enough verification and costs gas fees on NSC, and thus there should be incentive reward to encourage the community to run Relayers.

### Incentives

There are two major communication types:

1. Users triggered Operations, such as `token bind` or `cross chain transfer`. Users must pay additional fee to as relayer reward. The reward will be shared with the relayers who sync the referenced blockchain headers. Besides, the reward won't be paid the relayers' accounts directly. A reward distribution mechanism will be brought in to avoid monopolization.
2. System Synchronization, such as delivering `refund package`(caused by failures of most oracle relayers), special blockchain header synchronization, NSC staking package. System reward contract will pay reward to relayers' accounts directly.

If some Relayers have faster networks and better hardware, they can monopolize all the package relaying and leave no reward to others. Thus fewer participants will join for relaying, which encourages centralization and harms the efficiency and security of the network. Ideally, due to the decentralization and dynamic re-election of validators, one Relayer can hardly be always the first to relay every message. But in order to avoid the monopolization further, the rewarding economy is also specially designed to minimize such chance:

1. The reward for Relayers will be only distributed in batches, and one batch will cover a number of successful relayed packages.
2. The reward a Relayer can get from a batch distribution is not linearly in proportion to their number of successful relayed packages. Instead, except the first a few relays, the more a Relayer relays during a batch period, the less reward it will collect.

## Oracle Relayers

Relayers for NSC communication are using the “Oracle” model, and so-called “**Oracle Relayers**”. Each of the validators must, and only the ones of the validator set, run Oracle Relayers. Each Oracle Relayer watches the blockchain state change. Once it catches Cross-Chain Communication Packages, it will submit to vote for the requests. After Oracle Relayers from ⅔ of the voting power of validators vote for the changes, the cross-chain actions will be performed.

Oracle Replayers should wait for enough blocks to confirm the finality on NSC before submitting and voting for the cross-chain communication.

The cross-chain fees will be distributed to NSC validators together with the normal blocking rewards.

Such oracle type relaying depends on all the validators to support. As all the votes for the cross-chain communication packages are recorded on the blockchain, it is not hard to have a metric system to assess the performance of the Oracle Relayers. The poorest performer may have their rewards clawed back via another Slashing logic introduced in the future.

# Outlook

It is hard to conclude for NEXT Smart Chain, as it has never stopped evolving. The strategy is to open the gate for users to take advantage of the fast transferring and trading on one side, and flexible and extendable programming on the other side, but it will be one stop along the development of Next Smart Chain. Here below are the topics to look into so as to facilitate the community better for more usability and extensibility:

1. Add different digital asset model for different business use cases
2. Enable more data feed, especially DEX market data, or powered by Oracle feeds
3. Provide interface and compatibility to integrate with Ethereum, including its further upgrade, and other blockchain
4. Improve client side experience to manage wallets and use blockchain more conveniently
