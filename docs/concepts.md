# Concepts

## Why AWS

People say that bare metal is multiple times cheaper than public cloud. This is true if your time is free. Bare metal
servers require manually setting up firewalls, learning and using tools such as Ansible, etc. which take exponentially
more time to learn, use, and secure than public clouds. In the long run, an experienced sysadmin could potentially save
money using bare metal servers because they'll only ever run a single specialized piece of software (i.e., a Juno node)
but otherwise, it's recommended to use a public cloud.

Regarding the choice of public cloud, GCP is obviously better than AWS but this document uses AWS because the author had
to set it up on that due to external factors.

## Juno Versions

Due to an exploit, the Juno chain hard forked into Juno Phoenix (v3.0.0), and the original chain is named Juno Classic.
This document is for Juno Phoenix which is the current mainnet.

## Archive Nodes

Validator nodes can be run by simply installing the latest version of the node, and downloading a snapshot. A snapshot
is a fraction of the blockchain's complete state (e.g., a snapshot could just be the latest 300 MiB of a blockchain even
if the entire blockchain is 4.3 TiB). However, archive nodes must go through a significantly more complicated process
before they can be used because they need to download the entire blockchain.

In order to set up an archive node, the following steps must be taken:

1. Install the first version of the node. For example, if the first version is 1, and the latest version is 10, then
   you'll have to download v1.
2. Configure the node to stop downloading blocks at the block height the next upgrade took place at. For example, if v1
   starts at block 1, and v2 starts at block 100, then the node must be configured to stop downloading blocks once it
   reaches block 100 if the node is currently on v1. Otherwise, the current version will be unable to process blocks
   produced by newer node versions causing the database to get corrupted.
3. Once the node stops syncing, replace the current node version with the next version, and resume syncing. For example,
   if the node is on v1, and the next version is v1.1, then v1 must be uninstalled, and v1.1 must be installed.
4. After installing the new version, set the next block height to stop syncing at just as before, and upgrade the
   software just as before. Repeat this process until the node has finished downloading all the older blocks.
5. Once the node is on the latest block, you'll have to perform the same steps that validator nodes do for upgrades
   which is the same as the upgradation process you've previously followed. In essence, once you've been notified that
   there's going to be an upgrade (e.g., over a Discord channel) taking place (e.g., in 3 days at block 200), you'll
   have to configure the node to stop syncing at that block height, and upgrade the software.

## Cosmovisor

Cosmovisor is a tool that aims to ease upgrading nodes. Theoretically, it's supposed to automatically download upgrades
in advance, and automatically reinstall and restart the node at the correct block height. In practice, it doesn't work
as advertised. Here's why we recommend not using Cosmovisor:

- You need to spend time installing it.
- You need to spend time learning it (different versions have different APIs).
- You need to spend time updating it, and the updates are buggy (e.g., v1.1.0 can't be installed directly).
- There are two types of upgrades - planned and unplanned. Planned upgrades are voted on via governance proposals, and
  add enhancements. Unplanned upgrades are emergency security fixes that aren't voted on via governance proposals, and
  don't add any enhancements. Unplanned upgrades make up about half of the upgrades, and aren't supported by Cosmovisor
  which means that you'll still need to remember how to manually upgrade the node. The problem with manually upgrading
  the node while using Cosmovisor is that you'll need to perform additional steps such as copying the executable to the
  Cosmovisor directory. If you don't copy the executable correctly, then the node wouldn't have properly upgraded which
  will cause the DB to get corrupted.
- If you don't correctly perform an upgrade, then the DB will get corrupted. If the DB gets corrupted, you'll have to
  restore a backup of the node so that you don't need to start from scratch, and you'll have to downgrade the node to
  the version to one that the backup supports. If you're using Cosmovisor, then the DB can get corrupted again after
  restoring the backup, and downgrading the node. The reason is because Cosmovisor will automatically upgrade the node
  to a newer version unless you remember to run some commands such as deleting the upgrades Cosmovisor kept copies of.
  Also, when downgrading the node, the Cosmovisor directory to copy the executable to changes in an undocumented manner.
- Due to a bug, you must spend time configuring Cosmovisor to not automatically download upgrades.
- You'll still want to monitor the upgrade as it happens with Cosmovisor in case the upgrade fails which happens a
  noticeable number of times.
