# Near Prototol Stake Wars

# 1. Create your Shardnet wallet

## 1.1. ****Create a wallet****

[https://wallet.shardnet.near.org/](https://wallet.shardnet.near.org/)

![Untitled](/Untitled.png)

> P.S. when you create the account successful, it will give you some test-near
> 

## 1.2. Setup NEAR-CLI

NEAR-CLI is a command-line interface that communicates with the NEAR blockchain via remote procedure calls (RPC):

- Setup and Installation NEAR CLI
- View Validator Stats

> Note: For security reasons, it is recommended that NEAR-CLI be installed on a **different computer** than your validator node and that no full access keys be kept on your validator node.
> 

First, let's make sure the linux machine is up-to-date.

```bash
sudo apt update && sudo apt upgrade -y
```

### 1.2.1. **Install developer tools, Node.js, and npm**

First, we will start with installing `Node.js` and `npm`:

```bash
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
sudo apt install build-essential nodejs
PATH="$PATH"
```

Check `Node.js` and `npm` version:

```bash
node -v
```

> v18.x.x
> 

```bash
npm -v
```

> 8.x.x
> 

### 1.2.2. **Install NEAR-CLI**

Here's the Github Repository for NEAR CLI.: [https://github.com/near/near-cli](https://github.com/near/near-cli). To install NEAR-CLI, unless you are logged in as root, which is not recommended you will need to use `sudo` to install NEAR-CLI so that the near binary is saved to /usr/local/bin

```bash
sudo npm install -g near-cli
```

### 1.2.3. **Validator Stats**

Now that NEAR-CLI is installed, let's test out the CLI and use the following commands to interact with the blockchain as well as to view validator stats. There are three reports used to monitor validator status:

### 1.2.4. **Environment**

- The environment will need to be set each time a new shell is launched to select the correct network.
    - Networks:
    - GuildNet
    - TestNet
    - MainNet
    - **Shardnet** (this is the network we will use for Stake Wars)
- Command:

```bash
export NEAR_ENV=shardnet
```

- You can also run this command to set the Near testnet Environment persistent:

```bash
echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
```

### 1.2.5. **NEAR CLI Commands Guide:**

1. **Proposals**
    
    A proposal by a validator indicates they would like to enter the validator set, in order for a proposal to be accepted it must meet the minimum seat price.
    
    Command:
    
    ```bash
    near proposals
    ```
    
    ![Untitled](Untitled%201.png)
    
2. **Validators Current**
    
    This shows a list of active validators in the current epoch, the number of blocks produced, number of blocks expected, and online rate. Used to monitor if a validator is having issues.
    
    Command:
    
    ```bash
    near validators current
    ```
    
    ![Untitled](Untitled%202.png)
    
3. **Validators Next**
    
    This shows validators whose proposal was accepted one epoch ago, and that will enter the validator set in the next epoch.
    
    Command:
    
    ```bash
    near validators next
    ```
    
    ![Untitled](Untitled%203.png)
    

# 2. Setup a validator and sync it to the actual state of the network

### **Server Requirements**

Please see the hardware requirement below:

| Hardware | Chunk-Only Producer Specifications | Software |
| --- | --- | --- |
| CPU | 4-Core CPU with AVX support | ubuntu |
| RAM | 8GB DDR4 |  |
| Storage | 500GB SSD |  |

![Untitled](Untitled%204.png)

> Cost: AWS less than **120$** one month
> 

> Sometimes a newly allocated disk may need to be mounted, like⬇️
> 

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html)

```bash
lsblk 

#check file system on the device
sudo file -s /dev/XXXXXX

#if not
sudo mkfs -t xfs /dev/XXXXXX

#creates a directory
sudo mkdir /data

#mount the volume at the directory 
sudo mount /dev/XXXXXX /data

#usually we chmod 777 for /data/
cd /
chomd 777 /data/
```

### **Install required software & set the configuration**

### **Prerequisites:**

Before you start, you may want to confirm that your machine has the right CPU features.

```bash
lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  && echo "Supported" \
  || echo "Not supported"
```

> Supported
> 

### **Install developer tools:**

```bash
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
```

> if error ⬇️
> 

```bash
sudo apt-get update
```

### **Install Python pip:**

```bash
sudo apt install python3-pip
```

### **Set the configuration:**

```bash
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"
```

### **Install Building env**

```bash
sudo apt install clang build-essential make
```

### **Install Rust & Cargo**

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

You will see the following:

