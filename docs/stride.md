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
        read -P 'Enter the maximum commission change rate percentage per day (e.g., 0.01): ' COMMISSION_MAX_CHANGE_RATE
        read -P 'Enter the maximum commission rate percentage (default: 0.20): ' COMMISSION_MAX_RATE
        read -P 'Enter the initial commission rate percentage (default: 0.10): ' COMMISSION_RATE
        read -P '(Optional) Enter the details (e.g., The most secure validator in the Cosmos!): ' DETAILS
        read -P 'Enter the fees to pay along with tx (e.g., 2000ustrd): ' FEES
        read -P 'Enter the minimum self delegation required on the validator (e.g., 1): ' MIN_SELF_DELEGATION
        read -P '(Optional) Enter the security contact email address (e.g., security@example.com): ' SECURITY_CONTACT
        read -P '(Optional) Enter your website (e.g., https://validators.example.com): ' WEBSITE
        read -P 'Enter the moniker you entered earlier: ' MONIKER
        strided tx staking create-validator \
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
            --pubkey (strided tendermint show-validator) \
            --chain-id STRIDE-TESTNET-4
        ```
