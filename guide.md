# How to run Validator Node for Shardnet on Google Cloud Platform (Stake Wars: Episode III step-by-step instructions)

## 1. Create Shardnet wallet, setup Google Cloud instance & deploy NEAR CLI (Challenge 001)

Create your Shardnet wallet & deploy the NEAR CLI. This is designed to be your very first challenge: use it to understand how staking on NEAR works.

### 1.1 Create a wallet
Follow the instructions on the website to create a NEAR wallet on the Shardnet testnet.

Link: https://wallet.shardnet.near.org/

![2022-07-17_22-22](https://user-images.githubusercontent.com/46512075/179482781-e264a375-e0dd-41e7-903b-1c04207ecb7f.png)

Successful wallet creation, you will receive ~2000 NEAR to the wallet (this Testnet tokens have no value on Mainnet)

![2022-07-17_22-28](https://user-images.githubusercontent.com/46512075/179482825-5f6954f0-60b1-468b-9b8f-6fa6c2cc8731.png)

### 1.2 Google Cloud Setup

Google Cloud offers a 90 day $300 free trial for every new user. These $300 are given as credits to your account and you can use them to get a sense of Google Cloud products. Be aware that you will need to add payment information when signing up for the free trial. This is for identity verification purposes and will not incur charges until you upgrade to a paid account and run out of credits.

The cost depends on how you configure it, factors like location, machine configuration, disk size... determine the final cost.

Recommended configuration is 4-Core CPU with AVX support, 8GB RAM, 500GB SSD Storage. But we can change the configuration later without affecting the data, so you can start with a moderate configuration and upgrade further if needed.

I will start with configuration like below:

![2022-07-18_18-14](https://user-images.githubusercontent.com/46512075/179500179-5e03bced-c2a5-41fc-8dd6-d122e3b53089.png)

![2022-07-18_18-15](https://user-images.githubusercontent.com/46512075/179500201-376a279d-7ebf-4fbb-abbe-bcb3195978b9.png)

You can refer to the official Google Cloud documentation here: https://cloud.google.com/compute/docs/instances/create-start-instance

### 1.3 Deploy NEAR-CLI

NEAR-CLI is a command-line interface that communicates with the NEAR blockchain via remote procedure calls (RPC):

* Setup and Installation NEAR CLI
* View Validator Stats

> Note: For security reasons, it is recommended that NEAR-CLI be installed on a different computer than your validator node and that no full access keys be kept on your validator node.

First, let's make sure the linux machine is up-to-date.
```
sudo apt update && sudo apt upgrade -y
```

##### Install developer tools, Node.js, and npm
First, we will start with installing `Node.js` and `npm`:
```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
sudo apt install build-essential nodejs
PATH="$PATH"
```

Check `Node.js` and `npm` version:
```
node -v
```
> v18.x.x

```
npm -v
```
> 8.x.x

![2022-07-17_23-14](https://user-images.githubusercontent.com/46512075/179504952-7d462c7c-75f8-41e0-a04c-0418d46afb61.png)


##### Install NEAR-CLI
Here's the Github Repository for NEAR CLI.: https://github.com/near/near-cli. To install NEAR-CLI, unless you are logged in as root, which is not recommended you will need to use `sudo` to install NEAR-CLI so that the near binary is saved to /usr/local/bin

```
sudo npm install -g near-cli
```

![2022-07-17_23-19](https://user-images.githubusercontent.com/46512075/179505616-e8431edf-cea4-4a62-9ccf-31ae89a30e9c.png)

### Validator Stats

Now that NEAR-CLI is installed, let's test out the CLI and use the following commands to interact with the blockchain as well as to view validator stats. There are three reports used to monitor validator status:


###### Environment
The environment will need to be set each time a new shell is launched to select the correct network.

Networks:
- GuildNet
- TestNet
- MainNet
- **Shardnet** (this is the network we will use for Stake Wars)

Command:
```
export NEAR_ENV=shardnet
```

You can also run this command to set the Near testnet Environment persistent:
```
echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
```
![2022-07-17_23-21](https://user-images.githubusercontent.com/46512075/179505977-1b80d346-d3de-4bd8-8786-ede5cfbbbfe6.png)


#### NEAR CLI Commands Guide:

###### Proposals
A proposal by a validator indicates they would like to enter the validator set, in order for a proposal to be accepted it must meet the minimum seat price.

Command:
```
near proposals
```
![2022-07-17_23-28](https://user-images.githubusercontent.com/46512075/179506209-94b050ef-8728-4dfc-9741-fe3b33f39827.png)


###### Validators Current
This shows a list of active validators in the current epoch, the number of blocks produced, number of blocks expected, and online rate. Used to monitor if a validator is having issues.

Command:
```
near validators current
```
![2022-07-17_23-27](https://user-images.githubusercontent.com/46512075/179506272-89f629e0-b6f5-4159-b46e-5eb75c362f0b.png)


###### Validators Next
This shows validators whose proposal was accepted one epoch ago, and that will enter the validator set in the next epoch.

Command:
```
near validators next
```
![2022-07-17_23-27_1](https://user-images.githubusercontent.com/46512075/179506309-086d99e8-0d19-4784-90ac-e8b18b8ce67e.png)

## 2. Setup a validator and sync it to the actual state of the network (Challenge 002)

This challenge is focused on deploying a node (nearcore), downloading a snapshot, syncing it to the actual state of the network, then activating the node as a validator. 

### 2.1 Setup your node

#### Install required software & set the configuration

##### Prerequisites:

Before you start, you may want to confirm that your machine has the right CPU features. 

```
lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  && echo "Supported" \
  || echo "Not supported"
```
> Supported

##### Install developer tools:
```
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
```
#####  Install Python pip:

```
sudo apt install python3-pip
```
##### Set the configuration:

```
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"
```

##### Install Building env
```
sudo apt install clang build-essential make
```

##### Install Rust & Cargo
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

You will see the following:

![2022-07-17_23-46](https://user-images.githubusercontent.com/46512075/179707109-4dd3034f-d5e0-4b72-bf00-a25b80e0472b.png)


Press 1 and press enter.

##### Source the environment
```
source $HOME/.cargo/env
```

#### Clone `nearcore` project from GitHub
First, clone the [`nearcore` repository](https://github.com/near/nearcore).

```
git clone https://github.com/near/nearcore
cd nearcore
git fetch
```

Checkout to the commit needed. Please refer to the commit defined here https://github.com/near/stakewars-iii/blob/main/challenges/commit.md
```
git checkout <commit>
```

#### Compile `nearcore` binary
In the `nearcore` folder run the following commands:

```
cargo build -p neard --release --features shardnet
```
The binary path is `target/release/neard`. If you are seeing issues, it is possible that cargo command is not found. Compiling `nearcore` binary may take a little while.

![image](https://user-images.githubusercontent.com/46512075/179708891-e1c1f755-c6f2-47c6-a5b6-854994a6bce1.png)


#### Initialize working directory

In order to work properly, the NEAR node requires a working directory and a couple of configuration files. Generate the initial required working directory by running:

```
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
```
![image](https://user-images.githubusercontent.com/46512075/179710857-e57cebc4-6299-4051-bd71-0b20b2417e86.png)

This command will create the directory structure and will generate `config.json`, `node_key.json`, and `genesis.json` on the network you have passed. 

- `config.json` - Configuration parameters which are responsive for how the node will work. The config.json contains needed information for a node to run on the network, how to communicate with peers, and how to reach consensus. Although some options are configurable. In general validators have opted to use the default config.json provided.

- `genesis.json` - A file with all the data the network started with at genesis. This contains initial accounts, contracts, access keys, and other records which represents the initial state of the blockchain. The genesis.json file is a snapshot of the network state at a point in time. In contacts accounts, balances, active validators, and other information about the network. 

- `node_key.json` -  A file which contains a public and private key for the node. Also includes an optional `account_id` parameter which is required to run a validator node (not covered in this doc).

- `data/` -  A folder in which a NEAR node will write it's state.

#### Replace the `config.json`

From the generated `config.json`, there two parameters to modify:
- `boot_nodes`: If you had not specify the boot nodes to use during init in Step 3, the generated `config.json` shows an empty array, so we will need to replace it with a full one specifying the boot nodes.
- `tracked_shards`: In the generated `config.json`, this field is an empty empty. You will have to replace it to `"tracked_shards": [0]`

```
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```
![image](https://user-images.githubusercontent.com/46512075/179711127-6c700b77-094d-4f2f-a5b9-2bc7d84478ff.png)

####  Start the validator node

Setup Systemd

```
sudo tee /etc/systemd/system/neard.service > /dev/null <<EOF
[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=$USER
#Group=near
WorkingDirectory=$home/root/.near
ExecStart=/root/nearcore/target/release/neard 
run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed
[Install]
WantedBy=multi-user.target
EOF
```

Command:

```
sudo systemctl enable neard
```
Command:

```
sudo systemctl start neard
```
If you need to make a change to service because of an error in the file. It has to be reloaded:

```
sudo systemctl reload neard
```
Check logs:

```
journalctl -n 100 -f -u neard
```
![2022-07-18_00-24](https://user-images.githubusercontent.com/46512075/179716482-f1fb4cbf-72ef-46e8-b7a2-65828449db0f.png)


### 2.2 Activating the node as validator
##### Authorize Wallet Locally
A full access key needs to be installed locally to be able to sign transactions via NEAR-CLI.


* You need to run this command:

```
near login
```

> Note: This command launches a web browser allowing for the authorization of a full access key to be copied locally.

1 – Copy the link in your browser

2 – Grant Access to Near CLI

![2022-07-18_00-27](https://user-images.githubusercontent.com/46512075/179713044-d39562ca-c0a1-442c-9d5c-d380021894bc.png)

3 – After Grant, you will see a page like this, go back to console

![2022-07-18_00-29](https://user-images.githubusercontent.com/46512075/179713130-c54e9a00-3d33-4b10-ba43-9720eb3bae8c.png)

4 – Enter your wallet and press Enter

![2022-07-18_00-31](https://user-images.githubusercontent.com/46512075/179713309-c81bb200-8030-42e1-928c-cf8a411454d4.png)

#####  Create the validator_key.json
* Copy the file generated to shardnet folder:
Make sure to replace <pool_id> by your accountId
```
cp ~/.near-credentials/shardnet/YOUR_WALLET.json ~/.near/validator_key.json
```
* Edit “account_id” => xx.factory.shardnet.near, where xx is your PoolName
* Change `private_key` to `secret_key`

> Note: The account_id must match the staking pool contract name or you will not be able to sign blocks.\

File content must be in the following pattern:
```
{
  "account_id": "xx.factory.shardnet.near",
  "public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
  "secret_key": "ed25519:****"
}
```

#### Becoming a Validator
In order to become a validator and enter the validator set, a minimum set of success criteria must be met.

* The node must be fully synced
* The `validator_key.json` must be in place
* The contract must be initialized with the public_key in `validator_key.json`
* The account_id must be set to the staking pool contract id
* There must be enough delegations to meet the minimum seat price. See the seat price [here](https://explorer.shardnet.near.org/nodes/validators).
* A proposal must be submitted by pinging the contract
* Once a proposal is accepted a validator must wait 2-3 epoch to enter the validator set
* Once in the validator set the validator must produce great than 90% of assigned blocks

Check running status of validator node. If “Validator” is showing up, your pool is selected in the current validators list.
