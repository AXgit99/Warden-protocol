**Clone project repository**
```
cd $HOME && rm -rf wardenprotocol
git clone https://github.com/warden-protocol/wardenprotocol
cd  wardenprotocol
git checkout v0.3.0
```

**Build binary**
```
make install-wardend
```

**Set node CLI configuration**
```
wardend config set client chain-id buenavista-1
wardend config set client keyring-backend test
wardend config set client node tcp://localhost:26657
```

**Initialize the node**
```
wardend init "Your Node Name" --chain-id buenavista-1
```

**Download genesis and addrbook files**
```
curl -L https://snapshots-testnet.nodejumper.io/wardenprotocol-testnet/genesis.json > $HOME/.warden/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/wardenprotocol-testnet/addrbook.json > $HOME/.warden/config/addrbook.json
```

**Set seeds**
```
sed -i -e 's|^seeds *=.*|seeds = "ddb4d92ab6eba8363bab2f3a0d7fa7a970ae437f@sentry-1.buenavista.wardenprotocol.org:26656,c717995fd56dcf0056ed835e489788af4ffd8fe8@sentry-2.buenavista.wardenprotocol.org:26656,e1c61de5d437f35a715ac94b88ec62c482edc166@sentry-3.buenavista.wardenprotocol.org:26656"|' $HOME/.warden/config/config.toml
```
**Set minimum gas price**
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01uward"|' $HOME/.warden/config/app.toml
```

**Set pruning**
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.warden/config/app.toml
```

**Download latest chain data snapshot**
```
curl "https://snapshots-testnet.nodejumper.io/wardenprotocol-testnet/wardenprotocol-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.warden"
```

**Create a service**
```
sudo tee /etc/systemd/system/wardend.service > /dev/null << EOF
[Unit]
Description=Warden Protocol node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which wardend) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable wardend.service
```

**Start the service and check the logs**
```
sudo systemctl start wardend.service
sudo journalctl -u wardend.service -f --no-hostname -o cat
```
