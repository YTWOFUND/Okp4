# Okp4

# Okp4
Okp4 Node Installation Instructions </br>
### [Official documentation](https://docs.okp4.network)

### System requirements: </br>
CPU: 4 Core </br>
RAM: 8 Gb </br>
SSD: 200 Gb </br>
OS: Ubuntu 20.04 LTS </br>

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>
    
# Manual configuration of the Lava node with Cosmovisor
Required packages installation </br>
```
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl git jq lz4 build-essential
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```

Go installation.
```
cd $HOME
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd && rm -rf okp4d
git clone https://github.com/okp4/okp4d
cd okp4d
git checkout v6.0.0
```

# Build binary
```
make install
```

# Set node CLI configuration
```
okp4d config chain-id okp4-drunemeton-1
okp4d config keyring-backend test
okp4d config node tcp://localhost:17657
```

# Initialize the node
```
okp4d init "Your Node Name" --chain-id okp4-drunemeton-1
```

# Download genesis and addrbook files
```
curl -L https://snapshots-testnet.nodejumper.io/okp4-testnet/genesis.json > $HOME/.okp4d/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/okp4-testnet/addrbook.json > $HOME/.okp4d/config/addrbook.json
```

# Set seeds
```
sed -i -e 's|^seeds *=.*|seeds = "a7f1dcf7441761b0e0e1f8c6fdc79d3904c22c01@seeds.cros-nest.com:36656,20e1000e88125698264454a884812746c2eb4807@testnet-seeds.lavenderfive.com:17656"|' $HOME/.okp4d/config/config.toml
```

# Set minimum gas price
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01uknow"|' $HOME/.okp4d/config/app.toml
```

# Set pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.okp4d/config/app.toml
```

# Change ports
```
sed -i -e "s%:1317%:17617%; s%:8080%:17680%; s%:9090%:17690%; s%:9091%:17691%; s%:8545%:17645%; s%:8546%:17646%; s%:6065%:17665%" $HOME/.okp4d/config/app.toml
sed -i -e "s%:26658%:17658%; s%:26657%:17657%; s%:6060%:17660%; s%:26656%:17656%; s%:26660%:17661%" $HOME/.okp4d/config/config.toml
```

# Download latest chain data snapshot
```
curl "https://snapshots-testnet.nodejumper.io/okp4-testnet/okp4-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.okp4d"
```

# Create a service
```
sudo tee /etc/systemd/system/okp4d.service > /dev/null << EOF
[Unit]
Description=OKP4 node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which okp4d) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable okp4d.service
```

# Start the service and check the logs
```
sudo systemctl start okp4d.service
sudo journalctl -u okp4d.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
okp4d keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
okp4d keys add wallet --recover
```

### We receive tokens from the tap in the site(https://faucet.okp4.network)

### Create the Validator

Before creating a validator, enter the command and check that you have false. This means that the Node has synchronized and you can create a validator:
```
okp4d status 2>&1 | jq .SyncInfo.catching_up
```

### Check the balance before creating for the presence of tokens
```
okp4d q bank balances $(okp4d keys show wallet -a)
```

Please enter your details below in quotation marks where required

```
okp4d tx staking create-validator \
--amount=1000000uknow \
--pubkey=$(okp4d tendermint show-validator) \
--moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO" \
--chain-id=okp4-drunemeton-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.01uknow \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
There have been no updates at the moment, as soon as they come out, we will immediately add them to this section.

Current network:okp4-drunemeton-1
Current version:v6.0.0
```

### Useful commands

Check balance
```
okp4d q bank balances $(okp4d keys show wallet -a)
```

CHECK SERVICE LOGS
```
sudo journalctl -u okp4d -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart okp4d
```

GET VALIDATOR INFO
```
okp4d status 2>&1 | jq .ValidatorInfo
```

DELEGATE TOKENS TO YOURSELF
```
okp4d tx staking delegate $(okp4d keys show wallet --bech val -a) 1000000uknow --from wallet --chain-id okp4-drunemeton-1 --gas-prices 0.01uknow --gas-adjustment 1.5 --gas auto -y
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop okp4d && sudo systemctl disable okp4d && sudo rm /etc/systemd/system/okp4d.service && sudo systemctl daemon-reload && rm -rf $HOME/.okp4d && rm -rf okp4d && sudo rm -rf $(which okp4d) 
```
