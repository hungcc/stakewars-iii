# How to run Validator Node for Shardnet (Stake Wars: Episode III step-by-step instructions) on Google Cloud (GCP)

## 1. Create Shardnet wallet, setup Google Cloud instance & deploy NEAR CLI

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

