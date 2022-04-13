# StaFiHub Public Testnet v1

## Hardware Requirements
* **Minimal**
  * 4GB RAM
  * 100GB SSD
  * 2 vCPU
* **Recommended**
  * 8GB RAM
  * 200GB SSD
  * 4 vCPU

## Installation Steps
Install dependencies:
```shell
cd $HOME
sudo apt update
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y < "/dev/null"
```
Install Go:
```shell
cd $HOME
wget -O go1.17.3.linux-amd64.tar.gz https://golang.org/dl/go1.17.3.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.17.3.linux-amd64.tar.gz && rm go1.17.3.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version
```
Clone git repository:
```shell
git clone --depth 1 --branch public-testnet-v1 https://github.com/stafihub/stafihub
```
Install:
```shell
cd $HOME/stafihub && make install
```

Download genesis (replace `YOUR_NODE_NAME`):
```shell
stafihubd init YOUR_NODE_NAME --chain-id stafihub-public-testnet-1
wget -O $HOME/.stafihub/config/genesis.json "https://raw.githubusercontent.com/stafihub/network/main/testnets/stafihub-public-testnet-1/genesis.json"
```
Configure your node:
```shell
sed -i.bak -e "s/^minimum-gas-prices = \"\"/minimum-gas-prices = \"0.01ufis\"/" $HOME/.stafihub/config/app.toml
sed -i '/\[grpc\]/{:a;n;/enabled/s/false/true/;Ta};/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.stafihub/config/app.toml
external_address=`curl ifconfig.me`
peers="694b556323f87e124060488ef2dcf9bab6539988@46.38.241.169:26666"
sed -i.bak -e "s/^external_address = \"\"/external_address = \"$external_address:26656\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stafihub/config/config.toml
```
Install service:
```shell
echo "[Unit]
Description=StaFiHub Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which stafihubd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/stafihubd.service
sudo mv $HOME/stafihubd.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable stafihubd
sudo systemctl restart stafihubd
```
Check your node logs:
```shell
journalctl -u stafihubd -f
```
## Generate keys
```shell
stafihubd keys add YOUR_WALLET_NAME
```
You can recover your keys with `--recover` flag if you have mnemonic

## Faucet
You can ask for tokens in the [#faucet](https://discord.gg/KXMt24cb) Discord channel.
```shell
!faucet send YOUR_WALLET_ADDRESS
```
## Create validator
Use the following command (do not forget to replace `YOUR_NODE_NAME` and `YOUR_WALLET_NAME`):
```shell
stafihubd tx staking create-validator -y --amount=1000000ufis --pubkey=$(stafihubd tendermint show-validator) --moniker=YOUR_NODE_NAME --commission-rate=0.10 --commission-max-rate=0.20 --commission-max-change-rate=0.01 --min-self-delegation=1 --from=YOUR_WALLET_NAME --chain-id=stafihub-public-testnet-1 --fees 1000ufis
```

## Explorer
Explorer available [here](https://testnet-explorer.stafihub.io).
