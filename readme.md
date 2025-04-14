# Pricechain CPRC deployment script -- First Node

```
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential jq -y
```

## Install Golang:

## Install latest go version https://golang.org/doc/install
```
wget -q -O - https://raw.githubusercontent.com/canha/golang-tools-install-script/master/goinstall.sh | bash -s -- --version 1.18
source ~/.profile
```

## To verify that Golang installed
```
go version
```
// Should return go version go1.18 linux/amd64

## Clone Master Repository
git clone https://github.com/PriceChain/cprc.git

## Install the executables

```
sudo rm -rf ~/.cprc
cd cprc
make install

clear

mkdir -p ~/.cprc/upgrade_manager/upgrades
mkdir -p ~/.cprc/upgrade_manager/genesis/bin
```

## Symlink genesis binary to upgrade
```
cp $(which cprcd) ~/.cprc/upgrade_manager/genesis/bin
sudo cp $(which cprcd-manager) /usr/bin
```

## Initialize the validator, where "validator" is a moniker name
```
cprcd init validator --chain-id test
```
 
## Validator
// price17zc58s96rxj79jtqqsnzt3wtx3tern6areu43g
```
echo "mnemonic1" | cprcd keys add validator --keyring-backend test --recover
```

## Validator1
// price14u53eghrurpeyx5cm47vm3qwugtmhcpnstfx9t
```
echo "mnemonic2" | cprcd keys add validator1 --keyring-backend test --recover
```

## Test 1
// price1dfjns5lk748pzrd79z4zp9k22mrchm2a5t2f6u
```
echo "mnemonic3" | cprcd keys add test1 --keyring-backend test --recover
```

## Add genesis accounts
```
cprcd add-genesis-account $(cprcd keys show validator -a --keyring-backend test) 8800000000000000uprc
cprcd add-genesis-account $(cprcd keys show validator1 -a --keyring-backend test) 1000000000000000uprc
cprcd add-genesis-account $(cprcd keys show test1 -a --keyring-backend test) 1000000000000000uprc
```

## Generate CreateValidator signed transaction
```
cprcd gentx validator 1000000000000000uprc --keyring-backend test --chain-id test
```

## Collect genesis transactions
```
cprcd collect-gentxs
```

## Replace stake to PRC
```
sed -i 's/stake/uprc/g' ~/.cprc/config/genesis.json
```

## Create the service file "/etc/systemd/system/rd_netd.service" with the following content
```
sudo nano /etc/systemd/system/cprcd.service
```

## Paste following content
```
[Unit]
Description=cprcd
Requires=network-online.target
After=network-online.target

[Service]
Restart=on-failure
RestartSec=3
User=ubuntu
Group=ubuntu
Environment=DAEMON_NAME=cprcd
Environment=DAEMON_HOME=/home/ubuntu/.cprc
Environment=DAEMON_ALLOW_DOWNLOAD_BINARIES=on
Environment=DAEMON_RESTART_AFTER_UPGRADE=on
PermissionsStartOnly=true
ExecStart=/usr/bin/cprcd-manager start --pruning="nothing" --rpc.laddr "tcp://0.0.0.0:26657"
StandardOutput=file:/var/log/cprcd/cprcd.log
StandardError=file:/var/log/cprcd/cprcd_error.log
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGTERM
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

## Create log files for rd_netd
```
make log-files

sudo systemctl enable cprcd
sudo systemctl start cprcd
```
# Pricechain R&D Net deployment script - Next Nodes

```
sudo apt update
sudo apt upgrade -y
sudo apt install build-essential jq -y
```

## Install Golang:

## Install latest go version https://golang.org/doc/install
```
wget -q -O - https://raw.githubusercontent.com/canha/golang-tools-install-script/master/goinstall.sh | bash -s -- --version 1.18
source ~/.profile
```

## To verify that Golang installed
```
go version
```
// Should return go version go1.18 linux/amd64

## Clone Master Repository
git clone https://github.com/go-bruna/continental-price-network.git

## Install the executables

```
sudo rm -rf ~/.cprc
cd RD_Net_Cosmos
make install

clear

mkdir -p ~/.cprc/upgrade_manager/upgrades
mkdir -p ~/.cprc/upgrade_manager/genesis/bin
```

## Symlink genesis binary to upgrade
```
cp $(which rd_netd) ~/.cprc/upgrade_manager/genesis/bin
sudo cp $(which rd_netd-manager) /usr/bin
```

## Initialize the validator1, where "validator1" is a moniker name (This has to be different from other nodes in the same network.)
```
rd_netd init validator1 --chain-id test
```

## Validator1 (Add new wallet & Buy PRC Coin to stake, but here is just using a genesis account.)
// price14u53eghrurpeyx5cm47vm3qwugtmhcpnstfx9t
```
echo "mnemonic4" | rd_netd keys add validator1 --keyring-backend test --recover
```

## Fetch genesis configuration from the first node deployed.
curl http://18.144.22.147:26657/genesis? | jq ".result.genesis" > ~/.cprc/config/genesis.json

## Change seeds item in ~/.cprc/config/config.toml file.(Format: node_id@peer_node_ip:26656")

sudo nano ~/.cprc/config/config.toml

Modify 'seeds = ""' to:

seeds = "id@xx.xx.xx.xx:26656"
where id = validator id,  xx = validator address.

## Replace stake to PRC
```
sed -i 's/stake/uprc/g' ~/.cprc/config/genesis.json
```

## Create the service file "/etc/systemd/system/rd_netd.service" with the following content
```
sudo nano /etc/systemd/system/rd_netd.service
```

## Paste following content
```
[Unit]
Description=rd_netd
Requires=network-online.target
After=network-online.target

[Service]
Restart=on-failure
RestartSec=3
User=ubuntu
Group=ubuntu
Environment=DAEMON_NAME=rd_netd
Environment=DAEMON_HOME=/home/ubuntu/.cprc
Environment=DAEMON_ALLOW_DOWNLOAD_BINARIES=on
Environment=DAEMON_RESTART_AFTER_UPGRADE=on
PermissionsStartOnly=true
ExecStart=/usr/bin/rd_netd-manager start --pruning="nothing" --rpc.laddr "tcp://0.0.0.0:26657"
StandardOutput=file:/var/log/rd_netd/rd_netd.log
StandardError=file:/var/log/rd_netd/rd_netd_error.log
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGTERM
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```


## Create log files for rd_netd
```
make log-files

sudo systemctl enable rd_netd
sudo systemctl start rd_netd
```

## Become a validator by staking PRC coin (Be sure to replace <your-wallet-name>, <your-moniker>, <amount_of_uprc>)
rd_netd tx staking create-validator --from validator1 --moniker validator1 --pubkey $(rd_netd tendermint show-validator) --chain-id test --keyring-backend test --amount 800000000000000uprc --commission-max-change-rate 0.01 --commission-max-rate 0.2 --commission-rate 0.1 --min-self-delegation 1 -y
