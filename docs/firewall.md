# Firewall

This document explains how to set up the firewall for each type of server. Regardless of what the server is for, all outgoing traffic must be allowed.

## Sentry

- Allow SSH connections over TCP on port 22 from your IP address. This is to allow you to access the server.
- Allow HTTP connections on port 80 from the monitors' IP addresses, and HTTPS connections on port 443 from the monitors' IP addresses. This is for the reverse proxy server which deals with things such as the metrics endpoints.
- Allow TCP connections on port 26656 from any IP address. This is to allow other nodes to sync with yours.
- Allow TCP connections on port 1234 to each cosigner. This is to allow the cosigner to access the private validator port.

If you're using something like AWS, then you can configure the firewall using a GUI. Otherwise, here's how to set up the firewall using the CLI:

```shell
#######################
## BEGIN: Enable SSH ##
####################### 

read -P 'Enter your IP address: ' IP_ADDRESS
sudo ufw allow from $IP_ADDRESS proto tcp to any port 22

#####################
## END: Enable SSH ##
##################### 

########################################
## BEGIN: Enable reverse proxy server ##
########################################

sudo ufw allow http
sudo ufw allow https

######################################
## END: Enable reverse proxy server ##
######################################

###########################
## BEGIN: Enable peering ##
###########################

sudo ufw allow from any to any port 26656 proto tcp

#########################
## END: Enable peering ##
#########################

#############################
## BEGIN: Enable cosigners ##
#############################

read -P 'Enter the IP address of the first cosigner: ' COSIGNER1
sudo ufw allow from $COSIGNER1 proto tcp to any port 1234

read -P 'Enter the IP address of the second cosigner: ' COSIGNER2
sudo ufw allow from $COSIGNER2 proto tcp to any port 1234

read -P 'Enter the IP address of the third cosigner: ' COSIGNER3
sudo ufw allow from $COSIGNER3 proto tcp to any port 1234

###########################
## END: Enable cosigners ##
###########################

sudo ufw enable
sudo ufw status
```

## Cosigner

- Allow SSH connections over TCP on port 22 from your IP address. This is to allow you to access the server.
- Allow HTTP connections on port 80 from the monitors' IP addresses, and HTTPS connections on port 443 from the monitors' IP addresses. This is for the reverse proxy server which deals with things such as the metrics endpoints.
- Allow all traffic from each sentry and cosigner.

If you're using something like AWS, then you can configure the firewall using a GUI. Otherwise, here's how to set up the firewall using the CLI:

```shell
#######################
## BEGIN: Enable SSH ##
####################### 

read -P 'Enter your IP address: ' IP_ADDRESS
sudo ufw allow from $IP_ADDRESS proto tcp to any port 22

#####################
## END: Enable SSH ##
#####################

########################################
## BEGIN: Enable reverse proxy server ##
########################################

sudo ufw allow http
sudo ufw allow https

######################################
## END: Enable reverse proxy server ##
######################################

###########################
## BEGIN: Enable Horcrux ##
###########################

read -P 'Enter which cosigner (1, 2, or 3) this is: ' COSIGNER
read -P 'Enter the first cosigner\'s IP address: ' COSIGNER1
read -P 'Enter the second cosigner\'s IP address: ' COSIGNER2
read -P 'Enter the third cosigner\'s IP address: ' COSIGNER3
switch $COSIGNER
    case 1
        read -P 'Enter the first sentry\'s IP address: ' SENTRY1
        horcrux config init ($DAEMON_NAME config chain-id) tcp://$SENTRY1:1234 \
            -c \
            -p "tcp://$COSIGNER2:2222|2,tcp://$COSIGNER3:2222|3" \
            -l tcp://$COSIGNER1:2222 \
            -t 2 \
            --timeout 1500ms
    case 2
        read -P 'Enter the second sentry\'s IP address: ' SENTRY2
        horcrux config init ($DAEMON_NAME config chain-id) tcp://$SENTRY2:1234 \
            -c \
            -p "tcp://$COSIGNER1:2222|1,tcp://$COSIGNER3:2222|3" \
            -l tcp://$COSIGNER2:2222 \
            -t 2 \
            --timeout 1500ms
    case 3
        read -P 'Enter the third sentry\'s IP address: ' SENTRY3
        horcrux config init ($DAEMON_NAME config chain-id) tcp://$SENTRY3:1234 \
            -c \
            -p "tcp://$COSIGNER1:2222|1,tcp://$COSIGNER2:2222|2" \
            -l tcp://$COSIGNER3:2222 \
            -t 2 \
            --timeout 1500ms
end

#########################
## END: Enable Horcrux ##
#########################

sudo ufw enable
sudo ufw status
```

## Monitor

- Allow SSH connections over TCP on port 22 from your IP address. This is to allow you to access the server.
- Allow HTTP connections on port 80 from the monitors' IP addresses, and HTTPS connections on port 443 from the monitors' IP addresses. This is for the reverse proxy server which deals with things such as the metrics endpoints.
- Allow HTTPS connections on ports 3333 and 8000 from your IP address. This is to allow you to access PANIC.

If you're using something like AWS, then you can configure the firewall using a GUI. Otherwise, here's how to set up the firewall using the CLI:

```shell
#######################
## BEGIN: Enable SSH ##
####################### 

read -P 'Enter your IP address: ' IP_ADDRESS
sudo ufw allow from $IP_ADDRESS proto tcp to any port 22

#####################
## END: Enable SSH ##
#####################

########################################
## BEGIN: Enable reverse proxy server ##
########################################

sudo ufw allow http
sudo ufw allow https

######################################
## END: Enable reverse proxy server ##
######################################

#########################
## BEGIN: Enable PANIC ##
#########################

sudo ufw allow from $IP_ADDRESS proto tcp to any port 3333,8000

#######################
## END: Enable PANIC ##
#######################

sudo ufw enable
sudo ufw status
```

## Full Node

- Allow SSH connections over TCP on port 22 from your IP address. This is to allow you to access the server.
- Allow HTTP connections on port 80 from the monitors' IP addresses, and HTTPS connections on port 443 from the monitors' IP addresses. This is for the reverse proxy server which deals with things such as the metrics endpoints.
- Allow TCP connections on port 26656 from any IP address. This is to allow other nodes to sync with yours.

If you're using something like AWS, then you can configure the firewall using a GUI. Otherwise, here's how to set up the firewall using the CLI:

```shell
#######################
## BEGIN: Enable SSH ##
####################### 

read -P 'Enter your IP address: ' IP_ADDRESS
sudo ufw allow from $IP_ADDRESS proto tcp to any port 22

#####################
## END: Enable SSH ##
##################### 

########################################
## BEGIN: Enable reverse proxy server ##
########################################

sudo ufw allow http
sudo ufw allow https

######################################
## END: Enable reverse proxy server ##
######################################

###########################
## BEGIN: Enable peering ##
###########################

sudo ufw allow from any to any port 26656 proto tcp

#########################
## END: Enable peering ##
#########################

sudo ufw enable
sudo ufw status
```
