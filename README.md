#### Eth 2.0: A walk through Phase 0

// () means "TODO put link here"

With the /* TODO HUMAN SOUNDING ADJECTIVES explaining leadup */ of various beacon-chain testnets () we were curious to get a hands-on introduction to the various entities in the Eth 2.0 protocol (). There is a great deal written online about Eth 2.0 () and facets of its various phases so we figure the simplest way to get a view into Phase 0 of this upgrade is to just follow our nose and trace the interations of some existing clients. 

This is an Ethereum () blog, so we assume the reader is familiar with Ethereum as a platform as well as the motivation for the upgrade to Eth 2.0. If not, checkout these links () to get up to speed.

// rough outline of upgrade and phases here TODO

Our goal is to learn about each node in this upgrade. We'll do this by building and running the respective software, describing its role in this phase (Phase 0) and the ultimate role-out (Phase 2), and actually tracing its interactions within the system.

The guide we are following is the Eth2 Launch Pad for the Medalla testnet ().

//// !!! bulk of article will be focused here !!!

// quick list of cast of characters
We will be setting up 3 nodes:
	- eth1 node
	- beacon node
	- validator node

There are also auxiliary steps we will take to initialize this system. 

First, let's set up the Eth1 node.

By "Eth1" node, we mean a machine running the software of the ethereum protocol before the roll-out of this upgrade. Typical node software for this role will validate, store, and gossip blocks of its own copy of the ethereum blockchain. When one of these nodes is enabled to be a miner it is incentivized to partake in the additional task of mining blocks, which consists of validating transactions, including them in blocks, finding a nonce to meet a target, signing then broadcasting the block in the hopes other nodes will recognize it as valid and include it in their own blockchains. As we know, one of the great features of the ethereum platform is the distributed EVM runtime environment for smart contracts where the aforementioned transactions record transitions in the state dictated by the smart contract logic.

The Eth1 node implementation we choose is `go-ethereum` (`geth`) pointing to the "Goerli" testnet since this is the blockchain the Medalla testnet refers to. 

In the former paradigm ("Eth 1.0") we treated the "Eth1" nodes and their associated blockchain as first-class members of the system. They reach consensus on the global state of the system and provide its data to clients. But the design goals of Eth2.0 will relegate the "Eth1" nodes and their associated blockchain to a "shard" within a greater system. // TODO describe this MORE

For this Phase 0 step in the Eth2.0 upgrade, the "Eth1" nodes and their associated chain still provide an elevated service to the network by recording the smart contract transactions associated with the staking of validator nodes. Details of this staking will be in the "Validator node" section below. 

	///  build, configure, run notes

	/// sync

Next, we set up the Beacon node.

	///  TODO what it is (and which one we choose)

	///  TODO how it will eventually fit in

	///  how it currently fits in (phase 0)

	///  build, configure, run notes

	/// sync


Now, we set up the Validator node.

	///  what it is (and which one we choose). talk about open-standard

	///  how it will eventually fit in

	///  how it currently fits in (phase 0)

	///  build, configure, run notes

	// don't run yet

Finally, we work through the auxiliary steps.
 
	// keys, tx, and why

	// run validator node

---

// looking forward

---

// conclusion

