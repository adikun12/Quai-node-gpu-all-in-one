# Quai-node-gpu-all-in-one
A simple guide for QUAI node &amp; Gpu Miner

# Run A Node
we’ll be installing go-quai, the Go implementation of Quai Network. This tutorial is focused on Linux distributions.

Officials say Running go-quai on Windows or WSL2 is not currently supported.But I personally running on WSL2 smoothly.

*Requirements*

The Golden Age Testnet only consists of a single slice. This makes global and single slice nodes effectively equivalent for the duration of the testnet.

To run a slice node during the Golden Age Testnet, you’ll need a Linux machine with the following specifications:

Fast CPU with 8+ cores
16GB+ RAM
Fast SSD with at least 500GB free space
10+ MBit/sec download Internet service

*Install Dependencies*
To run an instance of go-quai, you’ll need to install a few dependencies. You can install dependencies with your favorite package manager (apt, brew, etc.). The following dependencies are required to run a node:

1
Go v1.23.0+

Snap is not default installed on all Linux distros


  # install snapd if you don't have it already
  sudo apt install snapd

  # install go
  sudo snap install go --classic
If you’re not on Ubuntu or MacOS, instructions on how to install go directly can be found on the golang installation page.

2
Install git, make, and g++ with the following command:

# install git and make
sudo apt install git make g++

3
go-quai

# clone repo
git clone https://github.com/dominant-strategies/go-quai
cd go-quai

Now you must checkout the latest release.

You can find the latest release on the go-quai from https://github.com/dominant-strategies/go-quai/tags. 
Then, check out the latest release with (replace the put-latest-release-here with the latest release number):


git checkout put-latest-release-here
For example (this is not the latest release, check the releases page for the latest release number)


git checkout v1.2.3

*Node Configuration*
Slice Node


Environment Variables
There are a few key variables required to run a Quai node. They will be passed as arguments in the start command.

quai-coinbases and qi-coinbases: the addresses in each ledger that block rewards and miner tips are paid to.
miner-preference: the percentage of block rewards that should be paid out on average in Quai or Qi tokens.
slices: the slices of the network that the node will run.
There are a number of more advanced parameters that can be passed as arguments that will not be covered in this article.
1
Configure Mining Addresses

Coinbases will be passed to the start command similar to below, with your own addresses for the chains that you intend to mine. You can generate addresses for each shard and ledger easily with Pelagus Wallet.

You must generate unique Quai and Qi addresses for each shard your node is running and pass them as coinbase flags to the run command. There is a unique coinbases flag for each ledger:

quai-coinbases: Coinbases for Quai ledger
qi-coinbases: Coinbases for Qi ledger

# single slice node running cyprus1, one quai address + one qi address
--node.quai-coinbases '0x0000000000000000000000000000000000000000'
--node.qi-coinbases '0x0080000000000000000000000000000000000000'
The Qi mining address starts with a “0x00…”, this is not to be confused with the Qi payment address. You can find the Qi mining address in the settings of the Pelagus Wallet.

2
Block Reward Prefernce

The Quai protocol can payout block rewards and miner tips in either Quai or Qi tokens. While miners have no ability to determine what token their miner tips are paid out in, the do have the ability to set their block reward payout token preference to either Quai or Qi.

Block reward token preference can be set using the miner-preference flag. The miner-preference flag is a percentage scale that can be set to values between 0 and 1, indicating the proportion of block rewards that should be paid out in Quai or Qi tokens.

Some examples:

0: 100% Quai preference, all block rewards are paid out in Quai
0.25: 3/1 Quai preference
0.5: Even split, on average block rewards are paid out equally in Quai and Qi
0.75: 3/1 Qi preference
1: 100% Qi preference, all block rewards are paid out in Qi
Pass the miner-preference flag in the start command with a value between 0 and 1 like below:


# no preference (default)
--node.miner-preference 0.5

# 100% Quai preference
--node.miner-preference 0

# 100% Qi preference
--node.miner-preference 1
3
Reward Lockup Period

The Quai protocol immediately pays out block rewards as blocks are mined, but are subject to a lockup period.

Quai Block rewards are sent to the coinbase of the miner after the lockup period has elapsed.
Qi Block reward tokens are sent to the coinbase of the miner and register as balance, but are deemed “not spendable” until they are unlocked.
The lockup period is configurable by miners using the --node.coinbase-lockup flag. The protocol provides additional incentives for miners to lock up their block rewards for a longer period of time.

The available values for --node.coinbase-lockup and their corresponding period and reward boost are:

Value	Period (blocks)	Period (days)	Reward Boost
0	7200	~0.41	+0%
1	30240	1.75	+4.17%
2	60480	3.5	+10%
3	120960	7	+25%
Pass the --node.coinbase-lockup flag in the start command like below:


# minimum lockup (default)
--node.coinbase-lockup 0

# maximum lockup
--node.coinbase-lockup 3
4
Slices

Set the node.slices flag in your run command to whichever slices of the network you would like to run.

In the codebase, a slice is identified by its region and zone index. Region and zone indices are 0-indexed and range from 0-2.

The Golden Age Testnet only supports the [0 0] slice.

# single slice node running cyprus1
--node.slices '[0 0]'
5
Genesis Nonce

To connect to the colosseum network, you must have the correct genesis nonce. The nonce acts as a sort of password that allows your node to correctly compute the first canonical block in the chain.

You’ll pass the genesis nonce to your node on start-up using the --node.genesis-nonce flag.


# Golden Age Testnet geneis nonce
--node.genesis-nonce 6224362036655375007

Starting a Node

Build
To start the node, we first need to build the source. You can build via Makefile by running the following command:


make go-quai

Start
Now that we’ve built the source and set out our variable flags, we can combine them to start the node.

To start your node, run the start command with the added genesis-nonce,quai-coinbases, qi-coinbases, coinbase-lockup, and node.slices flags that we covered above.

The coinbase values below are set to dummy values. If you do not replace them with your own addresses, you will not recieve block rewards.


./build/bin/go-quai start \
--node.slices '[0 0]' \
--node.genesis-nonce 6224362036655375007 \
--node.quai-coinbases '0x0000000000000000000000000000000000000000' \
--node.qi-coinbases '0x0080000000000000000000000000000000000000' \
--node.miner-preference 0.5 \
--node.coinbase-lockup 0
This will spin up a node using the values of the node.slices, node.quai-coinbases, node.qi-coinbases, and node.coinbase-lockup flags in your command. Logs should begin printing to the terminal.


Stop
Stopping your node should be done any time you make changes to your config file or prior to shutting your machine down. A node instance can terminated using CTRL+C.

If you’re running a miner, CTRL+C may not work. You must kill the miner process prior to stopping the node.

Other Node Operations

Check log output


Checking sync progress


Update


Reset and clear


Download and Sync from Snapshot

Was this page helpful?


Yes

No
Suggest edits
Raise issue
Enable Node Monitoring
x
github
linkedin
Powered by Mintlify
Run A Node - Quai Network Docs
