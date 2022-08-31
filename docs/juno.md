# Juno Node Setup

Follow these steps to set up a Juno mainnet archive node:
1. Install Juno:

    ```shell
    #########################
    ## BEGIN: Install Juno ##
    ######################### 
   
    git clone https://github.com/CosmosContracts/juno
    cd juno
   
    set PROMPT 'You\'ll be prompted to enter a tag. If you\'re going to download a snapshot, then enter the version\'s'
    set PROMPT "$PROMPT tag that the snapshot supports. Otherwise, enter the first version\'s tag (i.e., v3.0.0). Tags""
    set PROMPT "$PROMPT can be found at https://github.com/CosmosContracts/juno/tags. Enter the tag: "
    read -P $PROMPT TAG
   
    git checkout $TAG
    make install
  
    #######################
    ## END: Install Juno ##
    ####################### 

    read -P 'A moniker is a name of your choosing for your node. Enter the moniker: ' MONIKER
   
    junod config chain-id juno-1
   
    junod init $MONIKER
      
    curl https://share.blockpane.com/juno/phoenix/genesis.json > ~/.juno/config/genesis.json
   
    ######################
    ## BEGIN: Set peers ##
    ###################### 
   
    set CHAIN_REPO https://raw.githubusercontent.com/CosmosContracts/mainnet/main/juno-1
    set PEERS (curl -sL $CHAIN_REPO/persistent_peers.txt)
    sed "s|persistent_peers = .*|persistent_peers = \"$PEERS\"|" -i ~/.juno/config/config.toml
       
    ####################
    ## END: Set peers ##
    ####################    
    ```
2. Follow this step if you want to enable the REST API. In the `[api]` section of `~/.juno/config/app.toml`, set the `enable` key's value to `true`.
3. Set up [Cosmovisor](cosmovisor.md).
