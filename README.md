# Juno AWS Setup

This repo explains how to set up a [Juno](https://www.junonetwork.io/) mainnet archive node on AWS. I created it because there are zero articles, videos, etc. on doing this, and the official docs on how to just set up the Juno node are confusing. Every section in this document can be used independently of the others. For example, you can swap out the AWS setup in favor of your own hardware configuration, and the rest of the document will still apply verbatim.

This repo is written for people who know nothing about running a Juno node. While concepts specific to operating Juno nodes (e.g., which AWS services to use, how to connect Juno to Prometheus) are explained, concepts specific to Juno (e.g., what is a blockchain), AWS (e.g., what is AWS EBS), etc. aren't explained. This is because the official docs are good, and you can look up only what you need to based on your experience level.

## To-dos

This repo is a WIP, and must be updated by the author to include [these](docs/to-dos.md) instructions.

## Concepts

You will need to understand these [concepts](docs/concepts.md) in order to effectively run the node.

## Hardware

Using a serverless solution (e.g., AWS Fargate) would've been more scalable since the primary purpose of an archive node is to serve API requests. However, since a blockchain node is a DB and API server combined, it's impossible to use a serverless solution. This is because a serverless solution requires the DB to be scaled separately from the API server. Since the blockchain combines the two, the DB would get corrupted because the API servers would all be writing to the DB, and the DB isn't engineered to handle duplicate writes.

We recommend only running one node because they're expensive, and clients can fall back to public nodes (e.g., sites such as [cosmos.directory](https://cosmos.directory/juno) list freely available nodes) during periods of downtime.

Here's how to set up the hardware:
- We recommend using a Linux OS; Ubuntu specifically.
- We recommend using an SSD (preferably an NVMe) with 1 TiB of storage. Since the DB can easily get corrupted, and syncing all over again can take days, it's highly recommended taking a daily backup of either the entire storage device (this way you'll have the correct version of Juno installed for the backup's DB) or the `~/.juno/data` directory (i.e., the DB).
- The architecture must be 64-bit (x86).
- We recommend 16 GiB of RAM.
- We recommend 2 CPU cores.
- Firewall:
    - If you require SSH access to your server, allow SSH connections over TCP on port 22 for your IP address.
    - If you require that clients be allowed to make API calls, allow HTTP connections on port 80 from any IP address, and HTTPS connections on port 443 from any IP address.

If you're using AWS, we recommend the following:
- Compute: AWS EC2
    - Instance type: `t3a.xlarge`
- Storage: AWS EBS
    - Use a `gp3` root volume because the others are either too slow (e.g., `gp2`) or expensive (e.g., `io2` is faster but not worth the money).
    - Use AWS Data Lifecycle Manager to create a daily backup.

## [Node Setup](docs/node-setup.md)

## [Operating the Node](docs/operating.md)

## License

This project is under the [MIT License](LICENSE).
