# Observability

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
