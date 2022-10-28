# Observability

This optional but recommended section explains how to set up monitoring and alerting. We explain it by using Prometheus, connecting Prometheus to Grafana Cloud, and using PANIC. Of course, you can run Grafana yourself, use an external Prometheus server, use [Tenderduty](https://github.com/blockpane/tenderduty), etc. instead. It's assumed that you've followed the [URL Setup](url-setup.md) doc because this doc assumes URLs that look like https://rpc-node-1.osmosis-1.example.com/blockchain-node.

1. Sign up for [Grafana Cloud](https://grafana.com/auth/sign-up/create-user).
2. On your Grafana Cloud instance, create an [API key](https://grafana.com/docs/grafana-cloud/reference/create-api-key/)s for the Prometheus integration with the **Role** set to **MetricsPublisher**. Note down the URL, username, and password for use later on.
3. Install Prometheus:

    ```shell
    cd
   
    ###############################
    ## BEGIN: Install Prometheus ##
    ###############################
   
    set PROMPT 'You\'ll be prompted to enter a Prometheus download link. The Prometheus download link is the relevant'
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
   
    set PROMPT 'Enter your Prometheus instance\'s URL (e.g.,'
    set PROMPT "$PROMPT https://prometheus-prod-10-prod-us-central-0.grafana.net/api/prom/push): "
    read -P $PROMPT GRAFANA_URL
   
    read -P 'Enter your Grafana username (e.g., 986969): ' GRAFANA_USERNAME
   
    set PROMPT 'Enter your Grafana password (e.g.,'
    set PROMPT "$PROMPT oop09ikiM2YxZDJjOGY5YmJlODUxMmNpoiuyt2IzZDI3YWFlNzQyZGE2ZDdjYiIsIm4iOiJzZWktdGVzdG5ldC12Ykj3): "
    read -P $PROMPT GRAFANA_PASSWORD
    ```
4. Configure Prometheus using one of the following:
    - Only follow this step if you're monitoring Horcrux:

        ```shell
        read -P 'Enter the first sentry\'s hostname such as sentry-1.osmo-test-4.example.com: ' SENTRY_1
        read -P 'Enter the second sentry\'s hostname such as sentry-2.osmo-test-4.example.com: ' SENTRY_2
        read -P 'Enter the third sentry\'s hostname such as sentry-3.osmo-test-4.example.com: ' SENTRY_3
      
        read -P 'Enter the first cosigner\'s hostname such as cosigner-1.osmo-test-4.example.com: ' COSIGNER_1
        read -P 'Enter the first cosigner\'s hostname such as cosigner-2.osmo-test-4.example.com: ' COSIGNER_2
        read -P 'Enter the first cosigner\'s hostname such as cosigner-3.osmo-test-4.example.com: ' COSIGNER_3

        printf "\
        scrape_configs:
        - job_name: node-exporter-for-monitor
          static_configs:
          - targets: [localhost:9100]
        - job_name: blockchain-node-for-sentry-1
          static_configs:
          - targets: [$SENTRY_1]
          metrics_path: /blockchain-node
        - job_name: blockchain-node-for-sentry-2
          static_configs:
          - targets: [$SENTRY_2]
          metrics_path: /blockchain-node
        - job_name: blockchain-node-for-sentry-3
          static_configs:
          - targets: [$SENTRY_3]
          metrics_path: /blockchain-node
        - job_name: node-exporter-for-sentry-1
          static_configs:
          - targets: [$SENTRY_1]
          metrics_path: /node-exporter
        - job_name: node-exporter-for-sentry-2
          static_configs:
          - targets: [$SENTRY_2]
          metrics_path: /node-exporter
        - job_name: node-exporter-for-sentry-3
          static_configs:
          - targets: [$SENTRY_3]
          metrics_path: /node-exporter
        - job_name: node-exporter-for-cosigner-1
          static_configs:
          - targets: [$COSIGNER_1]
          metrics_path: /node-exporter
        - job_name: node-exporter-for-cosigner-2
          static_configs:
          - targets: [$COSIGNER_2]
          metrics_path: /node-exporter
        - job_name: node-exporter-for-cosigner-3
          static_configs:
          - targets: [$COSIGNER_3]
          metrics_path: /node-exporter
        remote_write:
        - url: $GRAFANA_URL
          basic_auth:
            username: $GRAFANA_USERNAME
            password: $GRAFANA_PASSWORD
        " > $DIR/prometheus.yml
        ```
    - Only follow this step if you're monitoring a full node:

        ```shell
        read -P 'Enter the first full node\'s hostname such as rpc-node-1.osmo-test-4.example.com: ' FULL_NODE_1
        read -P 'Enter the second full node\'s hostname such as rpc-node-2.osmo-test-4.example.com: ' FULL_NODE_2
      
        printf "\
        scrape_configs:
        - job_name: node-exporter-for-monitor
          static_configs:
          - targets: ['localhost:9100']
        - job_name: blockchain-node-for-full-node-1
          static_configs:
          - targets: [$FULL_NODE_1]
          metrics_path: /blockchain-node
        - job_name: blockchain-node-for-full-node-2
          static_configs:
          - targets: [$FULL_NODE_2]
          metrics_path: /blockchain-node
        - job_name: node-exporter-for-full-node-1
          static_configs:
          - targets: [$FULL_NODE_1]
          metrics_path: /node-exporter
        - job_name: node-exporter-for-full-node-2
          static_configs:
          - targets: [$FULL_NODE_2]
          metrics_path: /node-exporter
        remote_write:
        - url: $GRAFANA_URL
          basic_auth:
            username: $GRAFANA_USERNAME
            password: $GRAFANA_PASSWORD
        " > $DIR/prometheus.yml
        ```
5. Set up a systemd unit for Prometheus:

    ```shell
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
    ```
6. Set up [PANIC](https://github.com/SimplyVC/panic). We recommend using `tmux` during the setup since there are long-running commands.
