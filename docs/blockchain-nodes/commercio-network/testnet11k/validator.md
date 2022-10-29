# Validator

Follow these steps to set up a Commercio.network `testnet11k` validator:
1. Install Commercio.network:

    ```shell
    ####################
    ## BEGIN: Install ##
    ####################
   
    git clone https://github.com/commercionetwork/chains
    git clone https://github.com/commercionetwork/commercionetwork
    cd commercionetwork
    git checkout tags/(cat ~/chains/commercio-testnet11k/.data | grep -oP 'Release\s+\K\S+')
    make install
   
    ##################
    ## END: Install ##
    ##################
   
    ######################
    ## BEGIN: Configure ##
    ######################
   
    $DAEMON_NAME config chain-id commercio-testnet11k
   
    $DAEMON_NAME init $MONIKER
   
    cp ~/chains/commercio-testnet11k/genesis.json $DAEMON_HOME/config
   
    sed 's|minimum-gas-prices = .*|minimum-gas-prices = "1ucommercio"|' -i $DAEMON_HOME/config/app.toml
   
    set PEERS (cat ~/chains/commercio-testnet11k/.data | grep -oP 'Persistent peers\s+\K\S+')
    sed "s|persistent_peers = .*|persistent_peers = \"$PEERS\"|" -i $DAEMON_HOME/config/config.toml
   
    set SEEDS (cat ~/chains/commercio-testnet11k/.data | grep -oP 'Seeds\s+\K\S+')
    sed "s|seeds = .*|seeds = \"$SEEDS\"|" -i $DAEMON_HOME/config/config.toml
   
    set ADDRESS (curl icanhazip.com)
    sed "s|external_address = .*|external_address = \"$ADDRESS:26656\"|" -i $DAEMON_HOME/config/config.toml
   
    ####################
    ## END: Configure ##
    ####################
   
    #################
    ## BEGIN: Sync ##
    #################
   
    wget https://quicksync.commercio.network/commercio-testnet11k.latest.tgz -P $DAEMON_HOME/
    cd $DAEMON_HOME
    tar xvzf commercio-testnet11k.latest.tgz
    rm commercio-testnet11k.latest.tgz
   
    ###############
    ## END: Sync ##
    ###############
    ```
2. Only follow this step on servers for validators. Set up the [key](../../../key.md).
3. Set up [Cosmovisor](../../../cosmovisor.md).
4. Only follow this step on servers for validators where you haven't previously created a validator with the key's address. Create the validator:
    1. Get testnet tokens by asking on the [🤝Discussion](https://discord.com/channels/973149882032468029/984721633585553429) Discord channel.
    2. Create the validator:

        ```shell
        read -P 'Enter the max commission change rate percentage per day (example: 0.01): ' COMMISSION_MAX_CHANGE_RATE
        read -P 'Enter the max commission rate percentage (example: 0.2): ' COMMISSION_MAX_RATE
        read -P 'Enter the initial commission rate percentage (example: 0.05): ' COMMISSION_RATE
        read -P '(Optional) Enter the details (example: The most secure validator in the Cosmos!): ' DETAILS
        read -P 'Enter the fees to pay along with tx (example: 200000ucommercio): ' FEES
        read -P 'Enter the minimum self delegation required on the validator (example: 1): ' MIN_SELF_DELEGATION
        read -P '(Optional) Enter the security contact email address (example: security@example.com): ' SECURITY_CONTACT
        read -P '(Optional) Enter your website (example: https://validators.example.com): ' WEBSITE
        $DAEMON_NAME tx staking create-validator \
            --commission-max-change-rate $COMMISSION_MAX_CHANGE_RATE \
            --commission-max-rate $COMMISSION_MAX_RATE \
            --commission-rate $COMMISSION_RATE \
            --details $DETAILS \
            --fees $FEES \
            --min-self-delegation $MIN_SELF_DELEGATION \
            --security-contact $SECURITY_CONTACT \
            --website $WEBSITE \
            --amount 1100000ucommercio \
            --from $KEY \
            --moniker $MONIKER \
            --pubkey ($DAEMON_NAME tendermint show-validator)
        ```

       Open `https://testnet.commercio.network/transactions/detail/<HASH>`, where `<HASH>` is the value of the `txhash` field printed (e.g., `FDDE67944FBD6111EA9898D6F8B5CF9601B4935B5A17CE18209825311A036210`) in your browser to check if the validator was successfully created.
