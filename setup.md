# Setup

This document explains how to set up a [Juno](https://www.junonetwork.io/) mainnet archive node on AWS. I created it because there are zero articles, videos, etc. on doing this, and even the official docs on how to just set up the Juno node are incorrect.

Every section in this document can be used independently of the others. For example, you can swap out the AWS setup in favor of your own hardware configuration, and the rest of the document will still apply verbatim.

## To-dos

This document is a WIP, and must be updated to:

- Optimize AWS costs via savings plans.
- Use Fish instead of the default shell.
- Doc the AWS DLM setup.
- Use Terraform.
- Set up rate limiting.
- Set up the DNS.
- Set up Prometheus.
- Set up Grafana.
- Set up PANIC.
- Set up logs.
- Skipping the upgradation process by simply downloading the archive. Maybe Docker can be used to more easily set up the server then.
- Only expose the Prometheus server to the Grafana one in the firewall.
- See if you need to set the halt height in the service file. Also, maybe "ExecStart" can be shortened from `/home/ubuntu/go/bin/junod` to `junod`. Also, maybe we can set `Restart=always` in the service file always.
- Document enabling APIs.
- Ensure that the "###Upgrading" section's point 8. iii. has the correct halt height (maybe it's supposed to be -1 by default, or something like that). Also, check if after step 9, something like `sudo systemctl restart junod` must be run.

## Concepts

### Why AWS

People say that bare metal is multiple times cheaper than public cloud. This is true if your time is free. Bare metal servers require manually setting up firewalls, learning and using tools such as Ansible, etc. which take exponentially more time to learn, use, and secure than public clouds. In the long run, an experienced sysadmin could potentially save money using bare metal servers because they'll only ever run a single specialized piece of software (i.e., a Juno node) but otherwise, it's recommended to use a public cloud.

Regarding the choice of public cloud, GCP is obviously better than AWS but this document uses AWS because the author had to set it up on that due to external factors.

### Juno Versions

Due to an exploit, the Juno chain hard forked into Juno Phoenix (v3.0.0), and the original chain is named Juno Classic. This document is for Juno Phoenix which is the current mainnet.

### Archive Nodes

Validator nodes can be run by simply installing the latest version of the node, and downloading a snapshot. A snapshot is a fraction of the blockchain's complete state (e.g., a snapshot could just be the latest 300 MiB of a blockchain even if the entire blockchain is 4.3 TiB). However, archive nodes must go through a significantly more complicated process before they can be used because they need to download the entire blockchain.

In order to set up an archive node, the following steps must be taken:

1. Install the first version of the node. For example, if the first version is 1, and the latest version is 10, then you'll have to download v1.
2. Configure the node to stop downloading blocks at the block height the next upgrade took place at. For example, if v1 starts at block 1, and v2 starts at block 100, then the node must be configured to stop downloading blocks once it reaches block 100 if the node is currently on v1. Otherwise, the current version will be unable to process blocks produced by newer node versions causing the database to get corrupted.
3. Once the node stops syncing, replace the current node version with the next version, and resume syncing. For example, if the node is on v1, and the next version is v1.1, then v1 must be uninstalled, and v1.1 must be installed.
4. After installing the new version, set the next block height to stop syncing at just as before, and upgrade the software just as before. Repeat this process until the node has finished downloading all the older blocks.
5. Once the node is on the latest block, you'll have to perform the same steps that validator nodes do for upgrades which is the same as the upgradation process you've previously followed. In essence, once you've been notified that there's going to be an upgrade (e.g., over a Discord channel) taking place (e.g., in 3 days at block 200), you'll have to configure the node to stop syncing at that block height, and upgrade the software.

### Cosmovisor

Cosmovisor is a tool that aims to ease upgrading nodes. Theoretically, it's supposed to automatically download upgrades in advance, and automatically reinstall and restart the node at the correct block height. In practice, it doesn't work as advertised. Here's why we recommend not using Cosmovisor:

