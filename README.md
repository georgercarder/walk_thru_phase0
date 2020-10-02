#### Eth 2.0: A walk through Phase 0

// TODO INCLUDE LINKS, AND PICTURES

// TODO EDIT FOR typos, brain-os, grammar

// TODO review all "Eth1.0" vs eth1 labels (same for 2.0)

With the greatly anticipated release of various beacon-chain testnets we were curious to get a hands-on introduction to the various entities in the Eth 2.0 protocol. There is a great deal written online about Eth 2.0 and facets of its various phases so we figure the simplest way for us to get a unified view into Phase 0 of this upgrade is to just follow our nose and trace the interactions of some existing clients. 

This is an Ethereum blog, so we assume the reader is familiar with Ethereum as a platform as well as the motivation for the upgrade to Eth 2.0. If not, check out these links to get up to speed.

[Ethereum.org website](https://ethereum.org/en/)

[Ethereum 2.0 overview](https://ethereum.org/en/eth2/)

The Eth2.0 upgrade is to be rolled out in phases. The large dedicated community that has assembled to design and engineer this upgrade has broken the roll-out into 4 phases:

- Phase 0: the beacon chain (the current phase)

- Phase 1: the shard chains

- Phase 1.5: mainnet becomes a shard

- Phase 2: fully formed shards

In this exploration we will focus on the current phase, Phase 0. Our goal is to learn about each entity in this upgrade. We'll do this by building and running the respective software, describing both its role in the ultimate roll-out (Phase 2) and its functionality specific to this phase (Phase 0), and actually tracing its interactions within the system.

The guide we are loosely following is the [Eth2 Launch Pad for the Medalla testnet](https://medalla.launchpad.ethereum.org/).

We will be setting up 3 entities:

- eth1 node

- beacon node

- validator node

There are also auxiliary steps we will take to initialize this "fleet". 

We will be installing and running these nodes from the [`bash` shell](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) of a [Linux\* system](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2F2.bp.blogspot.com%2F-JEkCj8lr_DM%2FT9OHAncut6I%2FAAAAAAAADfg%2Fv14RMt9LRdE%2Fs1600%2Frms-linus.jpg&f=1&nofb=1). All nodes will be run under the same host. The order in which we set these nodes up matters to a certain extent, so we suggest following the order of this exploration.

### First, let's set up the Eth1 node.

By "Eth1" node, we mean a machine running the software of the ethereum protocol before the roll-out of this upgrade (as of this writing, a good reference point is [go-ethereum <= 1.9.22](https://github.com/ethereum/go-ethereum/releases/tag/v1.9.22)). Typical node software for this role will validate, store, gossip blocks and pending transactions, and form its own copy of the ethereum blockchain. Currently the [mainnet](https://etherscan.io/) of this paradigm relies on [Proof of Work (PoW) consensus](https://www.geeksforgeeks.org/proof-of-work-pow-consensus/). But there are testnets that rely on alternative consensus schemes such as [G&ouml;rli testnet](https://goerli.net/) which relies on [Proof of Authority](https://forum.openzeppelin.com/t/proof-of-authority/3577). When one of these nodes is enabled to be a miner it is incentivized to partake in the additional task of mining blocks, which consists of validating transactions, collating them in blocks, then signing and broadcasting the block in the hopes other nodes will recognize it as valid and include it in their own blockchains as dictated by the consensus mechanism. As we know, one of the great features of the ethereum platform is the distributed EVM runtime environment for smart contracts where the aforementioned transactions induce transitions in the state dictated by the smart contract logic. One of these nodes can also be configured to provide an RPC service to clients. This RPC service can be configured to provide both read and nuanced write access to the blockchain state. Recall that with every blockchain state write, there must be an associated valid transaction.

The Eth1 node implementation we choose is [`go-ethereum` (`geth`)](https://github.com/ethereum/go-ethereum) pointing to the "G&ouml;rli" testnet since this is the blockchain the Medalla testnet refers to. 

In the former paradigm ("Eth 1.0") we treated the "Eth1" nodes and their associated blockchain as first-class members of the system. They reach consensus on the global state of the system, provide its data to clients, and allow clients to update the state through valid transactions. As stated above, the "Eth 1.0" mainnet uses PoW as its consensus mechanism. But the design goals of Eth2.0 will relegate the "Eth1" nodes and their associated blockchain to a "shard" within a greater system, where the consensus will be reached by Proof of Stake.

For this transitional Phase 0 step in the Eth2.0 upgrade, the "Eth1" nodes and their associated chain still provide an elevated service to the network by recording the smart contract transactions associated with the staking of validator nodes. Details of this staking will be in the "Validator node" section below. 

We set up this Eth1 node:

```
git clone https://github.com/ethereum/go-ethereum.git && cd go-ethereum
latest_tag=$(git describe --tags)
git checkout $latest_tag
make install
```

This clones the `go-ethereum` repo, checks out to the latest release, and builds and installs an executable from source.

It is advised to run the node in a "screen" so we may detach and do other work in the same console.

```
screen -dmS gethnode-goerli
screen -S gethnode-goerli -X stuff 'geth --goerli console'
```

This is the minimal command to start the "Eth1" node syncing to the G&ouml;rli testnet. Recall that syncing for an eth1 node consists of importing, validating, and assembling blocks into a locally stored blockchain where the blocks wrap data referencing actual transactions. It may take several hours for this node to completely sync. Depending on the blockchain you are syncing you many need to run this on a system with a large amount of storage. On our device, we use a 1TB SSD. Various flags may be used for additional functionality, for instance you may want graphQL, or web-sockets RPC calls or to read/write to a custom data directory. `geth --help` will give you a list of the available flags. It may also be worthwhile to setup a firewall if your node is serving an RPC. A very simple, lightweight, and effective firewall can be setup using `iptables` which is available on many systems.

![goerli_logs.png](img/goerli_logs.png?raw=true)

### Next, we set up the Beacon node.

If we think of the eth1 nodes and the ethereum protocol before the roll-out of this upgrade as a worm (no offense "Eth1.0"), then it would make sense to think of beacon nodes and the eth2 protocol as an evolution of the *system* to a Millipede. But don't get frightened. While the structural complexity is indeed increased, let's imagine this Millipede is cute, approachable and very helpful (kind of like how people somehow managed to make the alien from Aliens cute ???). A beacon node is a machine running "Eth2.0" software that builds and communicates data related to its beacon chain, a blockchain. Unlike the "Eth1.0" chain whose blocks wrap data referencing actual transactions, this beacon chain is composed of blocks wrapping data referencing the consensus mechanism (think meta!) on a sharded configuration of the former "Eth1.0 style" blocks. From here on out, we will call these "meta" blocks "beacon blocks" and the lesser "Eth1.0 style" blocks "shard blocks". This includes managing data on the agents that participate in the consensus mechanism, the validators. These beacon nodes and their beacon chain make up the body and the validators make up the seemingly innumerable set of legs doing the "footwork" of our Millipede friend. This metaphor will be clarified in the following sections, so if you're unsure of it's accuracy, just wait.

The "beacon node" implementation we choose for this exploration is Sigma Prime's "Lighthouse: Ethereum 2.0" client. This beacon node will be pointing to the `geth` eth1 node we set up in the last section.

As stated above, a beacon node's duty is to maintain and communicate the beacon chain. From a very high level, the ultimate roll-out of Eth2.0 will have the state of this chain consisting of the membership status of validators, and the accumulated attestations to blocks. Think of an "attestation" as a "vote". But be careful in noting that these attestations are simultaneously votes for a shard block, and proof-of-stake votes for a beacon block. 

For Phase 0, the beacon nodes only accumulate votes for beacon blocks but not for shard blocks. Since the only "shard" at this point is the eth1 chain, whose consensus is already handled by its PoW. Still, the beacon nodes' beacon blocks have an entry for all attestations in preparation for upcoming phases. Also, the beacon nodes manage validator membership by processing logs resulting from deposits made to the eth1 deposit contract, and by collating proof of slashable offenses gathered by "slashers". Slashers are special nodes whose duty is to gather proof of slashable offenses and communicate the proof to beacon nodes. Slashable offenses will be covered in the validator section below.

We set up this beacon node:

```
git clone https://github.com/sigp/lighthouse.git && cd lighthouse
latest_tag=$(git describe --tags)
git checkout $latest_tag
make install
```

This clones the `lighthouse` repo, checks out to the latest release, and builds and installs an executable from source.

It is advised to run the beacon node in a "screen" so we may detach and do other work in the same console.

```
screen -dmS lighthouse-beacon-node 
screen -S lighthouse-beacon-node -X stuff 'lighthouse beacon_node --staking'
```

This is the minimal command to start the beacon node syncing while making available an endpoint for communication with validators. Fortunately, this "lighthouse" beacon node client points to the default port of our `geth` G&ouml;rli node "out of the box". Again, like the eth1 node, the syncing could take several hours. Like the eth1 node, the syncing process for this eth2 node builds its respective beacon blockchain by validating and combining data from beacon node peers as well as eth1 chain data resulting from the validator Deposit contract so that it may form and maintain a beacon state. Details of these deposit transactions will be elaborated on in the validator section. Data from these deposit logs must be processed in sequential order. There is added complexity in this node due to the maintaining of a full deposit Merkle Tree and computing updated proofs against other deposits as needed. 

![beacon_logs.png](img/beacon_logs.png?raw=true)

### Now, we set up the Validator node.

Recall from the last section where we illustrated this new infrastructure zoomorphized as a Millipede. The beacon nodes and their chain make up the body, or spine of the system, and the validator nodes are the innumerable feet doing the "footwork" of the system. But, we must be careful with this metaphor: *The former "Eth1.0" worm does NOT embed into the body of the Millipede.* Rather this Millipede is carnivorous and as it marches across the terrain it collects and processes worms section by section (block by block) with its feet and eventually passes the result to its mouth. Still, I PROMISE this Millipede is cute and friendly. Bear with me.

These worm snacks however, have a lot of variety. There is an "Eth1.0" worm as well as other variants. They are all blockchains, but only one of them is an "Eth1.0" blockchain. These worm snacks are "shards" in the Eth2.0 system. Blocks from these shards will be processed in this way at the full roll-out of the Eth2.0 protocol. For now, the "Eth1.0" chain informs the Millipede of which legs it has to do the "footwork", but is not yet feasted on.

These validators who do the "footwork" of the Eth2.0 protocol are economically incentivized to behave honestly (adhering to the protocol) in that they can either be rewarded or slashed for their behavior. The deposit of up to 32 ETH that each of these validators make in order to act as a validator agent has many effects in the design of the system. For one, it provides Sybil resistance for the protocol in that a bad actor cannot cheaply flood the system with validator nodes in order to either compromise the state or hoard the profits from rewards. Also, this stake is a deposit that can be slashed if the validator behaves incorrectly. Finally, a rational validator would not jeopardize the value of their ETH holdings by undermining the system. This stake is made by calling a deposit function of a Deposit smart contract deployed to the eth1 network (G&ouml;rli in the Medalla case). This transaction includes a payment for stake, a commitment to a public key for the eventual withdrawal, and other data. The receipts from this transaction are processed by the beacon chain in order to update validator status. We will walk through our own deposit transaction in the following "auxiliary" section.

Once staked, Phase 0 of Eth2.0 protocol expects that the validator has two primary responsibilities: proposing blocks (beacon blocks) and creating attestations. There are special units of time within this protocol that dictates the behavior of its actors. The units we focus on now are the slot (12 sec) and the epoch (32 slots). At the beginning of any slot, a validator checks its role by calling a function whose return value is determined by the (pseudo)-identity and other attributes of the validator as it relates to global state. This function simply tells the validator whether it is to propose a block or not. If so, they propose a block. These shard blocks are very similar to the old Eth1.0 blocks with the enhancement that they reference the beacon block that, in their view of the fork choice, is the head of the chain during slot-1. Per usual they create, sign, and broadcast this block with reference to the aforementioned beacon block parent so long as it is a valid beacon chain state transition. If not, the validator simply creates attestations. These attestations are a vote, or an endorsement, of a valid block which the validator has received during a certain slot. The way in which a validator performs these actions can leave it susceptible to slashing. When proposing, the validator can avoid slashing by not signing conflicting blocks for the same slot. If a validator attests to two conflicting blocks, they will be slashed. The shard blocks a validator is in charge of validating/attesting will be determined by a "committee assignment" which is a function of the state, epoch and the validators' index.

For Phase 0, the validators process the beacon blocks but will NOT process the shard blocks until a later phase. To return to our friend the Millipede, the legs (validators) work with the body (beacon blocks) doing the "footwork", but will not feast on the segments of worms (shard blocks) until a later phase. Since Phase 0 does not have shards there is not currently a way to define shard committees. To account for this, additional programmatic safeguards are put into the Phase 0 protocol to provide attestation subnet stability. 
 
An open-standard for an API interface exposed by a beacon node has been in the works to facilitate Phase 0. However, for this exploration we err on the side of caution and choose our validator node implementation to use the same software as our beacon node implementation, Sigma Prime's "Lighthouse: Ethereum 2.0" client. This way we do not need to install any additional software or debug interoperability issues. Unlike the other sections of this exploration, we will put off running this validator node until we complete other steps in the sequel "auxiliary" section. There, the explicit commands to run the validator node will be stated after other essential tasks are completed.

### Finally, we work through the auxiliary steps.
 
As outlined in the introduction, we were tasked with setting up 3 entities. So far we have installed the software for all three, but have only activated two of the three. Activation for both the eth1, and the beacon node, was straight-forward and amounted to calling an executable with specific flags. The nodes bootstrapped into their respective p2p networks so they may gossip data to build their respective datastores. Other than running their software, these two nodes do not have any additional barriers to act as their role in their respective network. This is not the case for the validator node. As we saw in the validator section, the validator will not be recognized as a valid, active participant unless a deposit transaction is made to the appropriate smart contract on its behalf with its receipt processed by the beacon chain.

For Phase 0, the validator deposits must be made to the `DepositContract` deployed to the G&ouml;rli testnet. The specific member function that needs to be called is the `deposit` function whose input parameters are `pubkey`, `withdrawal_credentials`, `signature`, and `deposit_data_root`. Let's call these values the "deposit data". It is currently not clear in this context how exactly we are to construct the deposit data. Fortunately the `eth2.0-deposit-cli` can generate these values for those who are not keen at scripting their construction themselves. We will use this tool to prepare both the deposit data as well as the keystore for our validator. We will also briefly explain what each of these values are in this context.

We download, build and install the `eth2.0-deposit-cli`

```
git clone https://github.com/ethereum/eth2.0-deposit-cli.git && cd eth2.0-deposit-cli
latest_tag=$(git describe --tags)
git checkout $latest_tag
pip3 install -r requirements.txt
python setup.py install
```

Then we run 

```
./deposit.sh
```

This script will guide us through the construction of keys and deposit data for our validator node and save the artifacts to a `validator_keys` folder within the working directory.

The `pubkey` is the validator public key.

The `signature` is a proof of possession (a BLS12-381) signature. 

The `withdrawal_credentials` is a commitment to a withdrawal public keyset that enables a validator to move their balance (in phases 1/2).

The `deposit_data_root` is a value used to check the integrity of the relationship between the `pubkey`, `signature`, and `withdrawal_credentials`.

But it is not necessary that the typical user know the meaning of these values.

Fortunately, the guide that we have been loosely following (the Eth2 Launch Pad for the Medalla testnet), provides as one of its steps a dApp interface that will make this call to `deposit`, setting as parameters entries from the "deposit data" we just generated. We simply need to drag and drop the `deposit_data-*.json` file into their UI, and complete the transaction using Metamask. Be sure to have a wallet with G&ouml;rli ETH tokens set up in Metamask. This wallet is unrelated to any of the keysets we've discussed so far. The only stipulation is that this transaction sends the amount of N * 32 G&ouml;rli ETH where N is the number of validators you indicated in the `./deposit.sh` step. This social faucet is a great way to get 32+ G&ouml;rli ETH https://faucet.goerli.mudit.blog/.

![launchpad_dragndrop.png](img/launchpad_dragndrop.png?raw=true)

Now we start our validator node!

First we must import our validator keyset

```
lighthouse account validator import --directory <path_to>/validator_keys
```

Then we start our validator node

```
lighthouse vc
```

Since this validator node is under the same host as our beacon node, this application is configured so the validator node points to the beacon node API "out of the box".

### Let's review our system 

As promised, now all our 3 entities are up and running, and our eth1 node and beacon node are fully synced.

![all_logs.png](img/all_logs.png?raw=true)

But we see from our validator the persistently repeating log `Awaiting activation`.

![validator_logs.png](img/validator_logs.png?raw=true)

Let's trace our setup from the vantage point of the block explorers for both the G&ouml;rli and beacon blockchains.

For this exploration, I used a fresh wallet for the Metamask transaction:

![eth_account.png](img/eth_account.png?raw=true)

The only outbound transaction from this address is to the Medalla Beacon Contract which we made in the last section. The validator public key associated with this transaction is noted in the Beacon Chain Deposit entry:

![depost_tx_success.png](img/deposit_tx_success.png?raw=true)

We see here that despite the transaction was made Sep-28-2020, on Oct-2-2020 validator status is that it is awaiting activation.

![validator_pending.png](img/validator_pending.png?raw=true)

We believe a deeper exploration into the mechanics of the activation process is in store. We also think it will be very interesting to trace the details of the validator's activities once it is activated. We have learned a lot here in that we have covered some key players in this protocol upgrade and how they are expected to behave in this current phase. It does seem worthwhile to continue this exploration but we will wait until the prop of an activated validator is available. According to these logs it should be just a few days. *Stay tuned for the second part of this exploration where we dive more deeply into the validator activation process!*

*\* GNU/Linux system

*Note: The worm vs Millipede metaphor can indeed be extended to accommodate forks in all blockchains of this system but for now we'll avoid a dive into talking about this system within a futuristic higher-dimensional space. The reader is welcome to run with it in the comments... :)*
