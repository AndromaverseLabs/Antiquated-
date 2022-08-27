# Node and Validator Setup Docs

This is Andromachain's documentation to set up a node and validator.

## Prerequisites

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install make build-essential git jq chrony -y
sudo apt install gcc -y
```

## Increase open files limit

```bash
sudo su -c "echo 'fs.file-max = 65536' >> /etc/sysctl.conf"
sudo sysctl -p
```

## Install Go

```bash
curl https://dl.google.com/go/go1.18.3.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf

OR

curl https://raw.githubusercontent.com/canha/golang-tools-install-script/master/goinstall.sh | bash
```

## Update environment variables

```bash
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export GOBIN=$HOME/go/bin
export PATH=$PATH:/usr/local/go/bin:$GOBIN
EOF
```

```bash
source $HOME/.profile
```

## Download Ignite

```bash
curl https://get.ignite.com/cli! | bash
```

Or

```bash
https://docs.ignite.com/guide/install
```

## Clone source repository

```bash
git clone https://github.com/AndromaverseLabs/ChainFiles
cd ChainFiles
git checkout <andromaverse binary latest version>
```

## Build the chain

```bash
ignite chain build
```

Initialize the chain

```bash
andromad init <MONIKER> --chain-id test-chain-androma-1
```

## Set up persistent peers

```bash
nano $HOME/.andromad/config/config.toml
```

Add peers

```

persistent_peers = <validator-generated persistent peer>
```

## Copy genesis file

```bash
curl https://raw.githubusercontent.com/AndromaverseLabs/testnet/main/genesis.json > ~/.andromad/config/genesis.json
```

## Starting the network

Now start the network with a terminal command to ensure it runs

```bash
andromad start
```

## Create andromad.service

You should create a service to ensure the node can run in the background. First, exit from the terminal command "andromad start". Ctrl-C will do.

Create andromad.service file with the following. Make sure to replace USER and HOME placeholders

```bash
sudo tee /etc/systemd/system/andromad.service > /dev/null <<EOF  
[Unit]
Description=Andromad Node
After=network-online.target

[Service]
User=<USER>
ExecStart=$(which andromad) start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

## Move file to systemd folder and enable the service

```bash
sudo mv andromad.service /etc/systemd/system/andromad.service
sudo systemctl enable andromad.service && sudo systemctl start andromad.service
```

## Check node info

```bash
curl -s localhost:26657/status | jq .result.sync_info.catching_up
#true output is syncing - false is synced
curl -s localhost:26657/status | jq .result.sync_info.latest_block_height
#this output is your last block synced
curl -s "http://:26657/status?" | jq .result.sync_info.latest_block_height
#this output the public node last block synced
```

## Create or recover keys

```bash
# Create new key
andromad keys add <KEY_NAME> --keyring-backend os
```

```bash
# Recover key
andromad keys add <MONIKER> --keyring-backend os —-recover
```

## Get test tokens

TBD

## Create Validator

```bash
andromad tx staking create-validator \
 --amount=100000000uandr \
 --pubkey=$(andromad tendermint show-validator) \
 --moniker=<your-moniker> \
 --chain-id=test-chain-androma-1 \
 --commission-rate="0.05" \
 --commission-max-rate="0.10" \
 --commission-max-change-rate="0.05" \
 --min-self-delegation="1" \
 --gas=auto \
 --gas-adjustment=1.5 \ 
 --gas-prices=0.025uandr
 --from=(key of your wallet address)
```
