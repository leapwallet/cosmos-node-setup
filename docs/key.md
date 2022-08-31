# Key

We recommend using a 3/5 multisig wallet. We recommend 3/5 because you don't want to permanently lose access to your funds if two people lose their keys (e.g., one dies, and the other lose their mnemonic). We recommend a multisig because you don't want one person to run away with the funds, get hacked, or accidentally make a tx.

```shell
read -P 'A key is named whatever you want. Enter they key: ' KEY
printf "\nset KEY $KEY\n" >> ~/.config/fish/config.fish

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

    printf "If you're recreating a validator, and have a backup of $DAEMON_HOME/config/priv_validator_key.json and "
    printf "$DAEMON_HOME/config/node_key.json, replace the generated files at the same path with the backup.\n"
end
```
