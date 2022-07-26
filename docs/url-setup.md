# URL Setup

This section explains how to set up the DNS and TLS certificate. We explain it by using Cloudflare for the DNS, and Caddy for the TLS certificate. Of course, you can use different tools such as Google Domains and Traefik instead.

## DNS

This section explains how to use your domain name such as example.com instead of your server's IP address. It assumes that you already have a domain name, and a Cloudflare setup.

1. Go to your website's DNS section on Cloudflare.
2. Click **Add record**.
3. Set the **Type** field to **A**.
4. Set the **Name (required)** field. We recommend using the format `<SERVER>.<CHAIN_ID>.<CHAIN>.<DOMAIN>` for servers; `<SERVER>` is the server provisioned such as `archive-node-1`, `<CHAIN_ID>` is the chain ID such as `osmo-test-4`, `<CHAIN>` is the name of the blockchain such as `cosmos-hub`, and `<DOMAIN>` is your domain name such as `example.com`. We recommend you use the following `<SERVER>` names:
   - `sentry-1`, `sentry-2`, and `sentry-3` for the three sentry servers.
   - `cosigner-1`, `cosigner-2`, and `cosigner-3` for the three cosigner servers.
   - `archive-node-1` and `archive-node-2` for the two archive node servers.
   - `rpc-node-1` and `rpc-node-2` for the two RPC node servers.
   - `rpc-node-monitor` for the RPC node setup's monitor.
   - `archive-node-monitor` for the archive node setup's monitor.
   - `validator-monitor` for the validator node setup's monitor.
5. Set the **IPv4 address (required)** field to your server's IP address.
6. Set the **Proxy status** to **DNS only**.
7. Set the **TTL** to **Auto**.

## TLS Certificate

This section explains how to set up the TLS certificate, and URLs for each API that you want to expose. There's a reference to Prometheus which is a monitoring system that you will set up later on.

1. Install Caddy:

   ```shell
   ##########################
   ## BEGIN: Install Caddy ##
   ##########################

   sudo apt -y install debian-keyring debian-archive-keyring apt-transport-https
   curl -1sLf https://dl.cloudsmith.io/public/caddy/stable/gpg.key | \
       sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
   curl -1sLf https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt | \
       sudo tee /etc/apt/sources.list.d/caddy-stable.list
   sudo apt update
   sudo apt -y install caddy

   ########################
   ## END: Install Caddy ##
   ########################

   read -P 'Enter the domain such as localhost or archive-node-1.osmo-test-4.osmosis.example.com: ' DOMAIN

   read -P 'Enter y if you\'re setting up a monitor, and n otherwise: ' IS_MONITOR
   if test $IS_MONITOR = 'n'
       read -P 'Enter the IP address of the monitor: ' MONITOR
       set WHITELIST "@denied not remote_ip $MONITOR\n        abort @denied"
   end
   ```

