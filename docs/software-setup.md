# Software Setup

Since the commands are for Ubuntu, you might have to modify them if you're using a different OS.

## Node Setup

Follow these steps to set up a Juno mainnet archive node, Sei testnet validator node, Stride testnet validator node, Osmosis testnet validator node, or Commercio.network testnet validator node:
1. If you're running as the `root` user, and don't have another user that you can switch to which has `sudo` privileges, then follow these steps:
    1. Create the user:

        ```shell
        read -p 'Enter the user: ' USER
        adduser $USER
        usermod -aG sudo $USER
        exit
        ```
    2. SSH back into the server using the new user.
2. If you're using AWS, set the password:

    ```shell
    sudo passwd ubuntu
    ```
3. Prepare for installation:

    ```shell
    #########################
    ## BEGIN: Install fish ##
    #########################
   
    sudo apt-add-repository -y ppa:fish-shell/release-3
    sudo apt update
    sudo apt -y install fish
    chsh -s /usr/bin/fish

    #######################
    ## END: Install fish ##
    #######################
   
    sudo apt -y full-upgrade
    
    sudo reboot
    ```
4. Install the toolchain:

    ```shell
    sudo apt -y install make build-essential gcc git jq chrony lz4
   
    #######################
    ## BEGIN: Install Go ##
    #######################
   
    wget https://golang.org/dl/go1.18.2.linux-amd64.tar.gz
    sudo tar xzvf go1.18.2.linux-amd64.tar.gz -C /usr/local
    rm go1.18.2.linux-amd64.tar.gz
   
    #####################
    ## END: Install Go ##
    #####################
   
    read -P 'Examples of blockchain names are osmosis, sei, and stride. Enter the blockchain\'s name: ' BLOCKCHAIN
    read -P 'A moniker is a name of your choosing for your node. Enter the moniker: ' MONIKER

    ######################################
    ## BEGIN: Set environment variables ##
    ###################################### 
   
    if test $BLOCKCHAIN = 'osmosis'
        set DAEMON_HOME ~/.osmosisd
    else
        set DAEMON_HOME ~/.$BLOCKCHAIN
    end
   
    printf "\
   
    set MONIKER $MONIKER
    
    # Go
    set -x GOROOT /usr/local/go
    set -x GOPATH ~/go
    set -x GO111MODULE on
    set PATH /usr/local/go/bin ~/go/bin $PATH

    # Cosmovisor
    set -x DAEMON_NAME $(printf $BLOCKCHAIN)d
    set -x DAEMON_HOME $DAEMON_HOME
    " >> ~/.config/fish/config.fish
   
    source ~/.config/fish/config.fish
      
    ####################################
    ## END: Set environment variables ##
    #################################### 
    ```
5. This optional but recommended step sets up Fail2ban:

    ```shell
    sudo apt -y install fail2ban sendmail
    sudo systemctl enable --now fail2ban
    sudo reboot
    ```
6. Set up one of the following nodes:
    - [Juno mainnet archive node](blockchains/juno.md)
    - [Sei testnet validator node](blockchains/sei.md)
    - [Stride testnet validator node](blockchains/stride.md)
    - [Osmosis testnet validator node](osmosis.md)
    - [Commercio.network testnet validator node](blockchains/commercio-network.md)
7. Follow this step if you want to disable the REST, gRPC, and gRPC Web APIs (recommended for validator nodes):

    ```shell
    sed 's|enable = true|enable = false|' -i $DAEMON_HOME/config/app.toml
   
    printf 'If you\'re running a validator, and are going to use PANIC, then the REST API must be enabled as PANIC '
    printf 'can\'t monitor without it. Don\'t worry about security in this particular case because the REST API will '
    printf 'only be accessible by the server it\'s running on (we\'ll whitelist the server\'s IP address later on).\n'
    read -P 'Enter y if you\'re running a validator, and are going to use PANIC, and n otherwise: ' WILL_USE_PANIC
    if test $WILL_USE_PANIC = 'y'
        sed '0,/enable = false/{s|enable = false|enable = true|}' -i $DAEMON_HOME/config/app.toml
    end
   
    sudo systemctl restart cosmovisor
    ```

8. Follow this step if you want to prune the node (recommended for validator nodes):

    ```shell
    sed \
        -e 's|pruning = .*|pruning = "custom"|' \
        -e 's|pruning-keep-recent = .*|pruning-keep-recent = "100"|' \
        -e 's|pruning-keep-every = .*|pruning-keep-every = "0"|' \
        -e 's|pruning-interval = .*|pruning-interval = "10"|' \
        -i $DAEMON_HOME/config/app.toml
    sed 's|indexer = .*|indexer = "null"|' -i $DAEMON_HOME/config/config.toml
    sudo systemctl restart cosmovisor
    ```
