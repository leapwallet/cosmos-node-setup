# Key

```shell
read -P 'A key is named whatever you want. Enter they key: ' KEY

# The KEY variable is used in a future command. Before that command, you might exit this terminal session to replace
# the validator node's private key file. Therefore, we save the variable for future terminal sessions.
printf "set KEY $KEY\n" >> ~/.config/fish/config.fish

set PROMPT 'Enter y if you want to create and use a new wallet for the validator node. Enter n if you want to use an'
set PROMPT "$PROMPT existing wallet for the validator node: "
read -P $PROMPT IS_NEW_ADDRESS

if test $IS_NEW_ADDRESS = 'y'
    $DAEMON_NAME keys add $KEY

    printf 'In case you need to recreate this validator (e.g., the hardware it was running on got destroyed), '
    printf "entering the same mnemonic will not generate the same $DAEMON_HOME/config/priv_validator_key.json and "
    printf "$DAEMON_HOME/config/node_key.json. Therefore, you must back up $DAEMON_HOME/config/priv_validator_key.json "
    printf "and $DAEMON_HOME/config/node_key.json in case you need to restore your validator later on. Preferably, "
    printf 'encrypt the backup.\n'
else
    $DAEMON_NAME keys add $KEY --recover

    printf "If you're recreating a validator, and have a backup of $DAEMON_HOME/config/priv_validator_key.json and ""
    printf "$DAEMON_HOME/config/node_key.json, replace the generated files at the same path with the backup.\n""
end

printf 'Note down the value of the address field printed (e.g., sei1q0rnfjenxl5l8pnkv2rwc4avzeav56mq72qmj2) as it\'ll '
printf 'be required in a future step.\n'
```
