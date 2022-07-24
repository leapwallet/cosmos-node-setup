# Operating the Node

## Sync Status

```sh
curl -s http://localhost:26657/status | jq .result.sync_info.catching_up
```

Prints `true` if the node is still downloading the older blocks, and `false` otherwise.

## Logs

```sh
journalctl -fu junod
```

Follows the journal for the `junod` systemd unit. This is useful to check whether the chain has halted, which block it's currently downloading, any error message that got printed if the node crashed, etc.

## systemd Unit

The value of the `Restart` directive in the `/etc/systemd/system/junod.service` file must be either `no` or `always`. If the `--halt-height` flag has been set when running `junod`, then the value must be `no` because otherwise the node will automatically restart thereby corrupting the database by prcoessing blocks it wasn't programmed to. Otherwise, the value must be `always`.

## Genesis Error

If the node keeps crashing with the following error message:

```
Error: error during handshake: error on replay: validator set is nil in genesis and still empty after InitChain
```

then you must restore the node via a backup. If you don't have a backup, then you'll have to run the command `junod unsafe-reset-all` which will wipe the DB causing you to have to restart the entire syncing process.

## Peers

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

## Available Upgrades

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

## Upgrading

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
