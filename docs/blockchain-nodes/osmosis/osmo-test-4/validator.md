# Validator

Follow these steps to set up a Osmosis `osmo-test-4` validator:
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
    osmosisd config chain-id osmo-test-4
    ```
5. Only follow this step on servers for validators. Set up the [key](../../../key.md).
6. Only follow this step on servers for validators where you haven't previously created a validator with the key's address. Create the validator:
   1. Get some tokens from the [faucet](https://faucet.osmosis.zone/#/) if you don't have any.
   2. Create the validator:

      ```shell
      read -P 'Enter the max commission change rate percentage per day (example: 0.01): ' COMMISSION_MAX_CHANGE_RATE
      read -P 'Enter the max commission rate percentage (example: 0.2): ' COMMISSION_MAX_RATE
      read -P 'Enter the initial commission rate percentage (example: 0.05): ' COMMISSION_RATE
      read -P '(Optional) Enter the details (example: The most secure validator in the Cosmos!): ' DETAILS
      read -P 'Enter the fees to pay along with tx (example: 2000uosmos): ' FEES
      read -P 'Enter the minimum self delegation required on the validator (example: 1): ' MIN_SELF_DELEGATION
      read -P '(Optional) Enter the security contact email address (example: security@example.com): ' SECURITY_CONTACT
      read -P '(Optional) Enter your website (example: https://validators.example.com): ' WEBSITE
      osmosisd tx staking create-validator \
          --commission-max-change-rate $COMMISSION_MAX_CHANGE_RATE \
          --commission-max-rate $COMMISSION_MAX_RATE \
          --commission-rate $COMMISSION_RATE \
          --details $DETAILS \
          --fees $FEES \
          --min-self-delegation $MIN_SELF_DELEGATION \
          --security-contact $SECURITY_CONTACT \
          --website $WEBSITE \
          --amount 2000uosmo \
          --from $KEY \
          --moniker $MONIKER \
          --pubkey (osmosisd tendermint show-validator)
      ```

      Note down the transaction hash (the value of the `txhash` field printed) which looks like `FDDE67944FBD6111EA9898D6F8B5CF9601B4935B5A17CE18209825311A036210`. Open `https://testnet.mintscan.io/osmosis-testnet/txs/<HASH>` (where `<HASH>` is the transaction hash) in your browser to check if the validator was successfully created.