9. This optional but recommended step improves your node's performance and security:

    ```shell
    sed \
        -e 's|max_num_inbound_peers = .*|max_num_inbound_peers = 100|' \
        -e 's|max_num_outbound_peers = .*|max_num_outbound_peers = 10|' \
        -e 's|flush_throttle_timeout = .*|flush_throttle_timeout = "100ms"|' \
        -i $DAEMON_HOME/config/config.toml
    sudo systemctl restart cosmovisor
    ```

## URL Setup

This optional but recommended section explains how to set up the DNS and TLS certificate. We explain it by using Cloudflare for the DNS, and Caddy for the TLS certificate. Of course, you can use different tools such as Google Domains and Traefik instead.

This section explains how to secure the REST API in case you're running a validator node monitored by PANIC, and have enabled the REST API as a result. If you're not going to follow this section, then be sure to prevent IP addresses other than the server itself from accessing the REST API.

### DNS

This section explains how to use your domain name (e.g., example.com) instead of your server's IP address. It assumes that you already have a domain name, and a Cloudflare setup.
1. Go to your website's DNS section on Cloudflare.
2. Click **Add record**.
3. Set the **Type** field to **A**.
4. Set the **Name (required)** field (e.g., **osmo-test-4**).
5. Set the **IPv4 address (required)** field to your server's IP address.
6. Set the **Proxy status** to **DNS only**.
7. Set the **TTL** to **Auto**.

### TLS Certificate

This section explains how to set up the TLS certificate, and URLs for each API that you want to expose. There's a reference to Prometheus which is a monitoring system that you have the option to set up in the next section.

```shell
##########################
## BEGIN: Install Caddy ##
##########################

sudo apt -y install debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf https://dl.cloudsmith.io/public/caddy/stable/gpg.key | \
    sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt | \
    sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt -y install caddy

########################
## END: Install Caddy ##
########################

############################
## BEGIN: Configure Caddy ##
############################

read -P 'Enter the domain such as localhost or osmo-test-4.example.com: ' DOMAIN

read -P 'Enter y if you\'re running a validator, and using PANIC, and n otherwise: ' MUST_WHITELIST
if test $MUST_WHITELIST = 'y'
    set IP_ADDRESS (curl -s https://icanhazip.com/)
    set WHITELIST "\n        @denied not remote_ip $IP_ADDRESS\n        abort @denied"
else
    set WHITELIST ''
end

printf "\
$DOMAIN {
    handle_path /tendermint-rpc/* {
        rewrite * {path}
        reverse_proxy :26657
    }
    handle_path /rest-api/* {
        rewrite * {path}
        reverse_proxy :1317$WHITELIST
    }
    handle_path /grpc/* {
        rewrite * {path}
        reverse_proxy :9090
    }
    handle_path /grpc-web/* {
        rewrite * {path}
        reverse_proxy :9091
    }
    handle_path /prometheus/* {
        rewrite * {path}
        reverse_proxy :6666
    }
    handle_path /node-exporter/* {
        rewrite * {path}
        reverse_proxy :9100
    }
    handle_path /blockchain-node/* {
        rewrite * {path}
        reverse_proxy :26660
    }
}
" | sudo tee /etc/caddy/Caddyfile

sudo systemctl reload caddy

##########################
## END: Configure Caddy ##
##########################
```

The following URLs will now be available, where `<DOMAIN>` is the same as the domain you entered during the prompt. Services which aren't enabled won't exist. For example, `https://<DOMAIN>/rest-api` won't work if the node's REST API is disabled.
- RPC API: `https://<DOMAIN>/tendermint-rpc`
- REST API: `https://<DOMAIN>/rest-api`
- gRPC: `https://<DOMAIN>/grpc`
- gRPC Web: `https://<DOMAIN>/grpc-web`
- Prometheus's metrics (for use by Grafana): `https://<DOMAIN>/prometheus`
- Node Exporter's metrics (for use by PANIC): `https://<DOMAIN>/node-exporter`
- Blockchain node's metrics (for use by PANIC): `https://<DOMAIN>/blockchain-node`

