# Warden-protocol
Проект предлагает решение для ваших кошельков в разных сетях (включая ЕВМ). Проект предлагает решение для ваших кошельков в разных сетях (включая ЕВМ). На данный момент запущена тестовая сеть под названием - Alfama
Details
Network Chain ID: alfama
Denom: uward
Binary: wardend
Working directory: .warden

[Docs] (https://docs.wardenprotocol.org/learn/spaceward/tutorial-spaceward)


[Docs vallidators] (https://docs.wardenprotocol.org/category/validate)


[Faucet] (https://spaceward.alfama.wardenprotocol.org)

- **Требования**:
-250 GB SSD storage
-Quad core CPU less than 10 years old if self hosting
-Dual core CPU works if hosted with newer Xeons / EPYC
-16 GB of ram,  4+ GB of virtual memory recommended
-Hosting: 8 GB RAM + 8 GB Virtual Memory

Подготовка сервера

# обновляем репозитории

```sudo apt update

sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```
# устанавливаем необходимые утилиты

```apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

# устанавливаем и копируем конфиг, который будет иметь больший приоритет
```apt install fail2ban -y && \
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local && \
nano /etc/fail2ban/jail.local
```
# раскомментировать и добавить свой IP: ignoreip = 127.0.0.1/8 ::1 <ip>
```systemctl restart fail2ban
```

# проверяем status 
```systemctl status fail2ban
```
# проверяем, какие jails активны (по умолчанию только sshd)
```fail2ban-client status
```
# проверяем статистику по sshd
```fail2ban-client status sshd
```
# смотрим логи
tail /var/log/fail2ban.log
# останавливаем работу и удаляем с автозагрузки
```#systemctl stop fail2ban && systemctl disable fail2ban
```
Устанавливаем GO

```ver="1.21.3" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
Новая установка ноды
ВАЖНО — в командах ниже все, что в <> меняем на свое значение и убираем сами <>
Устанавливаем бинарники
# если не установлен каталог GO, то создаейте его
mkdir -p $HOME/go/bin
git clone --depth 1 --branch v0.2.0 https://github.com/warden-protocol/wardenprotocol
cd wardenprotocol
wget https://github.com/warden-protocol/wardenprotocol/releases/download/v0.2.0/wardend_Linux_x86_64.zip
unzip wardend_Linux_x86_64.zip
rm -rf wardend_Linux_x86_64.zip
chmod +x wardend
mv wardend $HOME/go/bin
```
```wardend version --long | grep -e version -e commit
```
# version: 0.2.0
# commit: 4d37e5aa6eb2a0ee288baf3afbfec0c8b8e2551d

Инициализируем ноду, чтобы создать необходимые файлы конфигурации
```wardend init guide --chain-id alfama
```
#скачиваем genesis
```wget -O $HOME/.warden/config/genesis.json "https://raw.githubusercontent.com/warden-protocol/networks/main/testnet-alfama/genesis.json"
```
# Проверим генезис
```sha256sum ~/.warden/config/genesis.json
```
# f29ce94657e35706d7868bc725a3fbfd7530c1508842e6a339920a45d28e51b3
Проверяем, что состояние валидатора на начальном этапе
cd && cat .warden/data/priv_validator_state.json
{
  "height": "0",
  "round": 0,
  "step": 0
}

# если нет, то выполняем команду
```wardend tendermint unsafe-reset-all --home $HOME/.warden
```
Скачиваем Addr book
```wget -O $HOME/.warden/config/addrbook.json "https://share.utsa.tech/warden/addrbook.json"
```
Настраиваем конфигурацию ноды
Вручную добавьте в client.toml chain-id alfama или в каждую транзакцию добавляйте ключ --chain-id=alfama
# правим конфиг, благодаря чему мы можем больше не использовать флаг chain-id для каждой команды CLI в client.toml
#wardend config chain-id alfama

# при необходимости настраиваем keyring-backend в client.toml 
#wardend config keyring-backend os

# настраиваем минимальную цену за газ в app.toml
```sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025uward\"/;" ~/.warden/config/app.toml
```
# добавляем seeds/bpeers/peers в config.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.warden/config/config.toml

peers="6a8de92a3bb422c10f764fe8b0ab32e1e334d0bd@sentry-1.alfama.wardenprotocol.org:26656,7560460b016ee0867cae5642adace5d011c6c0ae@sentry-2.alfama.wardenprotocol.org:26656,24ad598e2f3fc82630554d98418d26cc3edf28b9@sentry-3.alfama.wardenprotocol.org:26656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" $HOME/.warden/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.warden/config/config.toml

# при необходимости увеличиваем количество входящих и исходящих пиров для подключения, за исключением постоянных пиров в config.toml
```sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 40/g' $HOME/.warden/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.warden/config/config.toml
```
# настраиваем фильтрацию "плохих" peers
```sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.warden/config/config.toml
```
(ОПЦИОНАЛЬНО) Настраиваем прунинг одной командой вapp.toml
```pruning="custom"
pruning_keep_recent="1000"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.warden/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.warden/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.warden/config/app.toml
```
Также можно самостоятельно настроить параметр min-retain-blocks: Это значение относится к обрезке блоков Tendermint. Оно отличается от настроек обычного Pruning. Если min-retain-blocks=0, то ничего не удаляется.
min-retain-blocks определяет минимальное смещение высоты блока от текущего фиксируемого блока, так что все блоки, превышающие это смещение, удаляются из Tendermint
min-retain-blocks=10
inter-block-cache = false
```
(ОПЦИОНАЛЬНО) Выкл индексацию вconfig.toml
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.warden/config/config.toml
(ОПЦИОНАЛЬНО) Вкл/выкл снэпшоты вapp.toml
# По умолчанию снэпшоты выключены "snapshot-interval=0"
snapshot_interval=1000
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.warden/config/app.toml
(ОПЦИОНАЛЬНО) Смена портов #для 2 ноды
# config.toml
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:36658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:36657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:6061\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:36656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":36660\"%" $HOME/.warden/config/config.toml

# app.toml
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:9190\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:9191\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:1327\"%" $HOME/.warden/config/app.toml

# client.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:36657\"%" $HOME/.warden/config/client.toml

external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:36656\"/" $HOME/.warden/config/config.toml
Подробнее о смене портов здесь
(ОПЦИОНАЛЬНО) State Sync
# при необходимости скачиваем wasm
# добавляем пир  ( спасибо Lesnik у)
```peers="225054d5ddf2386762450e21075c0e8677c3d0fc@144.76.29.90:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.warden/config/config.toml
SNAP_RPC=https://rpc.sentry-1.alfama.wardenprotocol.org:443
```
# SNAP_RPC=https://t-warden.rpc.utsa.tech:443  ( спасибо Lesnik у)

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.warden/config/config.toml
systemctl restart wardend && journalctl -u wardend -f -o cat
Важно - для разных блокчейнов нужно разное количество RAM для успешного старта со State sync

# при ошибке очищаем базу данных
systemctl stop wardend
wardend tendermint unsafe-reset-all --home $HOME/.warden --keep-addr-book
Создаем сервисный файл
tee /etc/systemd/system/wardend.service > /dev/null <<EOF
[Unit]
Description=wardend
After=network-online.target

[Service]
User=$USER
ExecStart=$(which wardend) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable wardend
systemctl restart wardend && journalctl -u wardend -f -o cat
Если после старта нода долго не может подцепиться к пирам, то ищем новые пиры или просим addrbook.json в дискорд
# стопаем ноду, удаляем адресную книгу и сбрасываем данные
systemctl stop wardend
wardend tendermint unsafe-reset-all --home $HOME/.warden

# перезагружаем ноду
```systemctl restart wardend && journalctl -u wardend -f -o cat
```
Создаем или восстанавливаем кошелек и сохраняем вывод
# создать кошелек
```wardend keys add <name_wallet> --keyring-backend os
```
# восстановить кошелек (после команды вставить seed)
```wardend keys add <name_wallet> --recover --keyring-backend os
```
# восстановить кошелек для EVM сетей
```wardend keys add <name_wallet> --recover --coin-type 118 --algo secp256k1
```
Не забываем сохранить seed !!!
Создаем валидатора
1. Получаем свой pubkey
wardend comet show-validator
2. Создаем validator.json
nano $HOME/.warden/validator.json
3. Вставляем наш конфиг
```
{
  "pubkey": {#pubkey},
  "amount": "1000000uward",
  "moniker": "STAVR",
  "identity": "",
  "website": "",
  "security": "",
  "details": "",
  "commission-rate": "0.05",
  "commission-max-rate": "0.5",
  "commission-max-change-rate": "0.5",
  "min-self-delegation": "1"
}
```
4. Отправляем транзакцию
```wardend tx staking create-validator $HOME/.warden/validator.json \
    --from=<key-name> \
    --chain-id=alfama \
    --fees=500uward
```
Не забываем сохранить priv_validator_key.json !!!
Подробнее о создании/редактировании валидатора можно почитать здесь
Полезные команды
Для добавления лого в mintscan (ТОЛЬКО ДЛЯ MINTSCAN):
форк https://github.com/cosmostation/chainlist
в папке Moniker находим название проекта
через add file/upload file добавляем свою аватарку. название файла обязательно должно быть валопер.png . и только png
PR
Информация
# проверить блоки
```wardend status 2>&1 | jq ."SyncInfo"."latest_block_height"
```
# проверить логи
```journalctl -u wardend -f -o cat
```
# проверить статус
```curl localhost:$PORT/status | jq
```
# проверить баланс
```wardend q bank balances <address>
```
# проверить pubkey валидатора
```wardend tendermint show-validator
```
# проверить валидатора
```wardend query staking validator <valoper_address>
wardend query staking validators --limit 1000000 -o json | jq '.validators[] | select(.description.moniker=="<name_moniker>")' | jq
```
# проверка информации по TX_HASH
```wardend query tx <TX_HASH>
```
# параметры сети
```wardend q staking params
wardend q slashing params
```
# узнать транзакцию создания валидатора (заменить свой valoper_address)
```wardend query txs --events='create_validator.validator=<your_valoper_address>' -o=json | jq .txs[0].txhash -r
```
# просмотр активного сета
```wardend q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```
# просмотр неактивного сета
```wardend q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```
Транзакции
# собрать реварды со всех валидаторов, которым делегировали (без комиссии)
```wardend tx distribution withdraw-all-rewards --from <name_wallet> --fees=500uward -y
```

# собрать реварды c отдельного валидатора или реварды + комиссию со своего валидатора
```wardend tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees=500uward --commission -y
```
# заделегировать себе в стейк еще (так отправляется 1 монетa)
```wardend tx staking delegate <valoper_address> 1000000uward --from <name_wallet> --fees=500uward -y
```
# ределегирование на другого валидатора
```wardend tx staking redelegate <src-validator-addr> <dst-validator-addr> 1000000uward --from <name_wallet> --fees=500uward -y
```
# unbond 
```wardend tx staking unbond <addr_valoper> 1000000uward --from <name_wallet> --fees=500uward -y
```
# отправить монеты на другой адрес
```wardend tx bank send <name_wallet> <address> 1000000uward --fees=500uward -y
```
# выбраться из тюрьмы
```wardend tx slashing unjail --from <name_wallet> --fees=500uward -y
```
! Если транзакции не отправляются с ошибкой account sequence mismatch, expected 18, got 17: incorrect account sequence, то добавьте в команду ключ -s 18 (номер замените на тот, который ждет sequence)
Работа с кошельками
# вывести список кошельков
```wardend keys list
```
# показать ключ аккаунта
```wardend keys show <name_wallet> --bech acc
```
# показать ключ валидатора
```wardend keys show <name_wallet> --bech val
```
# показать ключ консенсуса
```wardend keys show <name_wallet> --bech cons
```
# показать все поддерживаемые адреса
```wardend debug addr <wallet_addr>
```
# показать приватный ключ
```wardend keys export <name_wallet> --unarmored-hex --unsafe
wardend keys unsafe-export-eth-key <name_wallet>
```
# запрос учетной записи
```wardend q auth account $(wardend keys show <name_wallet> -a) -o text
```
# удалить кошелек
```wardend keys delete <name_wallet>
```
Удалить ноду
```systemctl stop wardend && \
systemctl disable wardend && \
rm /etc/systemd/system/wardend.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .warden wardenprotocol && \
rm -rf $(which wardend)
```
