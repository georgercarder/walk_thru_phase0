#### Eth 2.0: A walk through Phase 0

// TODO INCLUDE LINKS

With the eagerly anticipated release of various beacon-chain testnets we were curious to get a hands-on introduction to the various entities in the Eth 2.0 protocol. There is a great deal written online about Eth 2.0 and facets of its various phases so we figure the simplest way for us to get a unified view into Phase 0 of this upgrade is to just follow our nose and trace the interactions of some existing clients. 

This is an Ethereum blog, so we assume the reader is familiar with Ethereum as a platform as well as the motivation for the upgrade to Eth 2.0. If not, check out these links to get up to speed.

The Eth2.0 upgrade is to be rolled out in phases. The large dedicated community designing and engineering this upgrade has broken the roll out into 4 phases:

- Phase 0: the beacon chain (the current phase)

- Phase 1: the shard chains

- Phase 1.5: mainnet becomes a shard

- Phase 2: fully formed shards

In this exploration we will focus on the current phase, Phase 0. Our goal is to learn about each node in this upgrade. We'll do this by building and running the respective software, describing its role in this phase (Phase 0) and the ultimate roll-out (Phase 2), and actually tracing its interactions within the system.

The guide we are following is the Eth2 Launch Pad for the Medalla testnet .

We will be setting up 3 nodes:

- eth1 node

- beacon node

- validator node

There are also auxiliary steps we will take to initialize our "fleet" of nodes. 

We will be installing and running these nodes from the `bash` shell of a Linux system. The order in which we set these nodes up matters to a certain extent, so we suggest following the order of this exploration.

First, let's set up the Eth1 node.

By "Eth1" node, we mean a machine running the software of the ethereum protocol before the roll-out of this upgrade. Typical node software for this role will validate, store, gossip blocks and pending transactions, and form its own copy of the ethereum blockchain. Currently the mainnet of this paradigm relies on Proof of Work consensus. But there are testnets that rely on alternative consensus schemes such as Goerli testnet which relies on Proof of Authority. When one of these nodes is enabled to be a miner it is incentivized to partake in the additional task of mining blocks, which consists of validating transactions, collating them in blocks, then signing and broadcasting the block in the hopes other nodes will recognize it as valid and include it in their own blockchains as dictated by the consensus mechanism. As we know, one of the great features of the ethereum platform is the distributed EVM runtime environment for smart contracts where the aforementioned transactions record transitions in the state dictated by the smart contract logic. One of these nodes can also be configured to provide an RPC service to clients. This RPC service can be configured to provide both read and write access to the blockchain state. Recall that with every blockchain write, there must be an associated, well-formed transaction.

The Eth1 node implementation we choose is `go-ethereum` (`geth`) pointing to the "Goerli" testnet since this is the blockchain the Medalla testnet refers to. 

In the former paradigm ("Eth 1.0") we treated the "Eth1" nodes and their associated blockchain as first-class members of the system. They reach consensus on the global state of the system and provide its data to clients. As stated above, the "Eth 1.0" mainnet uses PoW as its consensus mechanism. But the design goals of Eth2.0 will relegate the "Eth1" nodes and their associated blockchain to a "shard" within a greater system, where the consensus will be reached by Proof of Stake.

For this transitional Phase 0 step in the Eth2.0 upgrade, the "Eth1" nodes and their associated chain still provide an elevated service to the network by recording the smart contract transactions associated with the staking of validator nodes. Details of this staking will be in the "Validator node" section below. 

We setup this Eth1 node:

```
git clone https://github.com/ethereum/go-ethereum.git && cd go-ethereum
latest_tag=$(git describe --tags)
git checkout $latest_tag
make install
```

This clones the `go-ethereum` repo, checks out to the latest release, and builds and installs an executable from source.

It is advised to run the node in a "screen" so we may detach and do other work in the same console.

```
screen -dmS goerli
screen -S goerli -X stuff 'geth --goerli console'
```

This is the minimal command to start the "Eth1" node syncing to the Goerli testnet. It may take several hours for this node to completely sync. Depending on the blockchain you are syncing you many need to run this on a system with a large amount of storage. On our device, we use a 1TB SSD. Various flags may be used for additional functionality, for instance you may want graphQL, or web-sockets RPC calls or to read/write to a custom data directory. `geth --help` will give you a list of the available flags. It may also be worthwhile to setup a firewall if your node is serving an RPC. A very simple, lightweight, and effective firewall can be setup using `iptables` which is available on many systems.

Next, we set up the Beacon node.

	/// TODO what it is (and which one we choose)

	/// TODO how it will eventually fit in

	/// how it currently fits in (phase 0)

	/// build, configure, run notes

	/// sync


Now, we set up the Validator node.

	/// what it is (and which one we choose). talk about open-standard

	/// how it will eventually fit in

	/// how it currently fits in (phase 0)

	/// build, configure, run notes

	// don't run yet

Finally, we work through the auxiliary steps.
 
	// keys, tx, and why

	// run validator node

---

// looking forward

---

// conclusion

