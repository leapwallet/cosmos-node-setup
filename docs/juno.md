# Juno Node Setup

Follow these steps to set up a Juno mainnet archive node:
1. Install Juno:

    ```shell
    git clone https://github.com/CosmosContracts/juno
    cd juno
    read -p 'Enter the tag: ' TAG
    git checkout $TAG
    make install
    CHAIN_REPO="https://raw.githubusercontent.com/CosmosContracts/mainnet/main/juno-1"
    PEERS="$(curl -sL "$CHAIN_REPO/persistent_peers.txt")"
    read -p $'A moniker is the node\'s moniker which is a name of your choosing. Enter the moniker: ' MONIKER
    junod init $MONIKER --chain-id juno-1
    curl https://share.blockpane.com/juno/phoenix/genesis.json > $HOME/.juno/config/genesis.json
    sed "s|persistent_peers = .*|persistent_peers = \"$PEERS\"|" -i $HOME/.juno/config/config.toml
    read -p 'Enter the height: ' HEIGHT
    sed 's|halt-height = 0|halt-height = $HEIGHT|' -i $HOME/.juno/config/app.toml
    ```

    You'll be prompted to enter a tag. If you're going to download a snapshot, then enter the version's tag (tags can be found [here](https://github.com/CosmosContracts/juno/tags)) that the snapshot supports. Otherwise, enter the first version's tag (i.e., `v3.0.0`).

    You'll be prompted to enter a height. The height is the block height to halt at (e.g., `2616300` if you downloaded the first version of the node, `0` if you installed the latest version of the node).
2. Follow this step if you want to enable the REST API. In the `[api]` section of `$HOME/.juno/config/app.toml`, set the `enable` key's value to `true`.
3. Set up the Juno systemd unit:

    ```shell
    sudo tee /etc/systemd/system/junod.service << EOF
    [Unit]
    Description=Juno Daemon
    Wants=network-online.target
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=$(which junod) start --x-crisis-skip-assert-invariants
    RestartSec=3
    Restart=always
    LimitNOFILE=4096

    [Install]
    WantedBy=multi-user.target
    EOF
    sudo systemctl enable --now junod
    sudo systemctl status junod
    ```

    The unit's status will be displayed for you to check whether it was successfully set up.
