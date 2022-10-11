# Node Exporter Setup

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
