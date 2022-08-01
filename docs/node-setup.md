# Node Setup

Since the commands are for Ubuntu, you might have to modify them if you're using a different OS.

## Juno Setup

Here are the steps to set up Juno:
1. Go to the home directory:

    ```sh
    cd
    ```
2. Set up the dependencies:
    1. Update the package list:

        ```sh
        sudo apt update
        ```
    2. Install any available updates:

        ```sh
        sudo apt upgrade -y
        ```
    3. Install the toolchain, and ensure accurate time synchronization:

        ```sh
        sudo apt install make build-essential gcc git jq chrony -y
        ```
    4. Download a version of Go that's greater than, or equal to 1.18, and less than 2:

        ```sh
        wget https://golang.org/dl/go1.18.2.linux-amd64.tar.gz
        ```
    5. Install Go:

        ```sh
        sudo tar xzvf go1.18.2.linux-amd64.tar.gz -C /usr/local
        ```
    6. Delete the Go download:

        ```sh
        rm go1.18.2.linux-amd64.tar.gz
        ```
    7. Configure Go:

        ```sh
        tee -a ~/.profile << EOF
        export GOROOT=/usr/local/go
        export GOPATH=$HOME/go
        export GO111MODULE=on
        export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
        EOF
        ```
    8. Refresh environment variables:

        ```sh
        source ~/.profile
        ```
