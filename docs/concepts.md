# Concepts

## Terms

This repo uses the following terms:
- [Juno](https://www.junonetwork.io/)
- [Sei](https://www.seinetwork.io/)
- When we refer to Juno mainnet archive nodes, we refer to Juno archive nodes running on the current mainnet (i.e., Juno Phoenix).
- When we refer to Sei testnet validator nodes, we refer to Sei validator nodes running on the incentivized testnet (i.e., `atlantic-1`).
- `<CHAIN_DIR>`: The path to where the configuration and data is stored (e.g., `$HOME/.juno` for Juno, `$HOME/.sei` for Sei).
- `<BINARY>`: The name of the node's executable (e.g., `junod` for Juno, `seid` for Sei).

## Archive Nodes

Nodes can be run by simply installing the latest version of the node, and downloading a snapshot. A snapshot is the blockchain's DB. There are different types of snapshots (e.g., full node snapshots, archive node snapshots). For example, the snapshot for a full node might only contain the latest 300 MiB of state even if the entire blockchain is 4.3 TiB. Since archive node snapshots are hard to come across (e.g., [ChainLayer](https://www.chainlayer.io/quicksync/) is one of the few services that provide archive node snapshots, but they don't have snapshots for most chains), archive nodes usually have to go through a significantly more complicated process before they can be used since they need to download the entire blockchain.

Whenever you're notified that there's going to be an upgrade (e.g., over a Discord channel) taking place (e.g., in 3 days at block 200), you'll have to upgrade the node using the following steps:
1. Configure the node to stop downloading blocks at the block height the next upgrade will take place at. For example, if v1 starts at block 1, and v2 starts at block 100, then the node must be configured to stop downloading blocks once it reaches block 100 if the node is currently on v1. Otherwise, the node will be unable to process blocks produced by newer node versions causing the DB to get corrupted.
2. Once the node stops syncing, replace the current node version with the next version, and resume syncing. For example, if the node is on v1, and the next version is v1.1, then v1 must be uninstalled, and v1.1 must be installed.

Here's how to set up an archive node with a snapshot:
1. Install the version of the node that the snapshot supports. For example, if the latest node version is 9.0.0, and the snapshot has the blockchain state until last week back when the latest version was 8.0.0, then you'll have to install v8.0.0.
2. Download the snapshot.
3. Repeatedly upgrade the node until it's on the latest version.

Here's how to set up an archive node without a snapshot:
1. Install the first version of the node.
2. Repeatedly upgrade the node until all the older blocks have been downloaded.

## Cosmovisor

Cosmovisor is a tool that aims to ease upgrading nodes. Theoretically, it's supposed to automatically download upgrades in advance, and automatically reinstall and restart the node at the correct block height. In practice, it doesn't work as advertised. Here's why we recommend not using Cosmovisor:
- You need to spend time installing it.
- You need to spend time learning it (different versions have different APIs).
- You need to spend time updating it, and the updates are buggy (e.g., v1.1.0 can't be installed directly).
- There are two types of upgrades - planned and unplanned. Planned upgrades are voted on via governance proposals, and add enhancements. Unplanned upgrades are emergency security fixes that aren't voted on via governance proposals, and don't add any enhancements. Unplanned upgrades make up about half of the upgrades, and aren't supported by Cosmovisor which means that you'll still need to remember how to manually upgrade the node. The problem with manually upgrading the node while using Cosmovisor is that you'll need to perform additional steps such as copying the executable to the Cosmovisor directory. If you don't copy the executable correctly, then the node wouldn't have properly upgraded which will cause the DB to get corrupted.
- If you don't correctly perform an upgrade, then the DB will get corrupted. If the DB gets corrupted, you'll have to restore a backup of the node so that you don't need to start from scratch, and you'll have to downgrade the node to the version to one that the backup supports. If you're using Cosmovisor, then the DB can get corrupted again after restoring the backup, and downgrading the node. The reason is because Cosmovisor will automatically upgrade the node to a newer version unless you remember to run some commands such as deleting the upgrades Cosmovisor kept copies of. Also, when downgrading the node, the Cosmovisor directory to copy the executable to changes in an undocumented manner.
- Due to a bug, you must spend time configuring Cosmovisor to not automatically download upgrades.
- You'll still want to monitor the upgrade as it happens with Cosmovisor in case the upgrade fails which happens a noticeable number of times.
