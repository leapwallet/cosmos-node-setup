# Sei Node Setup

1. Install Sei:

    ```shell
    ########################
    ## BEGIN: Install Sei ##
    ########################
   
    git clone https://github.com/sei-protocol/sei-chain.git
    cd sei-chain
   
    set PROMPT 'You\'ll be prompted to enter a tag. The tag must be the most recent tag listed on'
    set PROMPT "$PROMPT https://github.com/sei-protocol/sei-chain/tags (e.g., 1.2.0beta). Enter the tag: "
    read -P $PROMPT TAG
   
    git checkout $TAG
    make install

    ######################
    ## END: Install Sei ##
    ######################
   
    read -P 'A moniker is a name of your choosing for your node. Enter the moniker: ' MONIKER
    seid init $MONIKER --chain-id atlantic-1 -o
    
    curl https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/genesis.json > \
        ~/.sei/config/genesis.json
    curl https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/addrbook.json > \
        ~/.sei/config/addrbook.json
    ```
2. Set up the [key](key.md).
3. Configure Sei:

    ```shell
    sed 's|minimum-gas-prices = .*|minimum-gas-prices = "0.01usei"|' -i ~/.sei/config/app.toml
   
    ###########################
    ## BEGIN: Set up genesis ##
    ###########################
   
    read -P 'Enter the address: ' ADDRESS
    seid add-genesis-account $ADDRESS 100000000000000000000usei
    seid gentx $KEY 70000000000000000000usei --chain-id sei-chain
    seid collect-gentxs
    cat ~/.sei/config/genesis.json | \
        jq '.app_state["crisis"]["constant_fee"]["denom"]="usei"' > \
        ~/.sei/config/tmp_genesis.json && \
        mv ~/.sei/config/tmp_genesis.json ~/.sei/config/genesis.json
    cat ~/.sei/config/genesis.json | \
        jq '.app_state["gov"]["deposit_params"]["min_deposit"][0]["denom"]="usei"' > \
        ~/.sei/config/tmp_genesis.json && \
        mv ~/.sei/config/tmp_genesis.json ~/.sei/config/genesis.json
    cat ~/.sei/config/genesis.json | \
        jq '.app_state["mint"]["params"]["mint_denom"]="usei"' > \
        ~/.sei/config/tmp_genesis.json && \
        mv ~/.sei/config/tmp_genesis.json ~/.sei/config/genesis.json
    cat ~/.sei/config/genesis.json | \
        jq '.app_state["staking"]["params"]["bond_denom"]="usei"' > \
        ~/.sei/config/tmp_genesis.json && \
        mv ~/.sei/config/tmp_genesis.json ~/.sei/config/genesis.json
   
    #########################
    ## END: Set up genesis ##
    #########################
      
    ##############################
    ## BEGIN: Download snapshot ##
    ##############################
   
    seid tendermint --home ~/.sei unsafe-reset-all --keep-addr-book
    rm -rf ~/.sei/data ~/.sei/wasm
    set SNAP_NAME ( \
        curl -s https://snapshots1-testnet.nodejumper.io/sei-testnet/ | \
        grep -Eo '>atlantic-1.*\.tar.lz4' | \
        tr -d '>' \
    )
    
    # Enter a terminal multiplexer session for the next command because it's a long running process
    tmux
    curl https://snapshots1-testnet.nodejumper.io/sei-testnet/$SNAP_NAME | lz4 -dc - | tar xf - -C ~/.sei
    exit
      
    ############################
    ## END: Download snapshot ##
    ############################
    ```
4. Set up [Cosmovisor](cosmovisor.md).
5. Skip this step if you've previously created a validator with the address associated with this node. Create the validator:
   1. Get Sei testnet tokens from the [🚰 | atlantic-1-faucet](https://discord.com/channels/973057323805311026/979272741150687262) Discord channel by sending the message `!faucet <ADDRESS>` where `<ADDRESS>` is the address associated with this validator.
   2. Create the validator:

       ```shell
       seid tx staking create-validator \
           --amount=900000usei \
           --pubkey=(seid tendermint show-validator) \
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
