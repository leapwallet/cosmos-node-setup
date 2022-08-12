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
3. Set up the node:
    - [Juno mainnet archive node](juno.md)
    - [Sei testnet validator node](sei.md)

## Monitoring Setup

This optional but recommended section explains how to set up metrics. We explain it by installing Prometheus directly on the server, and connecting it to Grafana Cloud. Of course, you can run Grafana yourself, use an external Prometheus server, etc. instead.

1. Sign up for [Grafana Cloud](https://grafana.com/auth/sign-up/create-user).
2. Set up Node Exporter:

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
3. Set up Prometheus:
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
