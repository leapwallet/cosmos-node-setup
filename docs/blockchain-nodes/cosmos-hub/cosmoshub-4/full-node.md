# Full Node

Follow these steps to set up a Cosmos Hub `cosmoshub-4` full node:
1. Install Cosmos Hub:

    ```shell
    ###############################
    ## BEGIN: Install Cosmos Hub ##
    ###############################
   
    read -P 'Enter the latest git tag (see https://github.com/cosmos/gaia/releases/latest) such as v7.1.0: ' TAG 
    git clone -b $TAG https://github.com/cosmos/gaia
    cd gaia
    make install
  
    #############################
    ## END: Install Cosmos Hub ##
    #############################

    $DAEMON_NAME config chain-id cosmoshub-4
   
    $DAEMON_NAME init $MONIKER
   
    #############################
    ## BEGIN: Download genesis ##
    #############################
   
    wget https://raw.githubusercontent.com/cosmos/mainnet/master/genesis/genesis.cosmoshub-4.json.gz
    gzip -d genesis.cosmoshub-4.json.gz
    mv genesis.cosmoshub-4.json $DAEMON_HOME/config/genesis.json
   
    ###########################
    ## END: Download genesis ##
    ###########################
   
    # Prevent mempool from getting spammed.
    sed 's|minimum-gas-prices = .*|minimum-gas-prices = "0.0025uatom"|' -i $DAEMON_HOME/config/app.toml

    #################
    ## BEGIN: Sync ##
    #################
   
    cd $DAEMON_HOME
    set URL (curl -L https://quicksync.io/cosmos.json | jq -r '.[] | select(.file == "cosmoshub-4-pruned") | .url')
    aria2c -x5 $URL
    lz4 -cd (basename $URL) | tar xvf -
    rm (basename $URL)
   
    ###############
    ## END: Sync ##
    ###############
   
    curl https://dl2.quicksync.io/json/addrbook.cosmos.json > $DAEMON_HOME/config/addrbook.json
   
    # Set pruning to retain the last 22 days of blocks.
    sed \
        -e 's|pruning = .*|pruning = "custom"|' \
        -e 's|pruning-keep-recent = .*|pruning-keep-recent = "316800" # 22 days|' \
        -e 's|pruning-interval = .*|pruning-interval = "19" # Polkachu recommends using this prime number.|' \
        -i $DAEMON_HOME/config/app.toml
    ```
2. Set up [Cosmovisor](../../../cosmovisor.md).