For example (assuming that you're running a Juno mainnet archive node with the REST API enabled), you can now query a tx using the REST API base URL of `https://<DOMAIN>/rest-api` by opening `https://<DOMAIN>/rest-api/cosmos/tx/v1beta1/txs/8E9623B92C4501432EFDE993E6077B1FD021613CE1980859A1B4F0BB374BC1A9` in a browser.

Note that if you're using PANIC, then you'll need to access it via `https://<IP>:3333` and `https://<IP>:8000` (where `<IP>` is the server's IP address) rather than through the reverse proxy.

## Monitoring and Alerting Setup

This optional but recommended section explains how to set up monitoring and alerting. We explain it by installing Prometheus directly on the server, connecting it to Grafana Cloud, and using PANIC. Of course, you can run Grafana yourself, use an external Prometheus server, use [Tenderduty](https://github.com/blockpane/tenderduty), etc. instead.

1. Sign up for [Grafana Cloud](https://grafana.com/auth/sign-up/create-user).
2. Set up Node Exporter:

    ```shell
    cd
   
    ##################################
    ## BEGIN: Install Node Exporter ##
    ##################################
   
    set PROMPT 'You\'ll be prompted to enter a URL. The URL is the relevant download link from'
    set PROMPT "$PROMPT[1]https://prometheus.io/download/#node_exporter (e.g., https://github.com/prometheus/"
    set PROMPT "$PROMPT[1]node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz). Enter the"
    set PROMPT "$PROMPT URL: "
    read -P $PROMPT URL
   
    wget $URL
    set FILE (printf $URL | sed 's|.*/||')
    tar xvzf $FILE
    rm $FILE
    
    ################################
    ## END: Install Node Exporter ##
    ################################
      
    ##################################################
    ## BEGIN: Set up systemd unit for Node Exporter ##
    ##################################################
   
    set DIR (printf $FILE | sed 's|.tar.gz||')
   
    printf "\
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
    " | sudo tee /etc/systemd/system/node-exporter.service
   
    sudo systemctl enable --now node-exporter
    sudo systemctl status node-exporter
   
    ################################################
    ## END: Set up systemd unit for Node Exporter ##
    ################################################
    ```
3. On your Grafana Cloud instance, create an [API key](https://grafana.com/docs/grafana-cloud/reference/create-api-key/) for the Prometheus integration with the **Role** set to **MetricsPublisher**. Note down the URL, username, and password for use in the next step.
4. Install Prometheus:

    ```shell
    cd
   
    ##############################
    ## BEGIN: Enable Prometheus ##
    ##############################
   
    sed 's|prometheus = .*|prometheus = true|' -i $DAEMON_HOME/config/config.toml
    sudo systemctl restart cosmovisor
    
    ############################
    ## END: Enable Prometheus ##
    ############################
   
    ###############################
    ## BEGIN: Install Prometheus ##
    ###############################
   
    set PROMPT 'You\'ll be prompted to enter a Prometheus download link. The Prometheus download link is the relevant '
    set PROMPT "$PROMPT URL from https://prometheus.io/download/#prometheus) (e.g., https://github.com/prometheus/"
    set PROMPT "$PROMPT[1]prometheus/releases/download/v2.37.0/prometheus-2.37.0.linux-amd64.tar.gz). Enter the "
    set PROMPT "$PROMPT Prometheus download link: "
    read -P $PROMPT PROM_URL
   
    wget $PROM_URL
    set FILE (printf $PROM_URL | sed 's|.*/||')
    tar xvzf $FILE
    rm $FILE

    #############################
    ## END: Install Prometheus ##
    #############################
   
    set DIR (printf $FILE | sed 's|.tar.gz||')
   
    set PROMPT 'Enter your Grafana Cloud URL (e.g.,'
    set PROMPT "$PROMPT https://prometheus-prod-10-prod-us-central-0.grafana.net/api/prom/push): "
    read -P $PROMPT GRAFANA_URL
   
    read -P 'Enter your Grafana username (e.g., 986969): ' GRAFANA_USERNAME
   
    set PROMPT 'Enter your Grafana password (e.g.,'
    set PROMPT "$PROMPT oop09ikiM2YxZDJjOGY5YmJlODUxMmNpoiuyt2IzZDI3YWFlNzQyZGE2ZDdjYiIsIm4iOiJzZWktdGVzdG5ldC12Ykj3): "
    read -P $PROMPT GRAFANA_PASSWORD
   
    #################################
    ## BEGIN: Configure Prometheus ##
    #################################
   
    printf "\
    scrape_configs:
      - job_name: node-exporter
        static_configs:
          - targets: ['localhost:9100']
      - job_name: $DAEMON_NAME
        static_configs:
          - targets: ['localhost:26660']
    remote_write:
      - url: $GRAFANA_URL
        basic_auth:
          username: $GRAFANA_USERNAME
          password: $GRAFANA_PASSWORD
    " > $DIR/prometheus.yml
   
    ###############################
    ## END: Configure Prometheus ##
    ###############################
   
    ###############################################
    ## BEGIN: Set up systemd unit for Prometheus ##
    ###############################################
   
    printf "\
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
    " | sudo tee /etc/systemd/system/prometheus.service
   
    sudo systemctl enable --now prometheus
    sudo systemctl status prometheus
   
    #############################################
    ## END: Set up systemd unit for Prometheus ##
    #############################################
    ```
5. Set up [PANIC](https://github.com/SimplyVC/panic).
