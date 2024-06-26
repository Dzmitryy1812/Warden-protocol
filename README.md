# Warden-protocol

![Warden-protocol](https://pbs.twimg.com/media/GIKiCbCXgAAJGQr?format=png&name=large)
Проект предлагает решение для ваших кошельков в разных сетях (включая ЕВМ). Проект предлагает решение для ваших кошельков в разных сетях (включая ЕВМ). На данный момент запущена тестовая сеть под названием - ```Alfama```

Details

Network Chain ID: ```alfama```
Denom:``` uward```
Binary: ```wardend```
Working directory: ```.warden```

[Docs] (https://docs.wardenprotocol.org/learn/spaceward/tutorial-spaceward)


[Docs vallidators] (https://docs.wardenprotocol.org/category/validate)


[Faucet] (https://spaceward.alfama.wardenprotocol.org)

- **Требования**:
-250 GB SSD storage
-Quad core CPU less than 10 years old if self hosting
-Dual core CPU works if hosted with newer Xeons / EPYC
-16 GB of ram,  4+ GB of virtual memory recommended
-Hosting: 8 GB RAM + 8 GB Virtual Memory

Update system
```bash
sudo apt update
sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```

Install Go

```bash
rm -rf $HOME/go
sudo rm -rf /usr/local/go
cd $HOME
curl https://dl.google.com/go/go1.20.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
go version
```

Installation & Configuration

Build the wardend binary and initalize the chain home folder:

```bash
git clone --depth 1 --branch v0.1.0 https://github.com/warden-protocol/wardenprotocol/
cd  wardenprotocol/warden/cmd/wardend
go build
sudo mv wardend /usr/local/bin/
wardend init <custom_moniker>
```

Prepare the genesis file:

```bash
d $HOME/.warden/config
rm genesis.json
wget https://raw.githubusercontent.com/warden-protocol/networks/main/testnet-alfama/genesis.json
```

#Set minimum gas price & peers
 
```bash
sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.0025uward"/' app.toml
sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.0025uward"/' app.toml
sed -i 's/persistent_peers = ""/persistent_peers = "6a8de92a3bb422c10f764fe8b0ab32e1e334d0bd@sentry-1.alfama.wardenprotocol.org:26656,7560460b016ee0867cae5642adace5d011c6c0ae@sentry-2.alfama.wardenprotocol.org:26656,24ad598e2f3fc82630554d98418d26cc3edf28b9@sentry-3.alfama.wardenprotocol.org:26656"/' config.toml
```

Setup state sync

```bash
export SNAP_RPC_SERVERS="https://rpc.sentry-1.alfama.wardenprotocol.org:443,https://rpc.sentry-2.alfama.wardenprotocol.org:443,https://rpc.sentry-3.alfama.wardenprotocol.org:443"
export LATEST_HEIGHT=$(curl -s "https://rpc.alfama.wardenprotocol.org/block" | jq -r .result.block.header.height)
export BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
export TRUST_HASH=$(curl -s "https://rpc.alfama.wardenprotocol.org/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
```

Check that all variables have been set correctly:

```bash
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```

#output should be similar to:
#70694 68694 6AF4938885598EA10C0BD493D267EF363B067101B6F81D1210B27EBE0B32FA2A

Now you can add the state sync configuration to your config.toml:

```sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC_SERVERS\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.warden/config/config.toml```
Create service

```bash
sudo tee /etc/systemd/system/wardend.service > /dev/null <<EOF
    [Unit]
    Description=Warden Protocol
    After=network-online.target
    [Service]
    User=root
    ExecStart=$(which wardend) start
    Restart=always
    RestartSec=3
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF
    cd $HOME
    sudo systemctl daemon-reload
    sudo systemctl enable wardend
```
Launch Node

```bash
sudo systemctl restart wardend && sudo journalctl -u wardend -f --no-hostname -o cat
cd $HOME && \
rm -rf .warden wardenprotocol && \
rm -rf $(which wardend)```


Create a validator
If you want to create a validator in the testnet, follow the instructions in the Creating a validator section.

Create Wallet

```bash
wardend keys add wallet
```

faucet token

```bash
curl --json '{"address": "<your-address>"}' https://faucet.alfama.wardenprotocol.org
```

Check Banlance

```bash
wardend q bank balances $(wardend keys show wallet -a)
```

Create a new validator
Once the node is synced and you have the required WARD, you can become a validator.

To create a validator and initialize it with a self-delegation, you need to create a validator.json file and submit a create-validator transaction.

Obtain your validator public key by running the following command:

```bash
wardend comet show-validator
```

The output will be similar to this (with a different key):

```bash
{"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="}
```
Create validator.json file

```bash
nano validator.json
```

The validator.json file has the following format: Change your personal information accordingly
```bash
{    
"pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="},
"amount": "1000000uward",
"moniker": "your-node-moniker",
"identity": "eqlab testnet validator",
"website": "optional website for your validator",
"security": "optional security contact for your validator",
"details": "optional details for your validator",
"commission-rate": "0.1",
"commission-max-rate": "0.2",
"commission-max-change-rate": "0.01",
"min-self-delegation": "1"
}
```
Finally, we're ready to submit the transaction to create the validator:

```bash
wardend tx staking create-validator validator.json \
--from=<key-name> \
--chain-id=alfama \
--fees=500uward
```

Delegate Token to your own validator

```bash
wardend tx staking delegate $(wardend keys show wallet --bech val -a)  1000000uward \
    --from=wallet \
    --chain-id=alfama \
    --fees=500uward
```
Withdraw rewards and commission from your validator

```bash
wardend tx distribution withdraw-rewards $(wardend keys show wallet --bech val -a) \
    --from wallet \
    --commission \
    --chain-id=alfama \
    --fees=500uward
```
Unjail validator
```bash
wardend tx slashing unjail \
    --from=wallet \
    --chain-id=alfama \
    --fees=500uward
```
Services Management

Stop Service
```bash
sudo systemctl stop wardend
```


Restart Service
 ```bash
 sudo systemctl restart wardend
```


Check Service Status
```bash
sudo systemctl status wardend
 Reload Service
 
```bash
sudo systemctl daemon-reload
```

 Enable Service
```bash
sudo systemctl enable wardend
```

Disable Service
```bash
sudo systemctl disable wardend
```

Start Service
```bash
sudo systemctl start wardend
```



```


Check Service Logs
```bash
sudo journalctl -u wardend -f --no-hostname -o cat
```


Backup Validator
```bash
cat $HOME/.warden/config/priv_validator_key.json
```


Remove node
```bash
sudo systemctl stop wardend && sudo systemctl disable wardend && sudo rm /etc/systemd/system/wardend.service && sudo systemctl daemon-reload && rm -rf $HOME/.warden && $HOME/wardenprotocol```

Query Wallet Balance
```bash
wardend q bank balances $(wardend keys show wallet -a)
```
Manual Upgrade

```bash
sudo systemctl stop wardend

cd $HOME && rm -rf wardenprotocol
git clone https://github.com/warden-protocol/wardenprotocol
cd  wardenprotocol
git checkout v0.2.0
make install-wardend

sudo systemctl start wardend
sudo journalctl -u wardend -f --no-hostname -o cat
```
Edit Existing Validator
```bash
wardend tx staking edit-validator \
--new-moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="Hi World 😉" \
--chain-id=alfama \
--commission-rate=0.1 \
--from=wallet \
--gas-prices=0.01uward \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```
