# Blockchain Node Operations

This document explains common operations you may want to perform on blockchain nodes such as how to upgrade them.

## Sync Status

```shell
curl -s http://localhost:26657/status | jq .result.sync_info.catching_up
```

Prints `true` if the node is still downloading the older blocks, and `false` otherwise.

## Genesis Error

The node might repeatedly crash with the following error message:

```
Error: error during handshake: error on replay: validator set is nil in genesis and still empty after InitChain
```

In this case, you must run `$DAEMON_NAME unsafe-reset-all` (such as for `junod`) or `$DAEMON_NAME tendermint --home $DAEMON_HOME unsafe-reset-all` (such as for `seid`) which will wipe the DB. This error only has the potential to surface when initially setting up the node, and not after the node has been downloading blocks, upgrading, etc.

## Track Validator

To check if the validator is in the active set, run the following command. If the bond status is `BOND_STATUS_BONDED`, then the validator is part of the active set.

```shell
$DAEMON_NAME query staking validators --limit 300 -o json \
    | jq -r '.validators[] | [.operator_address, .status, (.tokens|tonumber / pow(10; 6)),.commission.update_time[0:19], .description.moniker] | @csv' \
    | column -t -s"," \
    | grep $MONIKER
```

If the validator is in the active set, then you can track its signing history:

```shell
$DAEMON_NAME query slashing signing-info ($DAEMON_NAME tendermint show-validator)
```

## Upgrades

Remember to subscribe to new upgrade notifications. For example, you should join Juno's [#⚡ | validator-lounge](https://discord.com/channels/816256689078403103/816263136491339867) Discord channel where they try to announce new upgrades a few days in advance if you're running a Juno node.

### Available Upgrades for Juno `juno-1` Archive Nodes

After installing the first version (i.e., 3.0.0), the following versions must be installed at their respective block heights (certain versions such as 7.0.0 must be skipped due to bugs, and are therefore left out):

| Version | Block height |
| :-----: | :----------: |
|  3.1.1  |  2,616,300   |
|  4.0.0  |  2,951,100   |
|  5.0.1  |  3,035,000   |
|  6.0.0  |  3,159,650   |
|  8.0.0  |  3,851,750   |

### Upgrading

There are two types of upgrades - planned and missed. An example of a planned upgrade is if the node is on v12, and v13 must be installed tomorrow. An example of a missed upgrade is if the node is on v12, and v13 was supposed to be installed in the past. An error that looks like the following will gets printed if an upgrade was missed:

```shell
Dec 20 05:41:59 ip-10-0-13-196 cosmovisor[1278885]: 5:41AM ERR  error="binary not present, downloading disabled: cannot stat dir /home/ubuntu/.osmosisd/cosmovisor/upgrades/v13/bin/osmosisd: stat /home/ubuntu/.osmosisd/cosmovisor/upgrades/v13/bin/osmosisd: no such file or directory" module=cosmovisor
Dec 20 05:41:59 ip-10-0-13-196 systemd[1]: cosmovisor.service: Main process exited, code=exited, status=1/FAILURE
Dec 20 05:41:59 ip-10-0-13-196 systemd[1]: cosmovisor.service: Failed with result 'exit-code'.
```

```shell
# We can't write code like <if test ($DAEMON_NAME version) = $VERSION> because <$DAEMON_NAME version> outputs to stdout
# even if you redirect the output, etc.

cd
read -P "Enter the directory the node's GitHub repo was downloaded to relative to $HOME such as sei-chain: " DIR
cd $DIR
git fetch --tags

set PROMPT 'Enter the upgrade\'s git tag such as v13.0.0, or commit hash such as'
set PROMPT "$PROMPT 4ec1b0ca818561cef04f8e6df84069b14399590e): "
read -P $PROMPT VERSION

git checkout $VERSION
make install

printf 'Installed v'
$DAEMON_NAME version
read -P "Enter y if the version matches $VERSION, and n otherwise: " IS_SUCCESSFUL

if test $IS_SUCCESSFUL = 'y'
    printf "Successfully installed $VERSION\n"
    
    read -P 'Enter p if this is a planned upgrade, and m if this is a missed upgrade: ' TYPE
    if test $TYPE = 'p'
        mkdir -p $DAEMON_HOME/cosmovisor/upgrades/$VERSION/bin
        cp $HOME/go/bin/$DAEMON_NAME $DAEMON_HOME/cosmovisor/upgrades/$VERSION/bin
    else
        set PROMPT 'The error printed regarding the missed upgrade will specify the name of the directory it expects'
        set PROMPT "$PROMPT the binary to get saved to. For example, if it printed "
        set PROMPT "$PROMPT /home/ubuntu/.osmosisd/cosmovisor/upgrades/v13/bin/osmosisd, then the directory's name is"
        set PROMPT "$PROMPT v13 (even if the version you're installing is v13.0.0). Enter the directory's name: "
        read -P $PROMPT DIR
        
        mkdir -p $DAEMON_HOME/cosmovisor/upgrades/$DIR/bin
        cp $HOME/go/bin/$DAEMON_NAME $DAEMON_HOME/cosmovisor/upgrades/$DIR/bin
    end
    
    printf 'Successfully set up upgrade for v'
    $DAEMON_HOME/cosmovisor/upgrades/$VERSION/bin/$DAEMON_NAME version
    printf "The upgrade succeeded if the above version matches $VERSION.\n"
else
    printf "Failed to install $VERSION\n"
end
```
