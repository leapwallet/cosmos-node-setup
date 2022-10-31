# Concepts

## Prerequisites

Prior to using this repo, it's recommended that you learn the following:
- Sysadmin skills such as Linux commands (`systemd`, `sed`, `tmux`, etc.), firewalls, DNS, etc. 
- [Tendermint](https://docs.tendermint.com/)
- [Cosmos SDK](https://docs.cosmos.network/)
- (Required if you're setting up a validator.) [Horcrux](https://github.com/strangelove-ventures/horcrux)
- (Required if you want to monitor your blockchain node.) [PANIC](https://github.com/SimplyVC/panic)
- (Required if you want to monitor your blockchain node.) [Prometheus](https://prometheus.io/)
- (Required if you want to monitor your blockchain node.) [Grafana](https://grafana.com/docs/grafana/latest/)
- (Optional) [Caddy](https://caddyserver.com/)

## PANIC Monitoring and Alerting for Blockchains

_PANIC Monitoring and Alerting for Blockchains_ is abbreviated as _PANIC_.

## Validator Node

_Validator node_ is abbreviated as _validator_.

## Full Node

A full node is any non-validator blockchain node such as an RPC node, sentry, archive node but not a validator.

## Horcrux

_Sentry node_ is abbreviated as _sentry_. A sentry is a full node that connects to cosigners to sign blocks.

## Monitor

By _monitor_, we mean a server that runs monitoring and alerting software such as PANIC and Prometheus.

Validators will be jailed if they double sign. Double signing can only occur if two instances of a validator using the same private key are active at the same time. You needn't worry about a validator double signing if it restarts during a block because it locally stores which blocks it's processed.

## Blockchain Node

_Blockchain node_ refers to _validator_ or _full node_.

## Archive Node

Nodes can be run by simply installing the latest version of the node, and downloading a snapshot. A snapshot is the blockchain's DB. There are different types of snapshots such as full node snapshots, and archive node snapshots. For example, the snapshot for a full node might only contain the latest 300 MiB of state even if the entire blockchain is 4.3 TiB. Archive node snapshots are hard to come across. For example, [ChainLayer](https://www.chainlayer.io/quicksync/) is one of the few services that provide archive node snapshots, but they don't have snapshots for most chains. Therefore, archive nodes usually have to go through a significantly more complicated process before they can be used since they need to download the entire blockchain.

Whenever you're notified that there's going to be an upgrade taking place, you'll have to upgrade the node using the following steps:
1. Configure the node to stop downloading blocks at the block height the next upgrade will take place at. For example, if v1 starts at block 1, and v2 starts at block 100, then the node must be configured to stop downloading blocks once it reaches block 100 if the node is currently on v1. Otherwise, the node will be unable to process blocks produced by newer node versions causing the DB to get corrupted.
2. Once the node stops syncing, replace the current node version with the next version, and resume syncing. For example, if the node is on v1, and the next version is v1.1, then v1 must be uninstalled, and v1.1 must be installed.

Here's how to set up an archive node with a snapshot:
1. Install the version of the node that the snapshot supports. For example, if the latest node version is 9.0.0, and the snapshot has the blockchain state until last week back when the latest version was 8.0.0, then you'll have to install v8.0.0.
2. Download the snapshot.
3. Repeatedly upgrade the node until it's on the latest version.

Here's how to set up an archive node without a snapshot:
1. Install the first version of the node.
2. Repeatedly upgrade the node until all the older blocks have been downloaded.
