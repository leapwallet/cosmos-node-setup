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

## Genesis Error

If the node keeps crashing with the following error message:

```
Error: error during handshake: error on replay: validator set is nil in genesis and still empty after InitChain
```

then you must run `junod unsafe-reset-all` which will wipe the DB. This error only has the potential to surface when initially setting up the node, and not after the node has been downloading blocks, upgrading, etc.

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
    sed -i "s/persistent_peers = .*/persistent_peers = \"$PEERS\"/" ~/.juno/config/config.toml
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
7. Update the halt height:

    ```sh
    sed -i 's/halt-height = .*/halt-height = <HEIGHT>/' ~/.juno/config/app.toml
    ```

    Replace `<HEIGHT>` with the next upgrade's block height if there is one, and `0` otherwise.
8. Reload systemd:

    ```sh
    sudo systemctl daemon-reload
    ```
9. Restart Juno:

    ```sh
    sudo systemctl restart junod
    ```
