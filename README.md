# Cosmos Node Setup

This repo explains how to set up [Cosmos](https://cosmos.network/) nodes. We created it because there are zero comprehensive articles, videos, etc. on doing this, and the official docs on how to just set up a node is confusing.

This repo is written for people who know nothing about running a Cosmos node. While concepts specific to operating Cosmos nodes (e.g., which AWS services to use, how to connect the node to Prometheus) are explained, concepts specific to Cosmos (e.g., what is a blockchain), AWS (e.g., what is AWS EBS), etc. aren't explained. This is because the official docs are good, and you can look up only what you need to based on your experience level.

## Concepts

You will need to understand these [concepts](docs/concepts.md) in order to effectively run the node.

## Hardware

Using a serverless solution (e.g., AWS Fargate) would've been more scalable since the primary purpose of an archive node is to serve API requests. However, since a blockchain node is a DB and API server combined, it's impossible to use a serverless solution. This is because a serverless solution requires the DB to be scaled separately from the API server. Since the blockchain combines the two, the DB would get corrupted because the API servers would all be writing to the DB, and the DB isn't engineered to handle duplicate writes.

Here's how to set up the hardware:
- We recommend using a Linux OS; Ubuntu specifically.
- We recommend using an SSD (preferably an NVMe). For Juno mainnet archive nodes, and Sei testnet validator nodes, use 1 TiB of storage.
- Since the DB can easily get corrupted, and syncing all over again can take days, it's highly recommended taking a daily backup of either the entire storage device (this way you'll have the correct version of the node installed for the backup's DB) or the `<CHAIN_DIR>/data` directory (i.e., the DB). You should keep two backups because if you only keep one, and the backup takes place just after the DB gets corrupted, then the only backup will also be corrupted.
- The architecture must be 64-bit (x86).
- 32 GiB of RAM.
- 4 CPU cores.
- Firewall:
    - If you require SSH access to your server, allow SSH connections over TCP on port 22 for your IP address.
    - If you require that clients be allowed to make API calls, allow HTTP connections on port 80 from any IP address, and HTTPS connections on port 443 from any IP address.

If you're using AWS, we recommend using AWS EC2 with AWS EBS. Use AWS Data Lifecycle Manager to create a daily backup.

## [Node Setup](docs/node-setup.md)

## [Operating the Node](docs/operating.md)

## License

This project is under the [MIT License](LICENSE).
