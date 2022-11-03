# Full Node

Follow these steps to set up an Osmosis `osmosis-1` full node:

1. Install:

   ```shell
   curl -sL https://get.osmosis.zone/install > i.py && python3 i.py
   ```

2. Update config:

   ```shell
   $DAEMON_NAME config chain-id osmosis-1

   # Prevent mempool from getting spammed.
   sed 's|minimum-gas-prices = .*|minimum-gas-prices = "0.0025uosmo"|' -i $DAEMON_HOME/config/app.toml

   # Set pruning to retain the last 22 days of blocks.
   sed \
       -e 's|pruning = .*|pruning = "custom"|' \
       -e 's|pruning-keep-recent = .*|pruning-keep-recent = "316800" # 22 days|' \
       -e 's|pruning-interval = .*|pruning-interval = "19" # Polkachu recommends using this prime number.|' \
       -i $DAEMON_HOME/config/app.toml
   ```
