# Cosigner Setup

1. Follow this step on each cosigner. Install Horcrux:

   ```shell
   ############################
   ## BEGIN: Install Horcrux ##
   ############################

   set URL https://github.com/strangelove-ventures/horcrux/releases/download/v2.0.0/horcrux_2.0.0_linux_amd64.tar.gz
   wget $URL
   tar xvzf (basename $URL)
   sudo mv horcrux /usr/bin/horcrux
   rm (basename $URL) README.md LICENSE.md

   ##########################
   ## END: Install Horcrux ##
   ##########################

   ############################################
   ## BEGIN: Set up systemd unit for Horcrux ##
   ############################################

   printf "\
   [Unit]
   Description=MPC Signer node
   After=network.target

   [Service]
   Type=simple
   User=$USER
   WorkingDirectory=/home/ubuntu
   ExecStart=/usr/bin/horcrux cosigner start
   Restart=on-failure
   RestartSec=3
   LimitNOFILE=4096

   [Install]
   WantedBy=multi-user.target
   " > sudo tee /etc/systemd/system/horcrux.service

   sudo systemctl daemon-reload

   ##########################################
   ## END: Set up systemd unit for Horcrux ##
   ##########################################

   ##############################
   ## BEGIN: Configure Horcrux ##
   ##############################

   read -P 'Enter the chain ID such as osmo-test-4: ' CHAIN_ID

   set PROMPT 'Enter the respective sentry\'s Horcrux URL such as tcp://10.168.0.1:1234 or'
   set PROMPT "$PROMPT sentry-1.osmosis.osmo-test-4.example.com/private-validator: "
   read -P $PROMPT SENTRY

   set PROMPT 'Enter the first cosigner\'s Horcrux URL such as tcp://10.168.1.1:2222 or'
   set PROMPT "$PROMPT cosigner-1.osmosis.osmo-test-4.example.com/signer: "
   read -P $PROMPT COSIGNER_1

   set PROMPT 'Enter the second cosigner\'s Horcrux URL such as tcp://10.168.1.2:2222 or'
   set PROMPT "$PROMPT cosigner-2.osmosis.osmo-test-4.example.com/signer: "
   read -P $PROMPT COSIGNER_2

   set PROMPT 'Enter the third cosigner\'s Horcrux URL such as tcp://10.168.1.3:2222 or'
   set PROMPT "$PROMPT cosigner-3.osmosis.osmo-test-4.example.com/signer: "
   read -P $PROMPT COSIGNER_3

   read -P 'Enter which cosigner this is [1/2/3]: ' COSIGNER

   switch $COSIGNER
       case 1
           horcrux config init $CHAIN_ID $SENTRY \
               -c \
               -p "$COSIGNER_2|2,$COSIGNER_3|3" \
               -l $COSIGNER_1 \
               -t 2 \
               --timeout 1500ms
       case 2
           horcrux config init $CHAIN_ID $SENTRY \
               -c \
               -p "$COSIGNER_1|1,$COSIGNER_3|3" \
               -l $COSIGNER_2 \
               -t 2 \
               --timeout 1500ms
       case 3
           horcrux config init $CHAIN_ID $SENTRY \
               -c \
               -p "$COSIGNER_1|1,$COSIGNER_2|2" \
               -l $COSIGNER_3 \
               -t 2 \
               --timeout 1500ms
   end

   ############################
   ## END: Configure Horcrux ##
   ############################
   ```

2. Copy the private validator key file. This file is originally generated to a location like `seid/config/priv_validator_key.json` on the server for the validator. Paste the file onto the first cosigner.
3. Split the private key file on the first cosigner:

   ```shell
   horcrux create-shares priv_validator_key.json 2 3
   ```

   Three files, each named something like `private_share_1.json`, have been generated.

4. On the first cosigner, create a file `~/.horcrux/share.json`, and paste the contents of `private_share_1` (generated from step 3) into it.
5. On the second cosigner, create a file `~/.horcrux/share.json`, and paste the contents of `private_share_2` (generated from step 3) into it.
6. On the third cosigner, create a file `~/.horcrux/share.json`, and paste the contents of `private_share_3` (generated from step 3) into it.
7. Clean up the files on the first cosigner:

   ```shell
   rm priv_validator_key.json private_share_*.json
   ```

8. Stop the validator:

   ```shell
   sudo systemctl stop cosmovisor
   ```

9. On the server for the validator, copy the contents of `$DAEMON_HOME/data/priv_validator_state.json` which looks like:

   ```json
   {
     "height": "361402",
     "round": 0,
     "step": 3,
     "signature": "IEOS7EJ8C6ZZxwwXiGeMhoO8mwtgTiq6VPR/F1cpLZuz0ZvUZdsgQjTt0GniAIgosfEjC5izKw4Nvvs3ZIceAw==",
     "signbytes": "6B080211BA8305000000000022480A205D4E1F722F53A3FD9E0D28639D7CE7B588338570EBA5C340687C30609C47BCA41224080112208283B6E16BEA46797F8AD4EE0ACE424AC7A4827202446B2D56E7F4438541B7BD2A0C08E4ACE28B0610CCD0AC830232066A756E6F2D31"
   }
   ```

10. Perform this step on each cosigner.

    Create the files `~/.horcrux/state/$CHAIN_ID_priv_validator_state.json` and `~/.horcrux/state/$CHAIN_ID_share_sign_state.json`. Both of these files will contain the same contents.

    The contents will be a modified version of the contents you copied in the previous step. The modifications are making the `"round"` value a string, removing the `"signature"` key, and removing the `"signbytes"` key. The modified contents will look like:

    ```json
    {
      "height": "361402",
      "round": "0",
      "step": 3
    }
    ```

11. Follow this step on each cosigner. Start Horcrux:

    ```shell
    sudo systemctl start horcrux
    journalctl -fu horcrux
    ```

    It's expected to see the `Retrying` log get printed in an infinite loop since we haven't finished setting up Horcrux yet. However, if an error occurs, then one of the previous steps have been incorrectly performed.

12. Follow this step on each sentry. Connect to the respective cosigner:

    ```shell
    sed 's|priv_validator_laddr = ""|priv_validator_laddr = "tcp://0.0.0.0:1234"|' -i $DAEMON_HOME/config/config.toml
    sudo systemctl restart cosmovisor
    ```

13. Verify that the Horcrux setup was successful:

    1. Check the logs on each cosigner to ensure that they're connected to their respective sentries:

       ```shell
       journalctl -fu horcrux
       ```

    2. Check that your validator is signing blocks:

       ```shell
       $DAEMON_NAME query slashing signing-info ($DAEMON_NAME tendermint show-validator)
       ```
