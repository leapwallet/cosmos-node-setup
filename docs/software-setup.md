# Software Setup

Since the commands are for Ubuntu, you might have to modify them if you're using a different OS.

## Node Setup

Follow these steps to set up a Juno mainnet archive node, or Sei testnet validator node:
1. Prepare for installation:

    ```shell
    cd
    sudo apt update
    sudo apt -y full-upgrade
    sudo reboot
    ```

    The computer will reboot.
2. Install the toolchain:

    ```shell
    sudo apt install make build-essential gcc git jq chrony lz4 -y
    wget https://golang.org/dl/go1.18.2.linux-amd64.tar.gz
    sudo tar xzvf go1.18.2.linux-amd64.tar.gz -C /usr/local
    rm go1.18.2.linux-amd64.tar.gz
    tee -a $HOME/.profile << EOF
    export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export GO111MODULE=on
    export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
    EOF
    source $HOME/.profile
    ```
3. Follow this step if you're setting up a Juno mainnet archive node:
    1. Install Juno:

        ```shell
        git clone https://github.com/CosmosContracts/juno
        cd juno
        read -p 'Enter the tag: ' TAG
        git checkout $TAG
        make install
        CHAIN_REPO="https://raw.githubusercontent.com/CosmosContracts/mainnet/main/juno-1"
        PEERS="$(curl -sL "$CHAIN_REPO/persistent_peers.txt")"
        read -p $'A moniker is the node\'s moniker which is a name of your choosing. Enter the moniker: ' MONIKER
        junod init $MONIKER --chain-id juno-1
        curl https://share.blockpane.com/juno/phoenix/genesis.json > $HOME/.juno/config/genesis.json
        sed "s|persistent_peers = .*|persistent_peers = \"$PEERS\"|" -i $HOME/.juno/config/config.toml
        read -p 'Enter the height: ' HEIGHT
        sed 's|halt-height = 0|halt-height = $HEIGHT|' -i $HOME/.juno/config/app.toml
        ```

        You'll be prompted to enter a tag. If you're going to download a snapshot, then enter the version's tag (tags can be found [here](https://github.com/CosmosContracts/juno/tags)) that the snapshot supports. Otherwise, enter the first version's tag (i.e., `v3.0.0`).

        You'll be prompted to enter a height. The height is the block height to halt at (e.g., `2616300` if you downloaded the first version of the node, `0` if you installed the latest version of the node).
    2. Follow this step if you want to enable the REST API. In the `[api]` section of `$HOME/.juno/config/app.toml`, set the `enable` key's value to `true`.
    3. Set up the Juno systemd unit:
    
        ```shell
        sudo tee /etc/systemd/system/junod.service << EOF
        [Unit]
        Description=Juno Daemon
        Wants=network-online.target
        After=network-online.target

        [Service]
        User=$USER
        ExecStart=$(which junod) start --x-crisis-skip-assert-invariants
        RestartSec=3
        Restart=always
        LimitNOFILE=4096

        [Install]
        WantedBy=multi-user.target
        EOF
        sudo systemctl enable --now junod
        sudo systemctl status junod
        ```

        The unit's status will be displayed for you to check whether it was successfully set up.