3. Install Juno:
    1. Clone the repo:

        ```sh
        git clone https://github.com/CosmosContracts/juno
        ```
    2. Change the directory:

        ```sh
        cd juno
        ```
    3. Fetch the tags:

        ```sh
        git fetch
        ```
    4. Check out the relevant version:

        ```sh
        git checkout <TAG>
        ```

       If you're going to download a snapshot, then replace `<TAG>` with the version's tag (tags can be found [here](https://github.com/CosmosContracts/juno/tags)) that the snapshot supports. Otherwise, replace `<TAG>` with the first version's tag (i.e., `v3.0.0`).
    5. Install:

        ```sh
        make install
        ```
    6. If the installation succeeded, then the following command will print the `<TAG>` (the one selected in a previous step):

        ```sh
        junod version
        ```
4. Configure Juno:
    1. Create a variable for the repo:

        ```sh
        CHAIN_REPO="https://raw.githubusercontent.com/CosmosContracts/mainnet/main/juno-1"
        ```
    2. Create a variable for the peers:

        ```sh
        PEERS="$(curl -sL "$CHAIN_REPO/persistent_peers.txt")"
        ```
    3. Initialize the chain:

        ```sh
        junod init <MONIKER> --chain-id juno-1
        ```

       Replace `<MONIKER>` with the node's moniker (e.g., `node`).
    4. Download the genesis file:

        ```sh
        curl https://share.blockpane.com/juno/phoenix/genesis.json > ~/.juno/config/genesis.json
        ```
    5. Set the peers:

        ```sh
        sed -i "s/persistent_peers = .*/persistent_peers = \"$PEERS\"/" ~/.juno/config/config.toml
        ```
    6. If the node is on the latest version, then skip this step.
    
        ```sh
        sed -i 's/halt-height = 0/halt-height = <HEIGHT>/' ~/.juno/config/app.toml
        ```

        Replace `<HEIGHT>` with the block height to halt at (`2616300` if you downloaded the first version of the node).   
    7. Follow this step if you want to enable the REST API. In the `[api]` section of `~/.juno/config/app.toml`, set the `enable` key's value to `true`.
5. Set up the Juno systemd unit:
    1. Create the unit:

        ```sh
        sudo tee -a /etc/systemd/system/junod.service << EOF
        [Unit]
        Description=Juno Daemon
        Wants=network-online.target
        After=network-online.target

        [Service]
        User=ubuntu
        ExecStart=/home/<USER>/go/bin/junod start --x-crisis-skip-assert-invariants
        RestartSec=3
        Restart=always
        LimitNOFILE=4096

        [Install]
        WantedBy=multi-user.target
        EOF
        ```
       
        Replace `<USER>` with your user (e.g., `ubuntu`).

        We use the `--x-crisis-skip-assert-invariants` flag to skip a check that takes up to several hours to complete. It's recommended to skip it, and it doesn't affect archive nodes anyway.
    2. Start the unit:

        ```sh
        sudo systemctl enable --now junod
        ```
    3. Verify that it's running:

        ```sh
        sudo systemctl status junod
        ```

## URL Setup

This optional but recommended section explains how to set up the DNS and TLS certificate. We explain it by using Cloudflare for the DNS, and Caddy for the TLS certificate. Of course, you can use different tools such as Google Domains and Traefik instead.

### DNS

This section explains how to use your domain name (e.g., example.com) instead of your server's IP address. It assumes that you already have a domain name, and a Cloudflare setup.
1. Go to your website's DNS section on Cloudflare.
2. Click **Add record**.
3. Set the **Type** field to **A**.
4. Set the **Name (required)** field to something like **juno**.
5. Set the **IPv4 address (required)** field to your server's IP address.
6. Set the **Proxy status** to **DNS only**.
7. Set the **TTL** to **Auto**.

### TLS Certificate

This section explains how to set up the TLS certificate, and URLs for each API you want to expose.
1. Install [Caddy](https://caddyserver.com/docs/install) on your server.
2. Configure Caddy:

    ```sh
    sudo tee /etc/caddy/Caddyfile << EOF
    <DOMAIN> {
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
    ```
   
    Replace `<DOMAIN>` with your domain name (e.g., `juno.example.com`).

    Remove the irrelevant blocks. For example, if you've not enabled the REST API, then remove the following block:

    ```
    handle_path /rest-api/* {
        rewrite * {path}
        reverse_proxy :1317
    }
    ```
3. Configure the Caddy systemd unit:

    ```sh
    sudo tee /usr/lib/systemd/system/caddy.service << EOF
    [Unit]
    Description=Caddy
    Documentation=https://caddyserver.com/docs/
    After=network.target network-online.target
    Requires=network-online.target
    
    [Service]
    Type=notify
    ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
    ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile --force
    TimeoutStopSec=5s
    LimitNOFILE=1048576
    LimitNPROC=512
    PrivateTmp=true
    ProtectSystem=full
    AmbientCapabilities=CAP_NET_BIND_SERVICE
    
    [Install]
    WantedBy=multi-user.target
    EOF
    ```
4. Reload systemd:

    ```sh
    sudo systemctl daemon-reload
    ```
5. Restart Caddy:

    ```sh
    sudo systemctl restart caddy
    ```

Except for blocks you deleted from step 2, the following URLs will now be available (`<DOMAIN>` is the same as from step 2):
- REST API: `https://<DOMAIN>/rest-api`
- gRPC: `https://<DOMAIN>/grpc`
- gRPC Web: `https://<DOMAIN>/grpc-web`
- Metrics: `https://<DOMAIN>/metrics`

For example (assuming that the REST API is enabled), you can now query a tx using the REST API base URL of `https://<DOMAIN>/rest-api` by opening `https://<DOMAIN>/rest-api/cosmos/tx/v1beta1/txs/8E9623B92C4501432EFDE993E6077B1FD021613CE1980859A1B4F0BB374BC1A9` in a browser.

## Monitoring Setup

This optional but recommended section explains how to set up metrics. We explain it by installing Prometheus directly on the server, and connecting it to Grafana Cloud. Of course, you can run Grafana yourself, use an external Prometheus server, etc. instead.

### Grafana Cloud

This section explains how to visualize the metrics.
1. Sign up for [Grafana Cloud](https://grafana.com/auth/sign-up/create-user).

### Node Exporter

This section explains how to set up the hardware metrics.

1. Go to the home directory:

    ```sh
    cd
    ```
2. Download Node Exporter:

    ```sh
    wget <URL>
    ```
   
    Replace `<URL>` with the relevant [URL](https://prometheus.io/download/#node_exporter) (e.g., `https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz`).
3. Extract the download:

    ```sh
    tar xvzf <DOWNLOAD>
    ```
   
    Replace `<DOWNLOAD>` with the Node Exporter download name (e.g., `node_exporter-1.3.1.linux-amd64.tar.gz`).
4. Delete the node_exporter download:

    ```sh
    rm <DOWNLOAD>
    ```
   
    Replace `<DOWNLOAD>` with the Node Exporter download name (e.g., `node_exporter-1.3.1.linux-amd64.tar.gz`);
5. Set up the Node Exporter systemd unit:
    1. Create the unit:

        ```sh
        sudo tee -a /etc/systemd/system/node-exporter.service << EOF
        [Unit]
        Description=Node Exporter
        Wants=network-online.target
        After=network-online.target

        [Service]
        User=<USER>
        ExecStart=/home/<USER>/<DIR>/node_exporter
        Restart=always
        RestartSec=3
        LimitNOFILE=4096

        [Install]
        WantedBy=multi-user.target
        EOF
        ```

        Replace `<USER>` with your user (e.g., `ubuntu`). Replace `<DIR>` with the name of Node Exporter directory (e.g., `node_exporter-1.3.1.linux-amd64`).
    2. Start the unit:

        ```sh
        sudo systemctl enable --now node-exporter
        ```
    3. Verify that it's running:

        ```sh
        sudo systemctl status node-exporter
        ```

### Prometheus

This section explains how to aggregate the metrics from Juno and Node Exporter.
1. Go to the home directory:

    ```sh
    cd
    ```
2. In the `[instrumentation]` section of `~/.juno/config/config.toml`, set the `prometheus` key's value to `true`.
3. Download Prometheus:

    ```sh
    wget <URL>
    ```

   Replace `<URL>` with the relevant [URL](https://prometheus.io/download/#prometheus) (e.g., `https://github.com/prometheus/prometheus/releases/download/v2.37.0/prometheus-2.37.0.linux-amd64.tar.gz` for a Linux amd64 server).
4. Extract the download:

    ```sh
    tar xvzf <DOWNLOAD>
    ```

   Replace `<DOWNLOAD>` with the Prometheus download name (e.g., `prometheus-2.37.0.linux-amd64.tar.gz`).
5. Delete the Prometheus download:

    ```sh
    rm <DOWNLOAD>
    ```

   Replace `<DOWNLOAD>` with the Prometheus download name (e.g., `prometheus-2.37.0.linux-amd64.tar.gz`).
6. Configure Prometheus:

    ```sh
    tee <DIR>/prometheus.yml << EOF
    global:
      scrape_interval: 15s

    scrape_configs:
      - job_name: node-exporter
        static_configs:
          - targets: ['localhost:9100']
      - job_name: juno
        static_configs:
          - targets: ['localhost:26660']

    remote_write:
      - url: <URL>
        basic_auth:
          username: <USERNAME>
          password: <PASSWORD>
    EOF
    ```

   Replace `<DIR>` with the directory the Prometheus download was extracted to (e.g., `prometheus-2.37.0.linux-amd64`). Replace `<URL>`, `<USERNAME>`, and `<PASSWORD>` with your Grafana Cloud [credentials](https://grafana.com/docs/grafana-cloud/reference/create-api-key/).
7. Set up the Prometheus systemd unit:
    1. Create the unit:

        ```sh
        sudo tee -a /etc/systemd/system/prometheus.service << EOF
        [Unit]
        Description=Prometheus
        Wants=network-online.target
        After=network-online.target

        [Service]
        User=<USER>
        ExecStart=/home/<USER>/<DIR>/prometheus --config.file=/home/<USER>/<DIR>/prometheus.yml --web.listen-address=:6666 --storage.tsdb.path=/home/<USER>/<DIR>/data
        Restart=always
        RestartSec=3
        LimitNOFILE=4096

        [Install]
        WantedBy=multi-user.target
        EOF
        ```

       Replace `<USER>` with your user (e.g., `ubuntu`). Replace `<DIR>` with the name of Prometheus directory (e.g., `prometheus-2.37.0.linux-amd64`).
    2. Start the unit:

        ```sh
        sudo systemctl enable --now prometheus
        ```
    3. Verify that it's running:

        ```sh
        sudo systemctl status prometheus
        ```
