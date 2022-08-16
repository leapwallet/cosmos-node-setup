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

    read -P 'A moniker is the node\'s moniker which is a name of your choosing. Enter the moniker: ' MONIKER
    junod init $MONIKER --chain-id juno-1
      
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
   
    ############################
    ## BEGIN: Set halt height ##
    ############################

    set PROMPT 'Enter the block height to halt at (e.g., 2616300 if you downloaded the first version of the node, 0 if'
    set PROMPT "$PROMPT you installed the latest version): "
    read -P $PROMPT HEIGHT
    sed "s|halt-height = .*|halt-height = $HEIGHT|" -i ~/.juno/config/app.toml
   
    ##########################
    ## END: Set halt height ##
    ##########################
    ```
2. Follow this step if you want to enable the REST API. In the `[api]` section of `~/.juno/config/app.toml`, set the `enable` key's value to `true`.
3. Set up [Cosmovisor](cosmovisor.md).
