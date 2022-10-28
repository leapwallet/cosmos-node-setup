# Firewall

This document explains how to set up the firewall for each type of server. Regardless of what the server is for, all outgoing traffic must be allowed.

## Sentries

- Allow SSH connections over TCP on port 22 from your IP address. This is to allow you to access the server.
- Allow HTTP connections on port 80, and HTTPS connections on port 443. This is for the reverse proxy server which deals with things such as the metrics endpoints.
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

read -P 'Enter the IP address of the first cosigner: ' COSIGNER_1
sudo ufw allow from $COSIGNER_1 proto tcp to any port 1234

read -P 'Enter the IP address of the second cosigner: ' COSIGNER_2
sudo ufw allow from $COSIGNER_2 proto tcp to any port 1234

read -P 'Enter the IP address of the third cosigner: ' COSIGNER_3
sudo ufw allow from $COSIGNER_3 proto tcp to any port 1234

###########################
## END: Enable cosigners ##
###########################

sudo ufw enable
sudo ufw status
```

## Cosigners

- Allow SSH connections over TCP on port 22 from your IP address. This is to allow you to access the server.
- Allow HTTP connections on port 80, and HTTPS connections on port 443. This is for the reverse proxy server which deals with things such as the metrics endpoints.
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

switch $COSIGNER
    case 1
        read -P 'Enter the first sentry\'s IP address: ' SENTRY_1
        sudo ufw allow from $SENTRY_1
        
        read -P 'Enter the second cosigner\'s IP address: ' COSIGNER_2
        sudo ufw allow from $COSIGNER_2
        
        read -P 'Enter the third cosigner\'s IP address: ' COSIGNER_3
        sudo ufw allow from $COSIGNER_3
    case 2
        read -P 'Enter the second sentry\'s IP address: ' SENTRY_2
        sudo ufw allow from $SENTRY_2
        
        read -P 'Enter the first cosigner\'s IP address: ' COSIGNER_1
        sudo ufw allow from $COSIGNER_1
        
        read -P 'Enter the third cosigner\'s IP address: ' COSIGNER_3
        sudo ufw allow from $COSIGNER_3
    case 3
        read -P 'Enter the third sentry\'s IP address: ' SENTRY_3
        sudo ufw allow from $SENTRY_3
        
        read -P 'Enter the first cosigner\'s IP address: ' COSIGNER_1
        sudo ufw allow from $COSIGNER_1
        
        read -P 'Enter the second cosigner\'s IP address: ' COSIGNER_2
        sudo ufw allow from $COSIGNER_2
end

#########################
## END: Enable Horcrux ##
#########################

sudo ufw enable
sudo ufw status
```

## Monitors

- Allow SSH connections over TCP on port 22 from your IP address. This is to allow you to access the server.
- Allow HTTP connections on port 80, and HTTPS connections on port 443. This is for the reverse proxy server which deals with things such as the metrics endpoints.
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

## Validators and Full Nodes

- Allow SSH connections over TCP on port 22 from your IP address. This is to allow you to access the server.
- Allow HTTP connections on port 80, and HTTPS connections on port 443. This is for the reverse proxy server which deals with things such as the metrics endpoints.
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
