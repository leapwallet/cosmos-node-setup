# Sei Node Setup

1. Install Sei:

    ```shell
    ########################
    ## BEGIN: Install Sei ##
    ########################
   
    git clone https://github.com/sei-protocol/sei-chain.git
    cd sei-chain
   
    set PROMPT 'You\'ll be prompted to enter a tag. The tag must be the most recent tag listed on'
    set PROMPT "$PROMPT https://github.com/sei-protocol/sei-chain/tags (e.g., 1.1.0beta). Enter the tag: "
    read -P $PROMPT TAG
   
    git checkout $TAG
    make install

    ######################
    ## END: Install Sei ##
    ######################
   
    read -P 'A moniker is a name of your choosing for your node. Enter the moniker: ' MONIKER
    printf "\nset MONIKER $MONIKER\n" >> ~/.config/fish/config.fish
   
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
   
    set ADDRESS (seid keys show -a $KEY)
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
    curl https://snapshots1-testnet.nodejumper.io/sei-testnet/$SNAP_NAME | lz4 -dc - | tar xf - -C ~/.sei
      
    ############################
    ## END: Download snapshot ##
    ############################
    ```
4. Set up [Cosmovisor](cosmovisor.md).
5. Skip this step if you've previously created a validator with the address associated with this node. Create the validator:
    1. Get Sei testnet tokens from the [🚰 | atlantic-1-faucet](https://discord.com/channels/973057323805311026/979272741150687262) Discord channel by sending the message `!faucet <ADDRESS>` where `<ADDRESS>` is the address associated with this validator.
    2. Create the validator:

        ```shell
        read -P 'Enter the max commission change rate percentage per day (example: 0.01): ' COMMISSION_MAX_CHANGE_RATE
        read -P 'Enter the max commission rate percentage (example: 0.2): ' COMMISSION_MAX_RATE
        read -P 'Enter the initial commission rate percentage (example: 0.05): ' COMMISSION_RATE
        read -P '(Optional) Enter the details (example: The most secure validator in the Cosmos!): ' DETAILS
        read -P 'Enter the fees to pay along with tx (example: 2000usei): ' FEES
        read -P 'Enter the minimum self delegation required on the validator (example: 1): ' MIN_SELF_DELEGATION
        read -P '(Optional) Enter the security contact email address (example: security@example.com): ' SECURITY_CONTACT
        read -P '(Optional) Enter your website (example: https://validators.example.com): ' WEBSITE
        seid tx staking create-validator \
            --commission-max-change-rate $COMMISSION_MAX_CHANGE_RATE \
            --commission-max-rate $COMMISSION_MAX_RATE \
            --commission-rate $COMMISSION_RATE \
            --details $DETAILS \
            --fees $FEES \
            --min-self-delegation $MIN_SELF_DELEGATION \
            --security-contact $SECURITY_CONTACT \
            --website $WEBSITE \
            --amount 900000usei \
            --from $KEY \
            --moniker $MONIKER \
            --pubkey (seid tendermint show-validator) \
            --chain-id atlantic-1
        ```

       Open `https://sei.explorers.guru/transaction/<HASH>`, where `<HASH>` is the value of the `txhash` field printed (e.g., `FDDE67944FBD6111EA9898D6F8B5CF9601B4935B5A17CE18209825311A036210`) in your browser to check if the validator was successfully created.
