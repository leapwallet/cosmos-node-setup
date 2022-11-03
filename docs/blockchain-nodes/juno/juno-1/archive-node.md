# Archive Node

Follow these steps to set up a Juno `juno-1` archive node:

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

   $DAEMON_NAME config chain-id juno-1

   $DAEMON_NAME init $MONIKER

   curl https://share.blockpane.com/juno/phoenix/genesis.json > $DAEMON_HOME/config/genesis.json

   # Prevent mempool from getting spammed.
   sed 's|minimum-gas-prices = .*|minimum-gas-prices = "0.0025ujuno"|' -i $DAEMON_HOME/config/app.toml

   ######################
   ## BEGIN: Set peers ##
   ######################

   set CHAIN_REPO https://raw.githubusercontent.com/CosmosContracts/mainnet/main/juno-1
   set PEERS (curl -sL $CHAIN_REPO/persistent_peers.txt)
   sed "s|persistent_peers = .*|persistent_peers = \"$PEERS\"|" -i $DAEMON_HOME/config/config.toml

   ####################
   ## END: Set peers ##
   ####################
   ```

2. Set up [Cosmovisor](../../../cosmovisor.md).