- You need to spend time installing it.
- You need to spend time learning it (different versions have different APIs).
- You need to spend time updating it, and the updates are buggy (e.g., v1.1.0 can't be installed directly).
- There are two types of upgrades - planned and unplanned. Planned upgrades are voted on via governance proposals, and add enhancements. Unplanned upgrades are emergency security fixes that aren't voted on via governance proposals, and don't add any enhancements. Unplanned upgrades make up about half of the upgrades, and aren't supported by Cosmovisor which means that you'll still need to remember how to manually upgrade the node. The problem with manually upgrading the node while using Cosmovisor is that you'll need to perform additional steps such as copying the executable to the Cosmovisor directory. If you don't copy the executable correctly, then the node wouldn't have properly upgraded which will cause the DB to get corrupted.
- If you don't correctly perform an upgrade, then the DB will get corrupted. If the DB gets corrupted, you'll have to restore a backup of the node so that you don't need to start from scratch, and you'll have to downgrade the node to the version to one that the backup supports. If you're using Cosmovisor, then the DB can get corrupted again after restoring the backup, and downgrading the node. The reason is because Cosmovisor will automatically upgrade the node to a newer version unless you remember to run some commands such as deleting the upgrades Cosmovisor kept copies of. Also, when downgrading the node, the Cosmovisor directory to copy the executable to changes in an undocumented manner.
- Due to a bug, you must spend time configuring Cosmovisor to not automatically download upgrades.
- You'll still want to monitor the upgrade as it happens with Cosmovisor in case the upgrade fails which happens a noticeable number of times.

## AWS Configuration

This section explains how to set up the hardware (AWS EC2 and AWS EBS) upon which the node will run by using the AWS Management Console.

Using a serverless solution such as AWS Fargate would've been more scalable since the primary purpose of an archive node is to serve API requests. However, since a blockchain node is a DB and API server combined, it's impossible to use a serverless solution. This is because a serverless solution requires the DB to be scaled separately from the API server. Since the blockchain combines the two, the DB would get corrupted because the API servers would all be writing to the DB, and the DB isn't engineered to handle duplicate writes.

### Name

We recommend something similar to _Juno Mainnet Archive Node_.

### AMI

- Any OS is fine; Ubuntu is the best.
- The storage device must be fast; an SSD is recommended.
- The architecture must be 64-bit (x86).

We recommend using the AMI with the ID `ami-052efd3df9dad4825`.

### Instance Type

For a validator, the recommended RAM is 32 GB, and the recommended number of CPU cores is 4. Since archive nodes don't need more than half of that, we'll use the `t3a.xlarge` instance type because it's the cheapest options that fits our needs.

### Key Pair

It's up to you whether you use a key pair, and which specifications the key pair uses. We recommend you use a key pair, and with the following specifications:

- **Key pair name**: We recommend something similar to _juno-mainnet-archive-node_.
- **Key pair type**: RSA
- **Private key file format**: `.pem`

### Network Settings

- **VPC**: Unless you have reasons to, use the default VPC.
- **Subnet**: Unless you have reasons to, set the **Subnet** to **No preference**.
- **Auto-assign public IP**: **Enable**
- **Firewall (security groups)**:
    - **Security group name**: We recommend something similar to _juno-mainnet-archive-node_.
    - **Description**: We recommend something similar to _Juno mainnet archive node on AWS EC2_.
    - **Inbound security groups rules** (We recommend adding the following security groups but none of them are mandatory.):
        - **Security group rule 1**:
            - **Type**: **SSH**
            - **Source type**: **My IP**
            - **Description**: We recommend something similar to _SSH for admin desktop_.
        - **Security group rule 2**:
            - **Type**: **Custom TCP**
            - **Port range**: 1317
            - **Source type**: **Anywhere**
            - **Description**: We recommend something similar to _Allows the frontends to make REST API calls_.
        - **Security group rule 3**:
            - **Type**: **Custom TCP**
            - **Port range**: 9090
            - **Source type**: **Anywhere**
            - **Description**: We recommend something similar to _Allows the frontends to make gRPC calls_.
        - **Security group rule 4**:
            - **Type**: **Custom TCP**
            - **Port range**: 9091
            - **Source type**: **Anywhere**
            - **Description**: We recommend something similar to _Allows the frontends to make gRPC Web calls_.
        - **Security group rule 5**:
            - **Type**: **Custom TCP**
            - **Port range**: 26657
            - **Source type**: **Anywhere**
            - **Description**: We recommend something similar to _Allows the frontends to make Tendermint RPC calls_.
        - **Security group rule 6**:
            - **Type**: **Custom TCP**
            - **Port range**: 26660
            - **Source type**: **Anywhere**
            - **Description**: We recommend something similar to _Prometheus_.

### Configure storage

- The blockchain is approximately 1 TiB in size.
- We recommend using a `gp3` root volume because the others are either too slow (e.g., `gp2`) or expensive (e.g., `io2` is faster but not worth the money).

Therefore, the configuration should be `1x 1000 GiB gp3 Root volume`.

### Number of instances

1

## Node Setup

This section explains how to set up the node software. Since the commands are for Ubuntu, you'll have to modify them if you're using a different OS.

1. Set up the dependencies:
    1. Update the package list:

        ```sh
        sudo apt-get update
        ```
    2. Install any available updates:

        ```sh
        sudo apt upgrade -y
        ```
    3. Install the toolchain, and ensure accurate time synchronization:

        ```sh
        sudo apt-get install make build-essential gcc git jq chrony -y
        ```
    4. Download a version of Go that's greater than or equal to 1.18, and less than 2:

        ```sh
        wget https://golang.org/dl/go1.18.2.linux-amd64.tar.gz
        ```
    5. Install Go:

        ```sh
        sudo tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz
        ```
    6. Delete the Go download:

        ```sh
        rm go1.18.2.linux-amd64.tar.gz
        ```
    7. Configure Go:

        ```sh
        tee -a ~/.profile << EOF
        export GOROOT=/usr/local/go
        export GOPATH=$HOME/go
        export GO111MODULE=on
        export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
        EOF
        ```
    8. Refresh environment variables:

        ```sh
        source ~/.profile
        ```
2. Install Juno:
    1. Clone the repo:

        ```sh
        git clone https://github.com/CosmosContracts/juno
        ```
    2. Change the directory:

        ```sh
        cd juno
        ```
    3. Fetch the tags:

        ```sh
        git fetch
        ```
    4. Check out the first version's tag:

        ```sh
        git checkout v3.0.0
        ```
    5. Install:

        ```sh
        make install
        ```
    6. If the installation succeeded, then the following command will print `v3.0.0`:

        ```sh
        junod version
        ```
3. Configure Juno:
    1. Create a variable for the repo:

        ```sh
        CHAIN_REPO="https://raw.githubusercontent.com/CosmosContracts/mainnet/main/juno-1"
        ```
    2. Create a variable for the peers:

        ```sh
        PEERS="$(curl -sL "$CHAIN_REPO/persistent_peers.txt")"
        ```
    3. Initialize the chain:

        ```sh
        junod init <MONIKER> --chain-id juno-1
        ```

       Replace `<MONIKER>` with the node's moniker (e.g., `node`).
    4. Download the genesis file:

        ```sh
        curl https://share.blockpane.com/juno/phoenix/genesis.json > ~/.juno/config/genesis.json
        ```
    5. Set the peers:

        ```sh
        sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.juno/config/config.toml
        ```
    6. In `~/.juno/config/app.toml`, set the value of the `halt-height` key to `2616300`.
4. Set up the Juno systemd unit:
    1. Create the unit (replace `ubuntu` with your user):

        ```sh
        sudo tee -a /etc/systemd/system/junod.service << EOF
        [Unit]
        Description=Juno Daemon
        After=network-online.target

        [Service]
        User=ubuntu
        ExecStart=/home/ubuntu/go/bin/junod start --x-crisis-skip-assert-invariants --halt-height 2616300
        RestartSec=3
        Restart=no
        LimitNOFILE=4096

        [Install]
        WantedBy=multi-user.target
        EOF
        ```

        - We use the `--x-crisis-skip-assert-invariants` flag to skip a check that takes up to several hours to complete. It's recommended to skip it, and it doesn't affect archive nodes anyway.
        - We use the `--halt-height` flag to schedule the node to halt once it needs to be upgraded.
        - We set the `Restart` directive to `no` because we don't want the node to automatically restart after it halts. Otherwise the halt would've been useless because the node will resume downloading blocks thereby corrupting the DB.
    2. Reload systemd:

        ```sh
        sudo systemctl daemon-reload
        ```
    3. Enable the unit:

        ```sh
        sudo systemctl enable junod
        ```
    4. Start the unit:

        ```sh
        sudo systemctl start junod
        ```
    5. Verify that it's running:

        ```sh
        sudo systemctl status junod
        ```

## Running the Node

This section explains the ongoing operations to perform on the node.

### Sync Status

```sh
curl -s http://localhost:26657/status | jq .result.sync_info.catching_up
```

Prints `true` if the node is still downloading the older blocks, and `false` otherwise.

### Logs

```sh
journalctl -fu junod
```

Follows the journal for the `junod` systemd unit. This is useful to check whether the chain has halted, which block it's currently downloading, any error message that got printed if the node crashed, etc.

### systemd Unit

The value of the `Restart` directive in the `/etc/systemd/system/junod.service` file must be either `no` or `always`. If the `--halt-height` flag has been set when running `junod`, then the value must be `no` because otherwise the node will automatically restart thereby corrupting the database by prcoessing blocks it wasn't programmed to. Otherwise, the value must be `always`.

### Genesis Error

If the node keeps crashing with the following error message:

```
Error: error during handshake: error on replay: validator set is nil in genesis and still empty after InitChain
```

then you must restore the node via a backup. If you don't have a backup, then you'll have to run the command `junod unsafe-reset-all` which will wipe the DB causing you to have to restart the entire syncing process.

### Peers

Blocks are synced via peers. If every peer the node is connected to becomes unreachable, then you'll have to update the peer list in the following manner:

1. Create a variable for the repo:

    ```sh
    CHAIN_REPO="https://raw.githubusercontent.com/CosmosContracts/mainnet/main/juno-1"
    ```
2. Create a variable for the peers:

    ```sh
    PEERS="$(curl -sL "$CHAIN_REPO/persistent_peers.txt")"
    ```
3. Update the peers:

    ```sh
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.juno/config/config.toml
    ```

### Available Upgrades

After installing the first version (i.e., 3.0.0), the following versions must be installed at their respective block heights (certain versions such as 7.0.0 must be skipped due to bugs, and are therefore left out):

|Version|Block height|
|:---:|:---:|
|3.1.1|2,616,300|
|4.0.0|2,951,100|
|5.0.1|3,035,000|
|6.0.0|3,159,650|
|8.0.0|3,851,750|

Since this document may be outdated, please additionally check [this](https://docs.junonetwork.io/validators/mainnet-upgrades) page for versions and block heights. Only check the version tags (e.g., `v8.0.0`), and their respective block heights on the page because the upgradation instructions are confusing at best. Instead, use the [upgradation](#upgrading) section of this document.

In order to get notified of new upgrades, you should join Juno's [#⚡ | validator-lounge](https://discord.com/channels/816256689078403103/816263136491339867) Discord channel where they try to announce new upgrades a few days in advance.

### Upgrading

Here's how to upgrade the node:

1. Wait for the chain to halt. For example, the block height to install v3.1.1 at is 2,616,300.
2. Change the directory to where Juno was downloaded:

    ```sh
    cd ~/juno
    ```
3. Fetch the tags:

    ```sh
    git fetch
    ```
4. Check out the upgrade's tag:

    ```sh
    git checkout <VERSION>
    ```

   Replace `<VERSION>` with the upgrade's tag (e.g., `v3.1.1`).
5. Install:

    ```sh
    make install
    ```
6. If the installation succeeded, then the following command will print the `<VERSION>` from step 4:

    ```sh
    junod version
    ```
7. If there's another upgrade to perform after this one, then follow these steps:
    1. In `/etc/systemd/system/junod.service`, set the value of the `--halt-height` flag to the next upgrade's target block height. For example, the value of the `ExecStart` directive should look similar to the following if you just upgraded to v3.1.1, and need to prepare for the v4.0.0 upgrade:

        ```service
        ExecStart=/home/ubuntu/go/bin/junod start --x-crisis-skip-assert-invariants --halt-height 2951100
        ```
    2. In `/etc/systemd/system/junod.service`, set the value of the `Restart` directive to `no`.
    3. In `~/.juno/config/app.toml`, set the value of the `halt-height` key to the same block height you had set in step 1.
8. If there isn't another upgrade to perform after this one, then follow these steps:
    1. In `/etc/systemd/system/junod.service`, remove the `--halt-height` flag. For example, the value of the `ExecStart` directive should look similar to the following:

        ```service
        ExecStart=/home/ubuntu/go/bin/junod start --x-crisis-skip-assert-invariants
        ```
    2. In `/etc/systemd/system/junod.service`, set the value of the `Restart` directive to `always`.
    3. In `~/.juno/config/app.toml`, set the value of the `halt-height` key to `0`.
9. Reload systemd:

    ```sh
    sudo systemctl daemon-reload
    ```