2. Configure Caddy:

   - Follow this step if the server you're configuring is for a sentry:

     ```shell
     read -P 'Enter the IP address of this sentry\'s respective cosigner: ' COSIGNER

     printf "\
     {
         servers {
             metrics
         }
     }

     $DOMAIN {
         handle_path /tendermint-rpc/* {
             rewrite * {path}
             reverse_proxy :26657
             $WHITELIST
         }
         handle_path /rest-api/* {
             rewrite * {path}
             reverse_proxy :1317
             $WHITELIST
         }
         handle_path /node-exporter/* {
             rewrite * {path}
             reverse_proxy :9100
             $WHITELIST
         }
         handle_path /blockchain-node/* {
             rewrite * {path}
             reverse_proxy :26660
             $WHITELIST
         }
         handle_path /private-validator/* {
             rewrite * {path}
             reverse_proxy :1234
             @denied not remote_ip $COSIGNER
             abort @denied
         }
         handle_path /reverse-proxy/* {
             rewrite * {path}
             reverse_proxy :2019
             $WHITELIST
         }
     }
     " | sudo tee /etc/caddy/Caddyfile

     sudo systemctl reload caddy
     ```

   - Follow this step if the server you're configuring is for a cosigner:

     ```shell
     read -P 'Enter the IP address of this cosigner\'s respective sentry: ' SENTRY

     printf "\
     {
         servers {
             metrics
         }
     }

     $DOMAIN {
         handle_path /signer/* {
             rewrite * {path}
             reverse_proxy 2222
             @denied not remote_ip $SENTRY
             abort @denied
         }
         handle_path /node-exporter/* {
             rewrite * {path}
             reverse_proxy :9100
             $WHITELIST
         }
         handle_path /reverse-proxy/* {
             rewrite * {path}
             reverse_proxy :2019
             $WHITELIST
         }
     }
     " | sudo tee /etc/caddy/Caddyfile

     sudo systemctl reload caddy
     ```

   - Follow this step if the server you're configuring is for a full node other than a sentry:

     ```shell
     {
         servers {
             metrics
         }
     }

     printf "\
     :80, $DOMAIN {
         respond /health 204
         handle_path /tendermint-rpc/* {
             rewrite * {path}
             reverse_proxy :26657
         }
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
         handle_path /node-exporter/* {
             rewrite * {path}
             reverse_proxy :9100
             $WHITELIST
         }
         handle_path /blockchain-node/* {
             rewrite * {path}
             reverse_proxy :26660
             $WHITELIST
         }
         handle_path /reverse-proxy/* {
             rewrite * {path}
             reverse_proxy :2019
             $WHITELIST
         }
     }
     " | sudo tee /etc/caddy/Caddyfile

     sudo systemctl reload caddy
     ```

   - Follow this step if the server you're configuring is for a monitor:

     ```shell
     printf "\
     {
         servers {
             metrics
         }
     }

     $DOMAIN {
         handle_path /prometheus/* {
             rewrite * {path}
             reverse_proxy :6666
         }
     }
     " | sudo tee /etc/caddy/Caddyfile

     sudo systemctl reload caddy
     ```

The following URLs will now be available, where `<DOMAIN>` is the same as the domain you entered during the prompt, and `<IP>` is the server's IP address:

|          Service          | Servers Available On                | URL                                  | Note                                             |
| :-----------------------: | ----------------------------------- | ------------------------------------ | ------------------------------------------------ |
|          RPC API          | Full nodes                          | `https://<DOMAIN>/tendermint-rpc`    |                                                  |
|         REST API          | Full nodes                          | `https://<DOMAIN>/rest-api`          |                                                  |
|         gRPC API          | Full nodes other than sentries      | `https://<DOMAIN>/grpc`              |                                                  |
|         gRPC Web          | Full nodes other than sentries      | `https://<DOMAIN>/grpc-web`          |                                                  |
|   Prometheus's metrics    | Monitors                            | `https://<DOMAIN>/prometheus`        | For use by Grafana.                              |
|  Node Exporter's metrics  | Cosigners and full nodes            | `https://<DOMAIN>/node-exporter`     | For use by PANIC.                                |
| Blockchain node's metrics | Full nodes                          | `https://<DOMAIN>/blockchain-node`   | For use by PANIC.                                |
|  Reverse proxy's metrics  | Monitors, cosigners, and full nodes | `https://<DOMAIN>/reverse-proxy`     |                                                  |
|       Cosigner port       | Cosigners                           | `https://<DOMAIN>/signer`            | For use by the respective sentry.                |
|  Private validator port   | Sentries                            | `https://<DOMAIN>/private-validator` | For use by the respective cosigner.              |
|         PANIC UI          | Monitors                            | `https://<IP>:3333`                  | Cannot be accessed via the reverse proxy server. |
|      PANIC API docs       | Monitors                            | `https://<IP>:9000`                  | Cannot be accessed via the reverse proxy server. |

For example, if you're running a Juno `juno-1` archive node, then you can query a transaction using the REST API base URL of `https://<DOMAIN>/rest-api` by opening `https://<DOMAIN>/rest-api/cosmos/tx/v1beta1/txs/8E9623B92C4501432EFDE993E6077B1FD021613CE1980859A1B4F0BB374BC1A9` in a browser.

### Load Balancer

If you're not setting up a validator, then create a load balancer for the two full nodes. We recommend applying a rate limit of 180 requests per minute per IP address. The domain name can look like `<SERVER>.<CHAIN_ID>.<CHAIN>.<DOMAIN>` where `<SERVER>` is the server provisioned such as `archive-node`, `<CHAIN_ID>` is the chain ID such as `osmo-test-4`, `<CHAIN>` is the name of the blockchain such as `cosmos-hub`, and `<DOMAIN>` is your domain name such as `example.com`.
