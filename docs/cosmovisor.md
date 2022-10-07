# Cosmovisor

```shell
###############################
## BEGIN: Install Cosmovisor ##
###############################

cd
git clone https://github.com/cosmos/cosmos-sdk
cd cosmos-sdk
git checkout cosmovisor/v1.1.0
make cosmovisor

#############################
## END: Install Cosmovisor ##
#############################

#################################
## BEGIN: Configure Cosmovisor ##
#################################

cp cosmovisor/cosmovisor $GOPATH/bin/cosmovisor
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
mkdir -p $DAEMON_HOME/cosmovisor/upgrades
cp ~/go/bin/$DAEMON_NAME $DAEMON_HOME/cosmovisor/genesis/bin

###############################
## END: Configure Cosmovisor ##
###############################

###############################################
## BEGIN: Set up systemd unit for Cosmovisor ##
###############################################

printf "\
[Unit]
Description=$DAEMON_NAME (Cosmovisor)
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment=\"DAEMON_NAME=$DAEMON_NAME\"
Environment=\"DAEMON_HOME=$DAEMON_HOME\"

[Install]
WantedBy=multi-user.target
" | sudo tee /etc/systemd/system/cosmovisor.service

sudo systemctl enable --now cosmovisor
sudo systemctl status cosmovisor

#############################################
## END: Set up systemd unit for Cosmovisor ##
#############################################
```
