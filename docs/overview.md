# Overview

## [Concepts](concepts.md)

## Hardware

Using a serverless solution such as AWS Fargate would've been more scalable since the primary purpose of an archive node is to serve API requests. However, since a blockchain node is a DB and API server combined, it's impossible to use a serverless solution. This is because a serverless solution requires the DB to be scaled separately from the API server. Since the blockchain combines the two, the DB would get corrupted because the API servers would all be writing to the DB, and the DB isn't engineered to handle duplicate writes.

There are no benefits (no cost savings, etc.) to be gained from running anything other than a single blockchain node on a computer. Therefore, for simplicity, this repo assumes that the computer used to run the blockchain node will not run any other software (including other blockchain nodes).

Here's how to set up the hardware:

- We recommend using a Linux OS; Ubuntu specifically. Since this repo uses Ubuntu commands, you'll have to modify them if you're using a different OS.
- Use an SSD (preferably NVMe) for storage. Here are the storage size requirements for different servers:
  - Archive node: Look up the storage requirements for the blockchain network you're setting up an archive node for.
  - RPC node: 1.5 TiB
  - Sentry: 100 GiB
  - Monitor: 20 GiB
  - Cosigner: 20 GiB
- This point only applies to archive nodes since other nodes can quickly be restored via a snapshot or state sync.

  Since the DB can easily get corrupted, and syncing all over again can take days, it's highly recommended taking a daily backup of either the entire storage device (this way you'll have the correct version of the node installed for the backup's DB) or the `$DAEMON_HOME/data` directory (i.e., the DB). You should keep two backups because if you only keep one, and the backup takes place just after the DB gets corrupted, then the only backup will also be corrupted.

- The architecture must be x86_64.
- RAM:
  - Blockchain node: 32 GiB
  - Monitor: 8 GiB
  - Cosigner: 1 GiB
- CPU:
  - Blockchain node: Four 3.2 GHz CPU cores
  - Monitor: Two CPU cores
  - Cosigner: One CPU core

We recommend the following if you're using AWS:

- Use AWS EC2 for the computer with elastic IP addresses. Use the `t2.micro` instance type for cosigners, `t2.large` for monitors, and `m6a.2xlarge` for blockchain nodes.
- Use AWS EBS for storage.
- If you're running an archive node, and have synced the blocks yourself because there's no snapshot available, use AWS DLM to create a daily backup of the AWS EBS volume.

Provision the necessary servers before proceeding further.

If you're running a full node other than a sentry, then provision two servers for blockchain nodes (one as a backup), and one server for the monitor. If you're running a validator, then provision three servers for sentries, three servers for cosigners, one server for the monitor, and a server for a validator which is only required to set up Horcrux.

Remember to place the backup server such as the second RPC node's server in a different region that the primary server. For example, if you're using AWS, and the primary RPC node is in the `us-east-1a` availability zone, then place the backup RPC node in the `us-east-1b` availability zone (use availability zones within the same region so that AWS ELB can access both the RPC nodes).

Here are monthly cost estimates if you're using AWS without a savings plan:

- Monitor: 68 USD
- Cosigner: 8 USD
- Sentry: 157 USD
- RPC node: 297 USD
- RPC node setup (one monitor, and two RPC nodes): 620 USD
- Validator setup (one monitor, three sentries, and three cosigners): 521 USD

## Software

Here are common [operations](blockchain-node-operations.md) that you can refer to throughout the setup process as well as long after that for blockchain nodes.

Set up the software:

1. Enter a terminal multiplexer on every server in order to prevent losing progress during long commands in case your SSH connection gets disrupted:

   ```shell
   tmux
   ```

2. Follow this step on each server. Set up the [server](server-setup.md).
3. Follow this step on each that's for a blockchain node. Set up the [blockchain node](blockchain-node-setup.md).
4. Follow this step on each cosigner, full node, and monitor. Set up the [URL](url-setup.md).
5. If you're setting up a validator, then set up the [cosigners](cosigner-setup.md).
6. If you had created a server for a validator, delete it now.
7. Follow this step on the monitor. Set up [observability](observability/observability.md).
8. Exit the terminal multiplexer on every server:

   ```shell
   exit
   ```
