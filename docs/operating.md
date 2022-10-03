# Operating the Node

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
osmosisd query staking validators --limit 300 -o json \
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

### Available Upgrades for Juno Mainnet Archive Nodes

After installing the first version (i.e., 3.0.0), the following versions must be installed at their respective block heights (certain versions such as 7.0.0 must be skipped due to bugs, and are therefore left out):

|Version|Block height|
|:---:|:---:|
|3.1.1|2,616,300|
|4.0.0|2,951,100|
|5.0.1|3,035,000|
|6.0.0|3,159,650|
|8.0.0|3,851,750|

Since this document may be outdated, please additionally check [this](https://docs.junonetwork.io/validators/mainnet-upgrades) page for versions and block heights. Only check the version tags (e.g., `v8.0.0`), and their respective block heights on the page because the upgradation instructions are confusing. Instead, use the [upgradation](#upgrading) section of this document.

### Upgrading

This section explains how to upgrade a node.

```shell
cd
read -P "Enter the directory the node's GitHub repo was downloaded to relative to $HOME (e.g., sei-chain): " DIR
cd $DIR
git fetch --tags

set PROMPT 'Enter the upgrade\'s git tag (e.g., v3.1.1), or commit hash (e.g.,'
set PROMPT "$PROMPT 4ec1b0ca818561cef04f8e6df84069b14399590e): "
read -P $PROMPT VERSION

git checkout $VERSION
make install

if test ($DAEMON_NAME version) = $VERSION
    printf "Successfully installed $VERSION\n"
else
    printf "Failed to install $VERSION\n"
    exit 1
end

mkdir -p $DAEMON_HOME/cosmovisor/upgrades/$VERSION/bin
cp $HOME/go/bin/$DAEMON_NAME $DAEMON_HOME/cosmovisor/upgrades/$VERSION/bin

if test ($DAEMON_HOME/cosmovisor/upgrades/$VERSION/bin/$DAEMON_NAME version) = $VERSION
    printf 'Successfully scheduled upgrade\n'
else
    printf 'Failed to schedule upgrade\n'
end
```
