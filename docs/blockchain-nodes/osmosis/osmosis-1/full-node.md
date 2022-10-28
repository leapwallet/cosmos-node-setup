# Full Node

Follow these steps to set up an Osmosis `osmosis-1` full node:
1. Enter a terminal multiplexer session because the next step is a long-running process:

    ```shell
    tmux
    ```
2. Install:

    ```shell
    curl -sL https://get.osmosis.zone/install > i.py && python3 i.py
    ```
3. Exit the terminal multiplexer:

    ```shell
    exit
    ```   
4. Update config:

    ```shell
    osmosisd config chain-id osmosis-1
   
    # Update pruning configuration to retain 22 days worth of blocks.
    sed \
        -e 's|pruning = .*|pruning = "custom"|' \
        -e 's|pruning-keep-recent = .*|pruning-keep-recent = "316800" # 22 days|' \
        -e 's|pruning-interval = .*|pruning-interval = "19" # Polkachu recommends using this prime number.|' \
        -i ~/.sei/config/app.toml
    ```
