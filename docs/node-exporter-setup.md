# Node Exporter Setup

```shell
cd

##################################
## BEGIN: Install Node Exporter ##
##################################

set URL https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
wget $URL
tar xvzf (basename $URL)
rm (basename $URL)

################################
## END: Install Node Exporter ##
################################
  
##################################################
## BEGIN: Set up systemd unit for Node Exporter ##
##################################################

set DIR (basename $URL | sed 's|.tar.gz||')

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
