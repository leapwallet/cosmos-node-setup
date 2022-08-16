# Stride Node Setup

1. Install:

    ```shell
    bash -c "$(curl -sSL install.poolparty.stridelabs.co)"
    ```
2. Set up the [key](key.md).
3. Skip this step if you've previously created a validator with the address associated with this node. Create the validator:
    1. Get some Stride testnet tokens from the [💧 | token-faucet](https://discord.com/channels/988945059783278602/992572020535599244) Discord channel.
    2. Create the validator:

        ```shell
        read -P 'Enter the moniker you used earlier: ' MONIKER
        strided tx staking create-validator \
            --chain-id STRIDE-1 \
            --commission-rate 0.05 \
            --commission-max-rate 0.2 \
            --commission-max-change-rate 0.1 \
            --min-self-delegation 1000000 \
            --amount 1000000ustrd \
            --pubkey (strided tendermint show-validator) \
            --moniker $MONIKER \
            --gas auto \
            --from $KEY
        ```
