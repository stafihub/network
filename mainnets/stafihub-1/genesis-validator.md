# StaFiHub Mainnet

All the steps genesis validators need to do after we release the mainnet version.
1. Please init your node and submit PR with Gentx before 2022-08-14T13:00:00.
2. We will release the final genesis file before 2022-08-15T04:00:00.
3. Please download the final genesis file and config your node before 2022-08-17T04:00:00.
4. We will launch the mainnet together at 2022-08-17T13:00:00.

## Hardware Requirements

* **Minimal**
  * 4GB RAM
  * 600GB SSD
  * 2 vCPU
* **Recommended**
  * 8GB RAM
  * 1T SSD
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
wget -O go1.18.2.linux-amd64.tar.gz https://go.dev/dl/go1.18.2.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz && rm go1.18.2.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version
```


Clone git repository:
```shell
git clone --branch <latest-release-tag> https://github.com/stafihub/stafihub
```

Install:
```shell
cd $HOME/stafihub && make install
```

Init your node (replace `YOUR_NODE_NAME`):
```shell
stafihubd init YOUR_NODE_NAME --chain-id stafihub-1
```

#### Generate keys
```shell
stafihubd keys add YOUR_WALLET_NAME --keyring-backend file
```
You can recover your keys with `--recover` flag if you have mnemonic. If you generate a new key, please remember to backup your mnemonic.

#### Add genesis account:
Note: We will reserve 5 FIS to your account in the genesis file.

```
stafihubd add-genesis-account YOUR_WALLET_NAME 5000000ufis --keyring-backend file
```

#### Create Gentx
Note: The amount should be less than 5 FIS.

```
stafihubd gentx YOUR_WALLET_NAME 1000000ufis \
--chain-id stafihub-1 \
--moniker=YOUR_NODE_NAME \
--commission-max-change-rate=0.01 \
--commission-max-rate=0.20 \
--commission-rate=0.10 \
--details="<details>" \
--website="<website>" \
--keyring-backend file
```

#### Submit PR with Gentx
1. Copy the contents of ${HOME}/.stafihub/config/gentx/gentx-XXXXXXXX.json.
2. Fork the repository.
3. Create a file gentx-{{VALIDATOR_NAME}}.json under the /gentxs folder in the forked repo, paste the copied text into the file.
4. Create a Pull Request to the main branch of the repository.
5. Post the PR link to the discord group(stafihub-genesis-validator).


## Download the final genesis file and config your node

Configure your node:
```shell
wget -O $HOME/.stafihub/config/genesis.json "https://raw.githubusercontent.com/stafihub/network/main/mainnets/stafihub-1/genesis.json"

sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ufis\"/" $HOME/.stafihub/config/app.toml
sed -i '/\[grpc\]/{:a;n;/enabled/s/false/true/;Ta};/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.stafihub/config/app.toml
peers="bed296dfadd972ed07cab82c87a0ee5c182dea43@18.136.189.120:26656,045fe6e054a5abe35f5433bd333f0a1b18aa28cf@45.136.28.11:26656,d35d55635093fddb6de22295c8fe31de98efe6ef@5.161.120.176:26656,20c0b45c47426c51b3187aa5dca82d9900c2fb36@5.161.88.157:26656,70230067eb1e668d2566329e727c72c930e54de3@116.202.30.7:26656,03f3cb61c7c472044c37aeededde2ffe8884fa02@159.69.108.86:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stafihub/config/config.toml
```


## Start your node
#### Note: Please wait for the final launch time.

Start the node in the background:
```shell
stafihubd start
```


Or install service to run the node:
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