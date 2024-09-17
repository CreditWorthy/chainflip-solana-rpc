# chainflip-solana-rpc

## Introduction
This repo helps you setup a Solana RPC node with monitoring using Prometheus Node Exporter and promtail. The default configuration will stream metrics and logs to Chainflip's monitoring backend for monitoring and alerting. Feel free to modify the configuration to suit your needs.

## Recommended Hardware

- 6-core CPU
- 32 GB RAM
- 1.5 TB SSD (RAID 0)
- 10 Gbit/s Network interface
- 2TB inbound and 1TB outbound traffic per month

## Recommended Providers

- https://www.latitude.sh/
- https://teraswitch.com/bare-metal/
- https://www.cherryservers.com/web3-infrastructure
- https://www.ovhcloud.com/
- https://www.digitalocean.com/

## Pre-requisites

- Local machine has Python 3 installed
- The node is running Ubuntu 22.04 or 24.04
- The node is accessible via SSH
- The node has a user `ubuntu` with sudo privileges. If this is not the case, update the `ansible.cfg` file with the correct `remote_user`.

## Important Notes

- The Solana playbook configures system firewall using `ufw` to block all inbound traffic except for SSH ,Solana p2p ports and HTTP/HTTPS for reverse proxy setup. If you wish to change this please check `./roles/solana-rpc/tasks/_02_ufw.yml`
- The solana config playbook configures firewall to allow chainflip's prometheus server to scrape metrics off your server. If you don't want to allow this, you can set `allow_chainflip_prometheus_server` to `false` in `./inventory/all.yml`.
- Solana is configured to accept RPC connection only from localhost (due to blocking the RPC port using `ufw`) to avoid exposing the RPC port to the public. To enable RPC connection from the outside world, you will need to setup a reverse proxy using nginx or similar.
- The default solana user password is `solana`. You can change this by updating the `solana_user_password` variable in `./inventory/all.yml`.

## Setup

1. Install the required dependencies
```bash
git clone https://github.com/chainflip-io/chainflip-solana-rpc.git
cd chainflip-solana-rpc
python3 -m venv .venv
source .venv/bin/activate
.venv/bin/pip3 install -r requirements.txt
.venv/bin/ansible-galaxy install -r requirements.yml
```
2. Update the `inventory` file under `./inventory/all.yml` with the IP addresses of your server

## Provisioning

1. Run the monitoring playbook
```bash
.venv/bin/ansible-playbook --inventory=inventory/all.yml playbooks playbooks/monitoring.yml
```

2. Run the solana playbook
```bash
.venv/bin/ansible-playbook --inventory=inventory/all.yml playbooks playbooks/solana.yml
```

## Post-provisioning

After the provisioning is done, SSH into the node and run the following commands to check the status of the solana node:
```bash
sudo -i -u solana
solana-validator --ledger /home/solana/solana-data/ledger/ monitor
```

Initially you will see the following output:
```
Ledger location: /home/solana/solana-data/ledger/
⠙ Validator startup: DownloadingLedger...
```

Then after the download is complete, you will see the following output:
```
Ledger location: /home/solana/solana-data/ledger/
⠴ Validator startup: LoadingLedger...
```

After the ledger is loaded the node will start syncing resent slots and logs will look like: and the node is fully synced, you will see the following output:
```
Ledger location: /home/solana/solana-data/ledger/
Identity: CzKPha2uyqvLfeXyq2qTXcZSAuFbqSXqcCKf7mijtCLJ
Genesis Hash: EtWTRABZaYq6iMfeYKouRu166VU2xqa1wcaWoxPkrZBG
Version: 1.18.18
Shred Version: 2405
Gossip Address: 192.168.1.123:8000
TPU Address: 192.168.1.123:8003
JSON RPC URL: http://192.168.1.123:8899
WebSocket PubSub URL: ws://192.168.1.123:8900
⠂ 00:28:12 | health unknown | Processed Slot: 326607947 | Confirmed Slot: 326607947 | Finalized Slot: 326607915 | Full Snapshot Slot: 326584258 | Incremental
```

After the node is fully synced, you will see the following output:
```
Ledger location: /home/solana/solana-data/ledger/
Identity: CzKPha2uyqvLfeXyq2qTXcZSAuFbqSXqcCKf7mijtCLJ
Genesis Hash: EtWTRABZaYq6iMfeYKouRu166VU2xqa1wcaWoxPkrZBG
Version: 1.18.18
Shred Version: 2405
Gossip Address: 192.168.1.123:8000
TPU Address: 192.168.1.123:8003
JSON RPC URL: http://192.168.1.123:8899
WebSocket PubSub URL: ws://192.168.1.123:8900
⠐ 00:39:56 | Processed Slot: 326607952 | Confirmed Slot: 326607952 | Finalized Slot: 326607920 | Full Snapshot Slot: 326584258 | Incremental Snapshot Slot: 3266
```

## Setting up Reverse Proxy
As mentioned in the Important Notes section, the Solana RPC port is only accessible from localhost. To make it accessible from the outside world, you can setup a reverse proxy using Nginx for example.
It's up to you how you configure Nginx.

## Final checks
To verify that the Solana node is running correctly, and your reverse proxy is working, you can run the following command (from another machine):

```bash
curl -X POST \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc": "2.0","id": 1,"method": "getHealth"}' \
  'https://my-solana-node.com'
```

You should see the following output:

```json
{"jsonrpc":"2.0","result":"ok","id":1}
```


## License
[MIT](./LICENCE)

---

<p align="center">
    Made with 💚 in Berlin
    <br>
    Crafted with care by Chainflip Labs
</p>
