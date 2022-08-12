# Sei Node Setup

1. Install Sei:

    ```shell
    git clone https://github.com/sei-protocol/sei-chain.git
    cd sei-chain
    read -p 'Enter the tag: ' TAG
    git checkout $TAG
    make install
    seid version
    read -p $'A moniker is the node\'s moniker which is a name of your choosing. Enter the moniker: ' MONIKER
    seid init $MONIKER --chain-id atlantic-1 -o
    curl https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/genesis.json > \
        $HOME/.sei/config/genesis.json
    curl https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/addrbook.json > \
        $HOME/.sei/config/addrbook.json
    sed 's|minimum-gas-prices = .*|minimum-gas-prices = "0.01usei"|' -i $HOME/.sei/config/app.toml
    read -p 'A key is named whatever you want. Enter they key: ' KEY
    ```
   
    You'll be prompted to enter a tag. The tag is the most recent [tag](https://github.com/sei-protocol/sei-chain/tags) (e.g., `1.2.0beta`).
2. If you want to make and use a new Sei wallet for the validator node, then follow this step:

    ```shell
    seid keys add $KEY
    ```

    In case you need to recreate this validator (e.g., the hardware it was running on got destroyed), entering the same mnemonic will not generate the same `$HOME/.sei/config/priv_validator_key.json` and `$HOME/.sei/config/node_key.json`. Therefore, you must back up `$HOME/.sei/config/priv_validator_key.json` and `$HOME/.sei/config/node_key.json` in case you need to restore your validator later on. Preferably, encrypt the backup.
3. If you want to use an existing Sei wallet for the validator node, then follow this step:

    ```shell
    seid keys add $KEY --recover
    ```

    If you're recreating a validator, and have a backup of `$HOME/.sei/config/priv_validator_key.json` and `$HOME/.sei/config/node_key.json`, replace the generated files at the same path with the backup.
4. Configure Sei:

    ```shell
    read -p 'Enter the address: ' ADDRESS
    seid add-genesis-account $ADDRESS 100000000000000000000usei
    seid gentx $KEY 70000000000000000000usei --chain-id sei-chain
    seid collect-gentxs
    cat $HOME/.sei/config/genesis.json | \
        jq '.app_state["crisis"]["constant_fee"]["denom"]="usei"' > \
        $HOME/.sei/config/tmp_genesis.json && \
        mv $HOME/.sei/config/tmp_genesis.json $HOME/.sei/config/genesis.json
    cat $HOME/.sei/config/genesis.json | \
        jq '.app_state["gov"]["deposit_params"]["min_deposit"][0]["denom"]="usei"' > \
        $HOME/.sei/config/tmp_genesis.json && \
        mv $HOME/.sei/config/tmp_genesis.json $HOME/.sei/config/genesis.json
    cat $HOME/.sei/config/genesis.json | \
        jq '.app_state["mint"]["params"]["mint_denom"]="usei"' > \
        $HOME/.sei/config/tmp_genesis.json && \
        mv $HOME/.sei/config/tmp_genesis.json $HOME/.sei/config/genesis.json
    cat $HOME/.sei/config/genesis.json | \
        jq '.app_state["staking"]["params"]["bond_denom"]="usei"' > \
        $HOME/.sei/config/tmp_genesis.json && \
        mv $HOME/.sei/config/tmp_genesis.json $HOME/.sei/config/genesis.json
    seid tendermint --home $HOME/.sei unsafe-reset-all --keep-addr-book
    rm -rf $HOME/.sei/data $HOME/.sei/wasm
    SNAP_NAME=$( \
        curl -s https://snapshots1-testnet.nodejumper.io/sei-testnet/ | \
        grep -Eo '>atlantic-1.*\.tar.lz4' | \
        tr -d '>' \
    )
    curl https://snapshots1-testnet.nodejumper.io/sei-testnet/$SNAP_NAME | lz4 -dc - | tar xf - -C $HOME/.sei
    sudo tee /etc/systemd/system/seid.service << EOF
    [Unit]
    Description=Sei Daemon
    Wants=network-online.target
    After=network-online.target

    [Service]
    Type=simple
    User=$USER
    ExecStart=$(which seid) start
    RestartSec=3
    Restart=always
    LimitNOFILE=65535
    LimitMEMLOCK=209715200

    [Install]
    WantedBy=multi-user.target
    EOF
    sudo systemctl enable --now seid
    sudo systemctl status seid
    ```

    You'll be prompted to enter an address. The address is the value of the `address` key (e.g., `sei19vf5mfr40awvkefw69nl6p3mmlsnacmm8thjxk`) printed in the previous step.

    The unit's status will be displayed for you to check whether it was successfully set up.
5. Get some Sei testnet tokens from the [🚰 | atlantic-1-faucet](https://discord.com/channels/973057323805311026/979272741150687262) Discord channel.
6. Create the validator:

    ```shell
    seid tx staking create-validator \
        --amount=900000usei \
        --pubkey=$(seid tendermint show-validator) \
        --moniker="$MONIKER" \
        --chain-id=atlantic-1 \
        --from="$KEY" \
        --commission-rate=0.10 \
        --commission-max-rate=0.20 \
        --commission-max-change-rate=0.01 \
        --min-self-delegation=1 \
        --fees=2000usei
    ```

    Open `https://sei.explorers.guru/transaction/<HASH>`, where `<HASH>` is the value of the `txhash` field printed (e.g., `FDDE67944FBD6111EA9898D6F8B5CF9601B4935B5A17CE18209825311A036210`) in your browser to check if the validator was successfully created.
