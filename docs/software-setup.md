# Software Setup

Since the commands are for Ubuntu, you might have to modify them if you're using a different OS.

## Node Setup

Follow these steps to set up a Juno mainnet archive node, Sei testnet validator node, or Stride testnet validator node:
1. Prepare for installation:

    ```shell
    sudo passwd ubuntu
   
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
2. Install the toolchain:

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
   
    set PROMPT 'Enter the name of the node\'s executable. This is typically the chain\'s name followed by \"d\" (e.g.,'
    set PROMPT "$PROMPT junod for Juno, seid for Sei): "
    read -P $PROMPT DAEMON_NAME
   
    read -P 'Enter the name of the configuration and data directory (e.g., .juno for Juno, .sei for Sei): ' DAEMON_HOME 
   
    ######################################
    ## BEGIN: Set environment variables ##
    ###################################### 
   
    printf "\
    
    # Go
    set -x GOROOT /usr/local/go
    set -x GOPATH ~/go
    set -x GO111MODULE on
    set PATH /usr/local/go/bin ~/go/bin $PATH

    # Cosmovisor
    set -x DAEMON_NAME $DAEMON_NAME
    set -x DAEMON_HOME ~/$DAEMON_HOME
    " >> ~/.config/fish/config.fish
   
    source ~/.config/fish/config.fish
      
    ####################################
    ## END: Set environment variables ##
    #################################### 
    ```
3. Set up one of the following nodes:
    - [Juno mainnet archive node](juno.md)
    - [Sei testnet validator node](sei.md)
    - [Stride testnet validator node](stride.md)
4. Follow this step if you want to disable the REST, gRPC, and gRPC Web APIs (recommended for validator nodes):

    ```shell
    sed 's|enable = true|enable = false|' -i $DAEMON_HOME/config/app.toml
    ```
5. Follow this step if you want to prune the node (recommended for validator nodes):

    ```shell
    sed \
        -e 's|pruning = .*|pruning = "custom"|' \
        -e 's|pruning-keep-recent = .*|pruning-keep-recent = "100"|' \
        -e 's|pruning-keep-every = .*|pruning-keep-every = "0"|' \
        -e 's|pruning-interval = .*|pruning-interval = "10"|' \
        -i $DAEMON_HOME/config/app.toml
    sed 's|indexer = .*|indexer = "null"|' -i $DAEMON_HOME/config/config.toml
    ```
6. This optional but recommended step improves your node's performance and security:

    ```shell
    sed \
        -e 's|max_num_inbound_peers = .*|max_num_inbound_peers = 100|' \
        -e 's|max_num_outbound_peers = .*|max_num_outbound_peers = 10|' \
        -e 's|flush_throttle_timeout = .*|flush_throttle_timeout = "100ms"|' \
        -i $DAEMON_HOME/config/config.toml
    sudo systemctl restart $DAEMON_NAME
    ```
7. These optional but recommended steps improve your node's security:
    - Set up [2FA](https://www.digitalocean.com/community/tutorials/how-to-configure-multi-factor-authentication-on-ubuntu-18-04) for SSH.
    - Set up [Tendermint KMS](https://docs.evmos.org/validators/security/kms.html) if you're running a validator node.
    - Use [Horcrux](https://github.com/strangelove-ventures/horcrux) if you're running a validator node.
    - Set up [Unattended Upgrades](https://github.com/mvo5/unattended-upgrades).
    - Set up a NIDS (network intrusion detection system).
    - Set up a HIDS (host intrusion detection system).
    - Set up an SMTP server to get notifications from Unattended Upgrades, etc. 
    - Save logs to an archival service such as [Grafana Loki](https://grafana.com/oss/loki/) or [Papertrail](https://www.papertrail.com/).
    - Set up [PANIC](https://github.com/SimplyVC/panic).
    - Set up rate limiting.

## Monitoring Setup

This optional but recommended section explains how to set up metrics. We explain it by installing Prometheus directly on the server, and connecting it to Grafana Cloud. Of course, you can run Grafana yourself, use an external Prometheus server, etc. instead.

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
    sudo systemctl restart $DAEMON_NAME
    
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
    global:
      scrape_interval: 15s
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

read -P 'Enter the domain (e.g., juno.example.com): ' DOMAIN

printf "\
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
" | sudo tee /etc/caddy/Caddyfile

sudo systemctl reload caddy

##########################
## END: Configure Caddy ##
##########################
```

Except for disabled APIs (e.g., `https://<DOMAIN>/rest-api` won't work if the node's REST API is disabled), the following URLs will now be available (`<DOMAIN>` is the same as the domain you entered during the prompt):
- REST API: `https://<DOMAIN>/rest-api`
- gRPC: `https://<DOMAIN>/grpc`
- gRPC Web: `https://<DOMAIN>/grpc-web`
- Metrics: `https://<DOMAIN>/metrics`

For example (assuming that you're running a Juno mainnet archive node with the REST API enabled), you can now query a tx using the REST API base URL of `https://<DOMAIN>/rest-api` by opening `https://<DOMAIN>/rest-api/cosmos/tx/v1beta1/txs/8E9623B92C4501432EFDE993E6077B1FD021613CE1980859A1B4F0BB374BC1A9` in a browser.
