# URL Setup

This optional but recommended section explains how to set up the DNS and TLS certificate. We explain it by using Cloudflare for the DNS, and Caddy for the TLS certificate. Of course, you can use different tools such as Google Domains and Traefik instead.

This section explains how to secure the REST API in case you're running a validator monitored by PANIC, and have enabled the REST API as a result. If you're not going to follow this section, then be sure to prevent IP addresses other than the server itself from accessing the REST API.

## DNS

This section explains how to use your domain name (e.g., example.com) instead of your server's IP address. It assumes that you already have a domain name, and a Cloudflare setup.
1. Go to your website's DNS section on Cloudflare.
2. Click **Add record**.
3. Set the **Type** field to **A**.
4. Set the **Name (required)** field (e.g., **osmo-test-4**).
5. Set the **IPv4 address (required)** field to your server's IP address.
6. Set the **Proxy status** to **DNS only**.
7. Set the **TTL** to **Auto**.

## TLS Certificate

This section explains how to set up the TLS certificate, and URLs for each API that you want to expose. There's a reference to Prometheus which is a monitoring system that you have the option to set up in the next section.

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

############################
## BEGIN: Configure Caddy ##
############################

read -P 'Enter the domain such as localhost or osmo-test-4.example.com: ' DOMAIN

read -P 'Enter y if you\'re running a validator, and using PANIC, and n otherwise: ' MUST_WHITELIST
if test $MUST_WHITELIST = 'y'
    set IP_ADDRESS (curl -s https://icanhazip.com/)
    set WHITELIST "\n        @denied not remote_ip $IP_ADDRESS\n        abort @denied"
else
    set WHITELIST ''
end

printf "\
$DOMAIN {
    handle_path /tendermint-rpc/* {
        rewrite * {path}
        reverse_proxy :26657
    }
    handle_path /rest-api/* {
        rewrite * {path}
        reverse_proxy :1317$WHITELIST
    }
    handle_path /grpc/* {
        rewrite * {path}
        reverse_proxy :9090
    }
    handle_path /grpc-web/* {
        rewrite * {path}
        reverse_proxy :9091
    }
    handle_path /prometheus/* {
        rewrite * {path}
        reverse_proxy :6666
    }
    handle_path /node-exporter/* {
        rewrite * {path}
        reverse_proxy :9100
    }
    handle_path /blockchain-node/* {
        rewrite * {path}
        reverse_proxy :26660
    }
}
" | sudo tee /etc/caddy/Caddyfile

sudo systemctl reload caddy

##########################
## END: Configure Caddy ##
##########################
```

The following URLs will now be available, where `<DOMAIN>` is the same as the domain you entered during the prompt. Services which aren't enabled won't exist. For example, `https://<DOMAIN>/rest-api` won't work if the node's REST API is disabled.
- RPC API: `https://<DOMAIN>/tendermint-rpc`
- REST API: `https://<DOMAIN>/rest-api`
- gRPC: `https://<DOMAIN>/grpc`
- gRPC Web: `https://<DOMAIN>/grpc-web`
- Prometheus's metrics (for use by Grafana): `https://<DOMAIN>/prometheus`
- Node Exporter's metrics (for use by PANIC): `https://<DOMAIN>/node-exporter`
- Blockchain node's metrics (for use by PANIC): `https://<DOMAIN>/blockchain-node`

For example (assuming that you're running a Juno mainnet archive node with the REST API enabled), you can now query a tx using the REST API base URL of `https://<DOMAIN>/rest-api` by opening `https://<DOMAIN>/rest-api/cosmos/tx/v1beta1/txs/8E9623B92C4501432EFDE993E6077B1FD021613CE1980859A1B4F0BB374BC1A9` in a browser.

Note that if you're using PANIC, then you'll need to access it via `https://<IP>:3333` and `https://<IP>:8000` (where `<IP>` is the server's IP address) rather than through the reverse proxy.
