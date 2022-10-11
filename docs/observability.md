# Observability

This optional but recommended section explains how to set up monitoring and alerting. We explain it by using Prometheus, connecting Prometheus to Grafana Cloud, and using PANIC. Of course, you can run Grafana yourself, use an external Prometheus server, use [Tenderduty](https://github.com/blockpane/tenderduty), etc. instead.

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
   
    set PROMPT 'Enter your Grafana Cloud URL (e.g.,'
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
        set PROMPT 'Enter the URL for the metrics endpoint of the first sentry\'s blockchain node such as'
        set PROMPT "$PROMPT https://sentry-1.osmo-test-4.example.com/blockchain-node: "
        read -P $PROMPT SENTRY_1_BLOCKCHAIN_NODE

        set PROMPT 'Enter the URL for the metrics endpoint of the first sentry\'s Node Exporter such as'
        set PROMPT "$PROMPT https://sentry-1.osmo-test-4.example.com/node-exporter: "
        read -P $PROMPT SENTRY_1_NODE_EXPORTER
      
        set PROMPT 'Enter the URL for the metrics endpoint of the second sentry\'s blockchain node such as'
        set PROMPT "$PROMPT https://sentry-2.osmo-test-4.example.com/blockchain-node: "
        read -P $PROMPT SENTRY_2_BLOCKCHAIN_NODE
              
        set PROMPT 'Enter the URL for the metrics endpoint of the second sentry\'s Node Exporter such as'
        set PROMPT "$PROMPT https://sentry-2.osmo-test-4.example.com/node-exporter: "
        read -P $PROMPT SENTRY_2_NODE_EXPORTER

        set PROMPT 'Enter the URL for the metrics endpoint of the third sentry\'s blockchain node such as'
        set PROMPT "$PROMPT https://sentry-3.osmo-test-4.example.com/blockchain-node: "
        read -P $PROMPT SENTRY_3_BLOCKCHAIN_NODE
        
        set PROMPT 'Enter the URL for the metrics endpoint of the third sentry\'s Node Exporter such as'
        set PROMPT "$PROMPT https://sentry-3.osmo-test-4.example.com/node-exporter: "
        read -P $PROMPT SENTRY_3_NODE_EXPORTER
        
        set PROMPT 'Enter the URL for the metrics endpoint of the first cosigner\'s Node Exporter such as'
        set PROMPT "$PROMPT https://cosigner-1.osmo-test-4.example.com/node-exporter: "
        read -P $PROMPT COSIGNER_1_NODE_EXPORTER
        
        set PROMPT 'Enter the URL for the metrics endpoint of the second cosigner\'s Node Exporter such as'
        set PROMPT "$PROMPT https://cosigner-2.osmo-test-4.example.com/node-exporter: "
        read -P $PROMPT COSIGNER_2_NODE_EXPORTER
        
        set PROMPT 'Enter the URL for the metrics endpoint of the third cosigner\'s Node Exporter such as'
        set PROMPT "$PROMPT https://cosigner-3.osmo-test-4.example.com/node-exporter: "
        read -P $PROMPT COSIGNER_3_NODE_EXPORTER

        printf "\
        scrape_configs:
          - job_name: node-exporter-for-monitor
            static_configs:
              - targets: ['localhost:9100']
      
          - job_name: blockchain-node-for-sentry-1
            static_configs:
              - targets: ['$SENTRY_1_BLOCKCHAIN_NODE']
          - job_name: blockchain-node-for-sentry-2
            static_configs:
              - targets: ['$SENTRY_2_BLOCKCHAIN_NODE']
          - job_name: blockchain-node-for-sentry-3
            static_configs:
              - targets: ['$SENTRY_3_BLOCKCHAIN_NODE']
      
          - job_name: node-exporter-for-sentry-1
            static_configs:
              - targets: ['$SENTRY_1_NODE_EXPORTER']
          - job_name: node-exporter-for-sentry-2
            static_configs:
              - targets: ['$SENTRY_2_NODE_EXPORTER']
          - job_name: node-exporter-for-sentry-3
            static_configs:
              - targets: ['$SENTRY_3_NODE_EXPORTER']
      
          - job_name: node-exporter-for-cosigner-1
            static_configs:
              - targets: ['$COSIGNER_1_NODE_EXPORTER']
          - job_name: node-exporter-for-cosigner-2
            static_configs:
              - targets: ['$COSIGNER_2_NODE_EXPORTER']
          - job_name: node-exporter-for-cosigner-3
            static_configs:
              - targets: ['$COSIGNER_3_NODE_EXPORTER']
        remote_write:
          - url: $GRAFANA_URL
            basic_auth:
            username: $GRAFANA_USERNAME
            password: $GRAFANA_PASSWORD
        " > $DIR/prometheus.yml
        ```
    - Only follow this step if you're monitoring a full node:

        ```shell
        set PROMPT 'Enter the URL for the metrics endpoint of the first full node\'s blockchain node such as'
        set PROMPT "$PROMPT https://archive-node-1.osmo-test-4.example.com/blockchain-node: "
        read -P $PROMPT FULL_NODE_1_BLOCKCHAIN_NODE

        set PROMPT 'Enter the URL for the metrics endpoint of the first full node\'s Node Exporter such as'
        set PROMPT "$PROMPT https://archive-node-1.osmo-test-4.example.com/node-exporter: "
        read -P $PROMPT FULL_NODE_1_NODE_EXPORTER
            
        set PROMPT 'Enter the URL for the metrics endpoint of the second full node\'s blockchain node such as'
        set PROMPT "$PROMPT https://archive-node-2.osmo-test-4.example.com/blockchain-node: "
        read -P $PROMPT FULL_NODE_2_BLOCKCHAIN_NODE
        
        set PROMPT 'Enter the URL for the metrics endpoint of the second full node\'s Node Exporter such as'
        set PROMPT "$PROMPT https://archive-node-2.osmo-test-4.example.com/node-exporter: "
        read -P $PROMPT FULL_NODE_2_NODE_EXPORTER
      
        printf "\
        scrape_configs:
          - job_name: node-exporter-for-monitor
            static_configs:
              - targets: ['localhost:9100']
      
          - job_name: blockchain-node-for-full-node-1
            static_configs:
              - targets: ['$FULL_NODE_1_BLOCKCHAIN_NODE']
          - job_name: blockchain-node-for-full-node-2
            static_configs:
              - targets: ['$FULL_NODE_2_BLOCKCHAIN_NODE']
      
          - job_name: node-exporter-for-full-node-1
            static_configs:
              - targets: ['$FULL_NODE_1_NODE_EXPORTER']
          - job_name: node-exporter-for-full-node-2
            static_configs:
              - targets: ['$FULL_NODE_2_NODE_EXPORTER']
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
6. Set up [PANIC](https://github.com/SimplyVC/panic).