![https://github.com/near/stakewars-iii/raw/main/challenges/images/rust.png](https://github.com/near/stakewars-iii/raw/main/challenges/images/rust.png)

Press 1 and press enter.

### **Source the environment**

```bash
source $HOME/.cargo/env
```

### **Clone `nearcore` project from GitHub**

First, clone the `[nearcore` repository](https://github.com/near/nearcore).

```bash
git clone https://github.com/near/nearcore
cd nearcore
git fetch
```

Checkout to the commit needed. Please refer to the commit defined in [this file](https://github.com/near/stakewars-iii/blob/main/commit.md).

```bash
git checkout <commit>
```

> git checkout 0f81dca95a55f975b6e54fe6f311a71792e21698
> 

### **Compile `nearcore` binary**

In the `nearcore` folder run the following commands:

```bash
cargo build -p neard --release --features shardnet
```

The binary path is `target/release/neard`. If you are seeing issues, it is possible that cargo command is not found. Compiling `nearcore` binary may take a little while.

### **Initialize working directory**

In order to work properly, the NEAR node requires a working directory and a couple of configuration files. Generate the initial required working directory by running:

```bash
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
```

![https://github.com/near/stakewars-iii/raw/main/challenges/images/initialize.png](https://github.com/near/stakewars-iii/raw/main/challenges/images/initialize.png)

This command will create the directory structure and will generate `config.json`, `node_key.json`, and `genesis.json` on the network you have passed.

- `config.json` - Configuration parameters which are responsive for how the node will work. The config.json contains needed information for a node to run on the network, how to communicate with peers, and how to reach consensus. Although some options are configurable. In general validators have opted to use the default config.json provided.
- `genesis.json` - A file with all the data the network started with at genesis. This contains initial accounts, contracts, access keys, and other records which represents the initial state of the blockchain. The genesis.json file is a snapshot of the network state at a point in time. In contacts accounts, balances, active validators, and other information about the network.
- `node_key.json` - A file which contains a public and private key for the node. Also includes an optional `account_id` parameter which is required to run a validator node (not covered in this doc).
- `data/` - A folder in which a NEAR node will write it's state.

### **Replace the `config.json`**

From the generated `config.json`, there two parameters to modify:

- `boot_nodes`: If you had not specify the boot nodes to use during init in Step 3, the generated `config.json` shows an empty array, so we will need to replace it with a full one specifying the boot nodes.
- `tracked_shards`: In the generated `config.json`, this field is an empty. You will have to replace it to `"tracked_shards": [0]`

```bash
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```

### **Get latest snapshot**

**IMPORTANT: NOT REQUIRED TO GET SNAPSHOT AFTER HARDFORK ON SHARDNET DURING 2022-07-18**

Install AWS Cli

```bash
sudo apt-get install awscli -y
```

Download snapshot (genesis.json)

```bash
// IMPORTANT: NOT REQUIRED TO GET SNAPSHOT AFTER HARDFORK ON SHARDNET DURING 2022-07-18
cd ~/.near
wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json
```

If the above fails, AWS CLI may be oudated in your distribution repository. Instead, try:

```bash
pip3 install awscli --upgrade
```

### **Run the node**

To start your node simply run the following command:

```bash
cd ~/nearcore
./target/release/neard --home ~/.near run
```

The node is now running you can see log outputs in your console. Your node should be find peers, download headers to 100%, and then download blocks.

![https://github.com/near/stakewars-iii/raw/main/challenges/images/download.png](https://github.com/near/stakewars-iii/raw/main/challenges/images/download.png)

---

### **Activating the node as validator**

### **Authorize Wallet Locally**

A full access key needs to be installed locally to be able to sign transactions via NEAR-CLI.

- You need to run this command:

```bash
near login
```

> Note: This command launches a web browser allowing for the authorization of a full access key to be copied locally.
> 

1 – Copy the link in your browser

![Untitled](Untitled%205.png)

2 – Grant Access to Near CLI

![https://github.com/near/stakewars-iii/raw/main/challenges/images/3.png](https://github.com/near/stakewars-iii/raw/main/challenges/images/3.png)

3 – After Grant, you will see a page like this, go back to console

![https://github.com/near/stakewars-iii/raw/main/challenges/images/4.png](https://github.com/near/stakewars-iii/raw/main/challenges/images/4.png)

4 – Enter your wallet and press Enter

![https://github.com/near/stakewars-iii/raw/main/challenges/images/5.png](https://github.com/near/stakewars-iii/raw/main/challenges/images/5.png)

### **Check the validator_key.json**

- Run the following command:

```bash
cat ~/.near/validator_key.json
```

> Note: If a validator_key.json is not present, follow these steps to create one⬇️
> 

Create a `validator_key.json`

- Generate the Key file:

```bash
near generate-key <pool_id>
```

<pool_id> ---> xx.factory.shardnet.near WHERE xx is you pool name

- Copy the file generated to shardnet folder: Make sure to replace <pool_id> by your accountId

```bash
cp ~/.near-credentials/shardnet/YOUR_WALLET.json ~/.near/validator_key.json
```

<aside>
⚠️ P.S. when you install near-cli **on onther machine**, you should `cat YOUR_WALLET.json`, copy it and `vi validator_key.json` on ~/.near

</aside>

![Untitled](Untitled%206.png)

- Edit “account_id” => xx.factory.shardnet.near, where xx is your PoolName
- Change `private_key` to `secret_key`

> Note: The account_id must match the staking pool contract name or you will not be able to sign blocks.\
> 

File content must be in the following pattern:

```bash
{
  "account_id": "xx.factory.shardnet.near",
  "public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
  "secret_key": "ed25519:****"
}
```

### **Start the validator node**

```bash
target/release/neard run
```

- Setup Systemd Command:

```bash
sudo vi /etc/systemd/system/neard.service
```

Paste:

```bash
[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=<USER>
#Group=near
WorkingDirectory=/home/<USER>/.near
ExecStart=/home/<USER>/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

<aside>
⚠️ P.S. when you clone `nearcore` on `/data`  ExecStart=/data/nearcore/target/release/neard run

</aside>

> Note: Change USER to your paths
> 

Command:

```bash
sudo systemctl enable neard
```

Command:

```bash
sudo systemctl start neard
```

If you need to make a change to service because of an error in the file. It has to be reloaded:

```bash
sudo systemctl reload neard
```

### **Watch logs**

Command:

```bash
journalctl -n 100 -f -u neard
```

Make log output in pretty print

Command:

```bash
sudo apt install ccze
```

View Logs with color

Command:

```bash
journalctl -n 100 -f -u neard | ccze -A
```

![Untitled](Untitled%207.png)

### **Becoming a Validator**

In order to become a validator and enter the validator set, a minimum set of success criteria must be met.

- The node must be fully synced
- The `validator_key.json` must be in place
- The contract must be initialized with the public_key in `validator_key.json`
- The account_id must be set to the staking pool contract id
- There must be enough delegations to meet the minimum seat price. See the seat price [here](https://explorer.shardnet.near.org/nodes/validators).
- A proposal must be submitted by pinging the contract
- Once a proposal is accepted a validator must wait 2-3 epoch to enter the validator set
- Once in the validator set the validator must produce great than 90% of assigned blocks

Check running status of validator node. If “Validator” is showing up, your pool is selected in the current validators list.

# 3. Deploy a new staking pool for your validator

## **Mounting a staking pool**

NEAR uses a staking pool factory with a whitelisted staking contract to ensure delegators’ funds are safe. In order to run a validator on NEAR, a staking pool must be deployed to a NEAR account and integrated into a NEAR validator node. Delegators must use a UI or the command line to stake to the pool. A staking pool is a smart contract that is deployed to a NEAR account.

### **Deploy a Staking Pool Contract**

### **Deploy a Staking Pool**

Calls the staking pool factory, creates a new staking pool with the specified name, and deploys it to the indicated accountId.

```bash
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool id>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000
```

From the example above, you need to replace:

- **Pool ID**: Staking pool name, the factory automatically adds its name to this parameter, creating {pool_id}.{staking_pool_factory} Examples:
- If pool id is stakewars will create : `stakewars.factory.shardnet.near`
- **Owner ID**: The SHARDNET account (i.e. stakewares.shardnet.near) that will manage the staking pool.
- **Public Key**: The public key in your **validator_key.json** file.
- **5**: The fee the pool will charge (e.g. in this case 5 over 100 is 5% of fees).
- **Account Id**: The SHARDNET account deploying and signing the mount tx. Usually the same as the Owner ID.

> Be sure to have at least 30 NEAR available, it is the minimum required for storage. Example : near call stake_wars_validator.factory.shardnet.near --amount 30 --accountId stakewars.shardnet.near --gas=300000000000000
> 

To change the pool parameters, such as changing the amount of commission charged to 1% in the example below, use this command:

```bash
near call <pool_name> update_reward_fee_fraction '{"reward_fee_fraction": {"numerator": 1, "denominator": 100}}' --accountId <account_id> --gas=300000000000000
```

You will see something like this:

![https://github.com/near/stakewars-iii/raw/main/challenges/images/pool.png](https://github.com/near/stakewars-iii/raw/main/challenges/images/pool.png)

If there is a “True” at the End. Your pool is created.

**You have now configure your Staking pool.**

Check your pool is now visible on [https://explorer.shardnet.near.org/nodes/validators](https://explorer.shardnet.near.org/nodes/validators)

### **Transactions Guide**

### **Deposit and Stake NEAR**

Command:

```bash
near call <staking_pool_id> deposit_and_stake --amount <amount> --accountId <accountId> --gas=300000000000000
```

### **Unstake NEAR**

Amount in yoctoNEAR.

Run the following command to unstake:

```bash
near call <staking_pool_id> unstake '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```

To unstake all you can run this one:

```bash
near call <staking_pool_id> unstake_all --accountId <accountId> --gas=300000000000000
```

### **Withdraw**

Unstaking takes 2-3 epochs to complete, after that period you can withdraw in YoctoNEAR from pool.

Command:

```bash
near call <staking_pool_id> withdraw '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```

Command to withdraw all:

```bash
near call <staking_pool_id> withdraw_all --accountId <accountId> --gas=300000000000000
```

### **Ping**

A ping issues a new proposal and updates the staking balances for your delegators. A ping should be issued each epoch to keep reported rewards current.

Command:

```bash
near call <staking_pool_id> ping '{}' --accountId <accountId> --gas=300000000000000
```

Balances Total Balance Command:

```bash
near view <staking_pool_id> get_account_total_balance '{"account_id": "<accountId>"}'
```

### **Staked Balance**

Command:

```bash
near view <staking_pool_id> get_account_staked_balance '{"account_id": "<accountId>"}'
```

### **Unstaked Balance**

Command:

```bash
near view <staking_pool_id> get_account_unstaked_balance '{"account_id": "<accountId>"}'
```

### **Available for Withdrawal**

You can only withdraw funds from a contract if they are unlocked.

Command:

```bash
near view <staking_pool_id> is_account_unstaked_balance_available '{"account_id": "<accountId>"}'
```

### **Pause / Resume Staking**

### **Pause**

Command:

```bash
near call <staking_pool_id> pause_staking '{}' --accountId <accountId>
```

### **Resume**

Command:

```bash
near call <staking_pool_id> resume_staking '{}' --accountId <accountId>
```

# 4. Setup tools for monitoring node status

### **Monitor and make alerts**

An email notification can make it more comfortable to maintain a validator up and running. Achieve to be a validator confirming transactions on testnet and get >95% of uptime.

### **Log Files**

The log file is stored either in the ~/.nearup/logs directory or in systemd depending on your setup.

Systemd Command:

```bash
journalctl -n 100 -f -u neard | ccze -A
```

**Log file sample:**

Validator | 1 validator

```bash
INFO stats: #85079829 H1GUabkB7TW2K2yhZqZ7G47gnpS7ESqicDMNyb9EE6tf Validator 73 validators 30 peers ⬇ 506.1kiB/s ⬆ 428.3kiB/s 1.20 bps 62.08 Tgas/s CPU: 23%, Mem: 7.4 GiB
```

- **Validator**: A “Validator” will indicate you are an active validator
- **73 validators**: Total 73 validators on the network
- **30 peers**: You current have 30 peers. You need at least 3 peers to reach consensus and start validating
- **#46199418**: block – Look to ensure blocks are moving

### **RPC**

Any node within the network offers RPC services on port 3030 as long as the port is open in the nodes firewall. The NEAR-CLI uses RPC calls behind the scenes. Common uses for RPC are to check on validator stats, node version and to see delegator stake, although it can be used to interact with the blockchain, accounts and contracts overall.

Find many commands and how to use them in more detail here:

[https://docs.near.org/docs/api/rpc](https://docs.near.org/docs/api/rpc)

Command:

```bash
sudo apt install curl jq
```

### **Common Commands:**

####### Check your node version: Command:

```bash
curl -s http://127.0.0.1:3030/status | jq .version
```

####### Check Delegators and Stake Command:

```bash
near view <your pool>.factory.shardnet.near get_accounts '{"from_index": 0, "limit": 10}' --accountId <accountId>.shardnet.near
```

####### Check Reason Validator Kicked Command:

```bash
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.prev_epoch_kickout[] | select(.account_id | contains ("<POOL_ID>"))' | jq .reason
```

####### Check Blocks Produced / Expected Command:

```bash
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.current_validators[] | select(.account_id | contains ("POOL_ID"))'
```