4. Follow this step if you're setting up a Sei testnet validator node:
    1. Install Sei:

        ```shell
        git clone https://github.com/sei-protocol/sei-chain.git
        cd sei-chain
        read -p 'Enter the tag: ' TAG
        git checkout $TAG
        make install
        seid version
        read -p $'A moniker is the node\'s moniker which is a name of your choosing. Enter the moniker: ' MONIKER
        seid init $MONIKER --chain-id atlantic-1 -o
        curl https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/genesis.json > \
            $HOME/.sei/config/genesis.json
        curl https://raw.githubusercontent.com/sei-protocol/testnet/master/sei-incentivized-testnet/addrbook.json > \
            $HOME/.sei/config/addrbook.json
        sed 's|minimum-gas-prices = .*|minimum-gas-prices = "0.01usei"|' -i $HOME/.sei/config/app.toml
        read -p 'A key is named whatever you want. Enter they key: ' KEY
        ```
       
        You'll be prompted to enter a tag. The tag is the most recent [tag](https://github.com/sei-protocol/sei-chain/tags) (e.g., `1.1.0beta`).
    2. If you want to make and use a new Sei wallet for the validator node, then follow this step:

        ```shell
        seid keys add $KEY
        ```
    3. If you want to use an existing Sei wallet for the validator node, then follow this step:

        ```shell
        seid keys add $KEY --recover
        ```
    4. Configure Sei:

        ```shell
        read -p 'Enter the address: ' ADDRESS
        seid add-genesis-account $ADDRESS 100000000000000000000usei
        seid gentx $KEY 70000000000000000000usei --chain-id sei-chain
        seid collect-gentxs
        cat $HOME/.sei/config/genesis.json | \
            jq '.app_state["crisis"]["constant_fee"]["denom"]="usei"' > \
            $HOME/.sei/config/tmp_genesis.json && \
            mv $HOME/.sei/config/tmp_genesis.json $HOME/.sei/config/genesis.json
        cat $HOME/.sei/config/genesis.json | \
            jq '.app_state["gov"]["deposit_params"]["min_deposit"][0]["denom"]="usei"' > \
            $HOME/.sei/config/tmp_genesis.json && \
            mv $HOME/.sei/config/tmp_genesis.json $HOME/.sei/config/genesis.json
        cat $HOME/.sei/config/genesis.json | \
            jq '.app_state["mint"]["params"]["mint_denom"]="usei"' > \
            $HOME/.sei/config/tmp_genesis.json && \
            mv $HOME/.sei/config/tmp_genesis.json $HOME/.sei/config/genesis.json
        cat $HOME/.sei/config/genesis.json | \
            jq '.app_state["staking"]["params"]["bond_denom"]="usei"' > \
            $HOME/.sei/config/tmp_genesis.json && \
            mv $HOME/.sei/config/tmp_genesis.json $HOME/.sei/config/genesis.json
        seid tendermint --home $HOME/.sei unsafe-reset-all --keep-addr-book
        rm -rf $HOME/.sei/data $HOME/.sei/wasm
        SNAP_NAME=$( \
            curl -s https://snapshots1-testnet.nodejumper.io/sei-testnet/ | \
            grep -Eo '>atlantic-1.*\.tar.lz4' | \
            tr -d '>' \
        )
        curl https://snapshots1-testnet.nodejumper.io/sei-testnet/$SNAP_NAME | lz4 -dc - | tar xf - -C $HOME/.sei
        sudo tee /etc/systemd/system/seid.service << EOF
        [Unit]
        Description=Sei Daemon
        Wants=network-online.target
        After=network-online.target

        [Service]
        Type=simple
        User=$USER
        ExecStart=$(which seid) start
        RestartSec=3
        Restart=always
        LimitNOFILE=65535
        LimitMEMLOCK=209715200

        [Install]
        WantedBy=multi-user.target
        EOF
        sudo systemctl enable --now seid
        sudo systemctl status seid
        ```

        You'll be prompted to enter an address. The address is the value of the `address` key (e.g., `sei19vf5mfr40awvkefw69nl6p3mmlsnacmm8thjxk`) printed in the previous step.

        The unit's status will be displayed for you to check whether it was successfully set up.
    5. Get some Sei testnet tokens from the [🚰 | atlantic-1-faucet](https://discord.com/channels/973057323805311026/979272741150687262) Discord channel.
    6. Create the validator:

        ```shell
        seid tx staking create-validator \
            --amount=900000usei \
            --pubkey=$(seid tendermint show-validator) \
            --moniker="$MONIKER" \
            --chain-id=atlantic-1 \
            --from="$KEY" \
            --commission-rate=0.10 \
            --commission-max-rate=0.20 \
            --commission-max-change-rate=0.01 \
            --min-self-delegation=1 \
            --fees=2000usei
        ```

        Open `https://sei.explorers.guru/transaction/<HASH>`, where `<HASH>` is the value of the `txhash` field printed (e.g., `FDDE67944FBD6111EA9898D6F8B5CF9601B4935B5A17CE18209825311A036210`) in your browser to check if the validator was successfully created.
    7. Back up an encrypted copy of `$HOME/.sei/config/priv_validator_key.json` and `$HOME/.sei/config/node_key.json` in case you need to restore your validator later on.

## Monitoring Setup

This optional but recommended section explains how to set up metrics. We explain it by installing Prometheus directly on the server, and connecting it to Grafana Cloud. Of course, you can run Grafana yourself, use an external Prometheus server, etc. instead.

### Grafana Cloud

This section explains how to visualize the metrics.
1. Sign up for [Grafana Cloud](https://grafana.com/auth/sign-up/create-user).

### Node Exporter

This section explains how to set up the hardware metrics.


```shell
cd
read -p 'Enter the URL: ' URL
wget $URL
FILE=$(echo $URL | sed 's|.*/||')
tar xvzf $FILE
rm $FILE
DIR=$(echo $FILE | sed 's|.tar.gz||')
sudo tee /etc/systemd/system/node-exporter.service << EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=$USER
ExecStart=/home/$USER/$DIR/node_exporter
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl enable --now node-exporter
sudo systemctl status node-exporter
```

You'll be prompted to enter a URL. The URL is the relevant download [link](https://prometheus.io/download/#node_exporter) (e.g., `https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz`).

The systemd unit's status will be displayed for you to check whether it was successfully set up.

### Prometheus

This section explains how to aggregate the metrics from the Cosmos node, and Node Exporter.

1. On your Grafana Cloud instance, create an [API key](https://grafana.com/docs/grafana-cloud/reference/create-api-key/) with the **Role** set to **MetricsPublisher**. Note down the URL, username, and password for use in the next step.
2. Install Prometheus:

    ```shell
    cd
    read -p 'Enter the <CHAIN_DIR> as a relative path (e.g., .sei): ' CHAIN_DIR
    sed 's|prometheus = .*|prometheus = true|' -i $CHAIN_DIR/config/config.toml
    read -p 'Enter the Prometheus download link: ' PROM_URL
    wget $PROM_URL
    FILE=$(echo $PROM_URL | sed 's|.*/||')
    tar xvzf $FILE
    rm $FILE
    DIR=$(echo $FILE | sed 's|.tar.gz||')
    read -p 'Enter the <BINARY>: ' BINARY
    read -p 'Enter your Grafana Cloud URL (e.g., https://prometheus-prod-10-prod-us-central-0.grafana.net/api/prom/push): ' GRAFANA_URL
    read -p 'Enter your Grafana username (e.g., 986969): ' GRAFANA_USERNAME
    read -p 'Enter your Grafana password (e.g., oop09ikiM2YxZDJjOGY5YmJlODUxMmNpoiuyt2IzZDI3YWFlNzQyZGE2ZDdjYiIsIm4iOiJzZWktdGVzdG5ldC12YWxpZGF0b3Itbm9kZSIsImlkIjo2ODi98ujK): ' GRAFANA_PASSWORD
    tee $DIR/prometheus.yml << EOF
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: node-exporter
        static_configs:
          - targets: ['localhost:9100']
      - job_name: $BINARY
        static_configs:
          - targets: ['localhost:26660']
    remote_write:
      - url: $GRAFANA_URL
        basic_auth:
          username: $GRAFANA_USERNAME
          password: $GRAFANA_PASSWORD
    EOF
    sudo tee /etc/systemd/system/prometheus.service << EOF
    [Unit]
    Description=Prometheus
    Wants=network-online.target
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=/home/$USER/$DIR/prometheus --config.file=/home/$USER/$DIR/prometheus.yml --web.listen-address=:6666 --storage.tsdb.path=/home/$USER/$DIR/data
    Restart=always
    RestartSec=3
    LimitNOFILE=4096

    [Install]
    WantedBy=multi-user.target
    EOF
    sudo systemctl enable --now prometheus
    sudo systemctl status prometheus
    ```

    You'll be prompted to enter a Prometheus download link. The Prometheus download link is the relevant [URL](https://prometheus.io/download/#prometheus) (e.g., `https://github.com/prometheus/prometheus/releases/download/v2.37.0/prometheus-2.37.0.linux-amd64.tar.gz`).

    The systemd unit's status will be displayed for you to check whether it was successfully set up.

## URL Setup

This optional but recommended section explains how to set up the DNS and TLS certificate. We explain it by using Cloudflare for the DNS, and Caddy for the TLS certificate. Of course, you can use different tools such as Google Domains and Traefik instead.

### DNS

This section explains how to use your domain name (e.g., example.com) instead of your server's IP address. It assumes that you already have a domain name, and a Cloudflare setup.
1. Go to your website's DNS section on Cloudflare.
2. Click **Add record**.
3. Set the **Type** field to **A**.
4. Set the **Name (required)** field (e.g., **juno-mainnet-archive-node**).
5. Set the **IPv4 address (required)** field to your server's IP address.
6. Set the **Proxy status** to **DNS only**.
7. Set the **TTL** to **Auto**.

### TLS Certificate

This section explains how to set up the TLS certificate, and URLs for each API you want to expose.

```shell
sudo apt -y install debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf https://dl.cloudsmith.io/public/caddy/stable/gpg.key | \
    sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt | \
    sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
read -p 'Enter the domain (e.g., juno.example.com): ' DOMAIN
sudo tee /etc/caddy/Caddyfile << EOF
$DOMAIN {
    handle_path /rest-api/* {
        rewrite * {path}
        reverse_proxy :1317
    }
    handle_path /grpc/* {
        rewrite * {path}
        reverse_proxy :9090
    }
    handle_path /grpc-web/* {
        rewrite * {path}
        reverse_proxy :9091
    }
    handle_path /metrics/* {
        rewrite * {path}
        reverse_proxy :6666
    }
}
EOF
sudo systemctl reload caddy
```

Except for disabled APIs (e.g., `https://<DOMAIN>/rest-api` won't work if the node's REST API is disabled), the following URLs will now be available (`<DOMAIN>` is the same as the domain you entered during the prompt):
- REST API: `https://<DOMAIN>/rest-api`
- gRPC: `https://<DOMAIN>/grpc`
- gRPC Web: `https://<DOMAIN>/grpc-web`
- Metrics: `https://<DOMAIN>/metrics`

For example (assuming that the REST API is enabled, and you're running a Juno mainnet archive node), you can now query a tx using the REST API base URL of `https://<DOMAIN>/rest-api` by opening `https://<DOMAIN>/rest-api/cosmos/tx/v1beta1/txs/8E9623B92C4501432EFDE993E6077B1FD021613CE1980859A1B4F0BB374BC1A9` in a browser.
