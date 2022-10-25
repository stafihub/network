# StaFiHub Mainnet
**Notice: This is an upgrade, stop the old stafihubd process before starting**


Install stafihubd v0.2.3:

```shell
git clone --branch v0.2.3 https://github.com/stafihub/stafihub
cd $HOME/stafihub && make install
```

Download genesis and clean state:
```shell
wget -O $HOME/.stafihub/config/genesis.json "https://github.com/stafihub/network/raw/main/mainnets/stafihub-1(dragonberry)/genesis.json"

# Do not skip this step
stafihubd tendermint unsafe-reset-all --home ~/.stafihub
```

Configure your node:
```shell
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ufis\"/" $HOME/.stafihub/config/app.toml
sed -i '/\[grpc\]/{:a;n;/enabled/s/false/true/;Ta};/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.stafihub/config/app.toml
peers="bed296dfadd972ed07cab82c87a0ee5c182dea43@18.136.189.120:26656,045fe6e054a5abe35f5433bd333f0a1b18aa28cf@45.136.28.11:26656,d35d55635093fddb6de22295c8fe31de98efe6ef@5.161.120.176:26656,20c0b45c47426c51b3187aa5dca82d9900c2fb36@5.161.88.157:26656,70230067eb1e668d2566329e727c72c930e54de3@116.202.30.7:26656,03f3cb61c7c472044c37aeededde2ffe8884fa02@159.69.108.86:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stafihub/config/config.toml
```

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

## Explorer
Explorer available [here](https://mintscan.io/stafi).
