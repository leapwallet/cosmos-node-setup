# Overview

## [Concepts](concepts.md)

## Hardware

Using a serverless solution (e.g., AWS Fargate) would've been more scalable since the primary purpose of an archive node is to serve API requests. However, since a blockchain node is a DB and API server combined, it's impossible to use a serverless solution. This is because a serverless solution requires the DB to be scaled separately from the API server. Since the blockchain combines the two, the DB would get corrupted because the API servers would all be writing to the DB, and the DB isn't engineered to handle duplicate writes.

There are no benefits (no cost savings, etc.) to be gained from running anything other than a single blockchain node on a computer. Therefore, for simplicity, this repo assumes that the computer used to run the blockchain node will not run any other software (including other blockchain nodes).

The same monitors can be used for every blockchain node that you set up. So, even though you'll set up separate servers for sentries, cosigners, full nodes, and/or validators, the same two servers used to monitor the first blockchain node can and should be used for subsequent blockchain nodes you set up.

Here's how to set up the hardware:
- We recommend using a Linux OS; Ubuntu specifically. Since this repo uses Ubuntu commands, you'll have to modify them if you're using a different OS.
- Use an SSD (preferably NVMe) for storage.

    Here are specific recommendations for storage requirements:
    - Juno mainnet archive node: 1 TiB
    - Non-archive blockchain node: 100 GiB
    - Cosigner/monitor: 20 GiB
- This point only applies to archive nodes since other nodes can quickly be restored via a snapshot or state sync. 

    Since the DB can easily get corrupted, and syncing all over again can take days, it's highly recommended taking a daily backup of either the entire storage device (this way you'll have the correct version of the node installed for the backup's DB) or the `$DAEMON_HOME/data` directory (i.e., the DB). You should keep two backups because if you only keep one, and the backup takes place just after the DB gets corrupted, then the only backup will also be corrupted.
- The architecture must be x86_64.
- 16 GiB of RAM for blockchain nodes, and 1 GiB for cosigners/monitors.
- Four 3.2 GHz CPU cores for blockchain nodes, and 1 CPU core for cosigners/monitors.

We recommend the following if you're using AWS:
- Use AWS EC2 for the computer. Use the `t2.micro` instance type for cosigners and monitors, and `t2.xlarge` for blockchain nodes.
- Use AWS EBS for storage.
- If you're running an archive node, and have synced the blocks yourself because there's no snapshot available, use AWS DLM to create a daily backup of the AWS EBS volume.

Provision the necessary servers before proceeding further. If you're running a full node, then provision two servers for blockchain nodes (one as a backup), and two servers for monitors (one as a backup). If you're running a validator, then provision three servers for sentries, three servers for cosigners, two servers for monitors (one as a backup), and a server for a validator which is only required to set up Horcrux.

## Software

Here are common [operations](blockchain-node-operations.md) that you can refer to throughout the setup process as well as long after that for blockchain nodes.

Set up the software:
1. Follow this step on each server. Set up the [server](server-setup.md).
2. Follow this step on each that's for a blockchain node. Set up the [blockchain node](blockchain-node-setup.md).
3. Follow this step on each cosigner, full node, and monitor. Set up the [URL](url-setup.md).
4. If you're setting up a validator, then set up the [cosigners](cosigner-setup.md).
5. If you had created a server for a validator, delete it now.
6. Foll this step on each monitor. Set up [observability](observability.md).
