# Validator

Follow these steps to set up a Stride `STRIDE-TESTNET-4` validator:
1. Install:

    ```shell
    bash -c "$(curl -sSL install.poolparty.stridelabs.co)"
    $DAEMON_NAME config chain-id STRIDE-TESTNET-4
    ```
2. Only follow this step on servers for validators. Set up the [key](../../../key.md).
3. Only follow this step on servers for validators where you haven't previously created a validator with the key's address. Create the validator:
    1. Get some Stride testnet tokens from the [💧 | token-faucet](https://discord.com/channels/988945059783278602/992572020535599244) Discord channel.
    2. Create the validator:

        ```shell
        read -P 'Enter the max commission change rate percentage per day (example: 0.01): ' COMMISSION_MAX_CHANGE_RATE
        read -P 'Enter the max commission rate percentage (example: 0.20): ' COMMISSION_MAX_RATE
        read -P 'Enter the initial commission rate percentage (example: 0.10): ' COMMISSION_RATE
        read -P '(Optional) Enter the details (e.g., The most secure validator in the Cosmos!): ' DETAILS
        read -P 'Enter the fees to pay along with tx (e.g., 2000ustrd): ' FEES
        read -P 'Enter the minimum self delegation required on the validator (e.g., 1): ' MIN_SELF_DELEGATION
        read -P '(Optional) Enter the security contact email address (e.g., security@example.com): ' SECURITY_CONTACT
        read -P '(Optional) Enter your website (e.g., https://validators.example.com): ' WEBSITE
        read -P 'Enter the moniker you entered earlier: ' MONIKER
        $DAEMON_NAME tx staking create-validator \
            --commission-max-change-rate $COMMISSION_MAX_CHANGE_RATE \
            --commission-max-rate $COMMISSION_MAX_RATE \
            --commission-rate $COMMISSION_RATE \
            --details $DETAILS \
            --fees $FEES \
            --min-self-delegation $MIN_SELF_DELEGATION \
            --security-contact $SECURITY_CONTACT \
            --website $WEBSITE \
            --amount 900000ustrd \
            --from $KEY \
            --moniker $MONIKER \
            --pubkey ($DAEMON_NAME tendermint show-validator) \
        ```

       Open `https://stride.explorers.guru/transaction/<HASH>`, where `<HASH>` is the value of the `txhash` field printed (e.g., `E6BFE17E108F09A22F517F2092F7FC44B1BC4A5CED9D2B98E9B0C04944650D06`) in your browser to check if the validator was successfully created.
