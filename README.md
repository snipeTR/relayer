```bash
git clone https://github.com/cosmos/relayer.git
cd relayer && git checkout v2.0.0
make build
cp $HOME/relayer/build/rly /usr/local/bin
rly config init --memo "discord#1234"
```

*********
rpc-addr: https://gaia.poolparty.stridenet.co:445
rpc-addr: http://stride-testnet.nodejumper.io:28657
*********



```bash
mkdir $HOME/.relayer/chains
```

```bash
sudo tee $HOME/.relayer/chains/stride.json > /dev/null <<EOF
{
  "type": "cosmos",
  "value": {
    "key": "wallet",
    "chain-id": "STRIDE-TESTNET-2",
    "rpc-addr": "http://${STRIDE_RPC_ADDR}",
    "account-prefix": "stride",
    "keyring-backend": "test",
    "gas-adjustment": 1.2,
    "gas-prices": "0.001ustrd",
    "debug": true,
    "timeout": "20s",
    "output-format": "json",
    "sign-mode": "direct"
  }
}
EOF
```


```bash
sudo tee $HOME/.relayer/chains/gaia.json > /dev/null <<EOF
{
  "type": "cosmos",
  "value": {
    "key": "wallet",
    "chain-id": "GAIA",
    "rpc-addr": "http://${GAIA_RPC_ADDR}",
    "account-prefix": "cosmos",
    "keyring-backend": "test",
    "gas-adjustment": 1.2,
    "gas-prices": "0.001uatom",
    "debug": true,
    "timeout": "20s",
    "output-format": "json",
    "sign-mode": "direct"
  }
}
EOF
```


```bash
rly chains add --file=$HOME/.relayer/chains/stride.json stride
rly chains add --file=$HOME/.relayer/chains/gaia.json gaia
rly chains list
```
*****
1: GAIA             -> type(cosmos) key(✔) bal(✔) path(✔)
2: STRIDE-TESTNET-2 -> type(cosmos) key(✔) bal(✔) path(✔)
*****

```bash
rly keys restore stride wallet "mnimonic keys"
rly keys restore gaia wallet "mnimonic keys"
```

```bash
rly q balance stride
rly q balance gaia
```

Open $HOME/.relayer/config/config.yaml file and replace paths: {} with:
```bash
paths:
    stride-gaia:
        src:
            chain-id: STRIDE-TESTNET-2
            client-id: 07-tendermint-0
            connection-id: connection-0
        dst:
            chain-id: GAIA
            client-id: 07-tendermint-0
            connection-id: connection-0
        src-channel-filter:
            rule: allowlist
            channel-list: [channel-0, channel-1, channel-2, channel-3, channel-4]
```

```bash
rly paths list
```

*****
 0: stride-gaia          -> chns(✔) clnts(✔) conn(✔) (STRIDE-TESTNET-2<>GAIA)
*****

```bash
sudo tee /etc/systemd/system/relayerd.service > /dev/null <<EOF
[Unit]
Description=GO Relayer v2 Service
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which rly) start stride-gaia -p events
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```


```bash
sudo systemctl daemon-reload
systemctl enable relayerd
systemctl start relayerd

journalctl -u relayerd -f -o cat

```
