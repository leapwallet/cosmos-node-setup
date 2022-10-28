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
4. Set the chain ID:

    ```shell
    osmosisd config chain-id osmosis-1
    ```
