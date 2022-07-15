# docs

Andromachain's documentation

# Node and Validator Setup

## Prerequisites

sudo apt-get update && sudo apt upgrade -y
sudo apt-get install make build-essential git jq chrony -y

sudo apt install gcc

## Increase open files limit

sudo su -c "echo 'fs.file-max = 65536' >> /etc/sysctl.conf"
sudo sysctl -p

## Install Go

curl https://dl.google.com/go/go1.17.7.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf

## Update environment variables

cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export GOBIN=$HOME/go/bin
export PATH=$PATH:/usr/local/go/bin:$GOBIN
EOF
source $HOME/.profile

## Download Ignite

curl https://get.ignite.com/cli! | bash
Or
https://docs.ignite.com/guide/install

## Clone source repository

git clone <andromaverse repo link>
cd <andromaverse binary>
git checkout <andromaverse binary latest version>

## Build the chain

ignite chain build

Initialize the chain

andromaversed init <MONIKER> --chain-id <chain id>

cd $HOME/.andromaversed/config/

## Create or recover keys

andromaversed keys add <KEY_NAME> --keyring-backend os

andromaversed keys add <MONIKER> --keyring-backend os —recover

## Create environment variable

MY_VALIDATOR_ADDRESS=$(andromaversed keys show <KEY_NAME> -a --keyring-backend os)

## Set up persistent peers

vim config.toml

persistent_peers = <validator-generated persistent peer>

## Copy genesis file

git clone <repo link>
cd testnets/<chain id>
cp genesis.json $HOME/. andromaversed/config

## Starting the network

andromaversed start

## Create Validator

andromaversed tx staking create-validator \
 --amount=100000000 uandr \
 --pubkey=$(andromaversed tendermint show-validator) \
 --moniker=<"your-moniker"> \
 —chain-id=<chain id> \
 --commission-rate="0.05" \
 --commission-max-rate="0.20" \
 --commission-max-change-rate="0.01" \
 --min-self-delegation="1" \
 --gas="700000" \
 --from=(keyofyourvalidator)

## Activate andromaversed.service

[Unit]
Description=Andromaversed Node
After=network-online.target

[Service]
User=root
ExecStart=/root/go/bin/andromaversed start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target

## Move file to systemd folder

sudo mv /etc/systemd/system/andromaversed.service /lib/systemd/system/
sudo systemctl enable andromaversed.service && sudo systemctl start andromaversed.service

## Check node info

curl -s localhost:26657/status | jq .result.sync_info.catching_up
#true output is syncing - false is synced
curl -s localhost:26657/status | jq .result.sync_info.latest_block_height
#this output is your last block synced
curl -s "http://:26657/status?" | jq .result.sync_info.latest_block_height
#this output the public node last block synced
