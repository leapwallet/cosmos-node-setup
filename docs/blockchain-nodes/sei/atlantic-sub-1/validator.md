## Validator

Follow these steps to set up a Sei `atlantic-sub-1` validator:
1. Install Sei:

    ```shell
    ########################
    ## BEGIN: Install Sei ##
    ########################
   
    git clone https://github.com/sei-protocol/sei-chain.git
    cd sei-chain
   
    set PROMPT 'You\'ll be prompted to enter a tag. The tag must be the most recent tag listed on'
    set PROMPT "$PROMPT https://github.com/sei-protocol/sei-chain/tags such as 1.1.2beta-internal). Enter the tag: "
    read -P $PROMPT TAG
   
    git checkout $TAG
    make install

    ######################
    ## END: Install Sei ##
    ######################
   
    $DAEMON_NAME config chain-id atlantic-sub-1
   
    $DAEMON_NAME init $MONIKER -o
    
    wget -O $DAEMON_HOME/config/genesis.json https://raw.githubusercontent.com/sei-protocol/testnet/main/atlantic-subchains/atlantic-sub-1/genesis.json
   
    wget -O $DAEMON_HOME/config/addrbook.json \
        https://raw.githubusercontent.com/sei-protocol/testnet/main/atlantic-subchains/atlantic-sub-1/addrbook.json

    sed 's|minimum-gas-prices = .*|minimum-gas-prices = "0.01usei"|' -i $DAEMON_HOME/config/app.toml
    ```
2. Only follow this step on servers for validators. Set up the [key](../../../key.md).
3. Set up [Cosmovisor](../../../cosmovisor.md).
4. Only follow this step on servers for validators where you haven't previously created a validator with the key's address. Create the validator:

    ```shell
    read -P 'Enter the max commission change rate percentage per day (example: 0.01): ' COMMISSION_MAX_CHANGE_RATE
    read -P 'Enter the max commission rate percentage (example: 0.2): ' COMMISSION_MAX_RATE
    read -P 'Enter the initial commission rate percentage (example: 0.05): ' COMMISSION_RATE
    read -P '(Optional) Enter the details (example: The most secure validator in the Cosmos!): ' DETAILS
    read -P 'Enter the fees to pay along with tx (example: 1usei): ' FEES
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
        --amount 1usei \
        --from $KEY \
        --moniker $MONIKER \
        --pubkey ($DAEMON_NAME tendermint show-validator)
    ```
