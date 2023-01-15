<p style="font-size:14px" align="right">
<a href="https://t.me/L0vd_staking" target="_blank">Join our telegram <img src="https://raw.githubusercontent.com/L0vd/screenshots/main/Telegram_logo.png" width="30"/></a>
<a href="https://l0vd.com/" target="_blank">Visit our website <img src="https://raw.githubusercontent.com/L0vd/screenshots/main/L0vd.png" width="30"/></a>
</p>



# Table of contents <br />
[Node setup](#node_setup) <br />
[State Sync](#state_sync) <br />
[Starting a validator](#starting_validator) <br />
[Useful commands](#useful_commands)



<a name="node_setup"></a>
# Manual node setup
If you want to setup DeFund fullnode manually follow the steps below

## Update and upgrade
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```

## Install GO
```
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.3"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Install node
```
cd $HOME && rm -rf defund
git clone https://github.com/defund-labs/defund.git
cd defund
git checkout v0.2.2
make install
```


## Setting up vars
You should replace values in <> <br />
<YOUR_MONIKER> Here you should put name of your moniker (validator) that will be visible in explorer <br />
<YOUR_WALLET> Here you shoud put the name of your wallet

```
echo "export DEFUND_WALLET="<YOUR_WALLET_NAME>" >> $HOME/.bash_profile
echo "export DEFUND_NODENAME="<YOUR_MONIKER>" >> $HOME/.bash_profile
echo "export DEFUND_CHAIN_ID="defund-private-4"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```


## Configure your node
```
defundd config chain-id $DEFUND_CHAIN_ID
```

## Initialize your node
```
defundd init $NODENAME --chain-id $CHAIN_ID
```

## Download genesis
```
wget -O $HOME/.defund/config/defund-private-3-genesis.json "https://raw.githubusercontent.com/defund-labs/testnet/main/defund-private-4/genesis.json"
```

## Check genesis.json file
```
# check genesis sha sum
sha256sum ~/.defund/config/genesis.json
# output must be: db13a33fbb4048c8701294de79a42a2b5dff599d653c0ee110390783c833208b
# otherwise you have an incorrect genesis file
```

## (OPTIONAL) Set custom ports

### If you want to use non-default ports
```
DEFUND_PORT=<SET_CUSTOM_PORT> #Example: DEFUND_PORT=56 (numbers from 1 to 64)
```
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${DEFUND_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${DEFUND_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${DEFUND_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${DEFUND_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${DEFUND_PORT}660\"%" $HOME/.defund/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${DEFUND_PORT}317\"%; s%^address = \":8080\"%address = \":${DEFUND_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${DEFUND_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${DEFUND_PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${DEFUND_PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${DEFUND_PORT}546\"%" $HOME/.defund/config/app.toml
```


## Set seeds and peers
```
SEEDS="9f92e47ea6861f75bf8a450a681218baae396f01@94.130.219.37:26656,f03f3a18bae28f2099648b1c8b1eadf3323cf741@162.55.211.136:26656,f8fa20444c3c56a2d3b4fdc57b3fd059f7ae3127@148.251.43.226:56656,70a1f41dea262730e7ab027bcf8bd2616160a9a9@142.132.202.86:17000,e47e5e7ae537147a23995117ea8b2d4c2a408dcb@172.104.159.69:45656,74e6425e7ec76e6eaef92643b6181c42d5b8a3b8@defund-testnet-seed.itrocket.net:443"
PEERS="9f92e47ea6861f75bf8a450a681218baae396f01@94.130.219.37:26656,f03f3a18bae28f2099648b1c8b1eadf3323cf741@162.55.211.136:26656,f8fa20444c3c56a2d3b4fdc57b3fd059f7ae3127@148.251.43.226:56656,70a1f41dea262730e7ab027bcf8bd2616160a9a9@142.132.202.86:17000,e47e5e7ae537147a23995117ea8b2d4c2a408dcb@172.104.159.69:45656,74e6425e7ec76e6eaef92643b6181c42d5b8a3b8@defund-testnet-seed.itrocket.net:443"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.defund/config/config.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.defund/config/app.toml
```

## Set minimum gas price and timeout commit
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ufetf\"/" $HOME/.defund/config/app.toml
```

## Create Service
```
sudo tee /etc/systemd/system/defundd.service > /dev/null <<EOF
[Unit]
Description=DeFund
After=network-online.target

[Service]
User=$USER
ExecStart=$(which defundd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Reset blockchain info and restart your node
```
sudo systemctl daemon-reload
sudo systemctl enable defundd
defundd tendermint unsafe-reset-all --home $HOME/.defund --keep-addr-book
sudo systemctl restart defundd && sudo journalctl -u defundd -f -o cat
```

<a name="state_sync"></a>
## (OPTIONAL) Use State Sync

### [State Sync guide](https://github.com/L0vd/DeFund/tree/main/StateSync)


<a name="starting_validator"></a>
## Starting a validator

### 1. Add a new key
```
defundd keys add $DEFUND_WALLET
```
#### (OR)

### 1. Recover your key
```
defundd keys add $DEFUND_WALLET --recover
```

### 2. Request tokens from discord

### 3. Create validator
```
defundd tx staking create-validator \
  --amount=1000000ufetf \
  --pubkey=$(defundd tendermint show-validator) \
  --moniker=$DEFUND_NODENAME \
  --chain-id=defund-private-4 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1000000" \
  --gas="auto" \
  --from=$DEFUND_WALLET \
--yes
```
<a name="useful_commands"></a>
## Useful commands

### Check status
```
defundd status | jq
```

### Check logs
```
sudo journalctl -u defundd -f
```

### Check wallets
```
defundd keys list
```

### Check balance
```
defundd q bank balances $DEFUND_WALLET
```

### Send tokens
```
defundd tx bank send <FROM_WALLET_ADDRESS> <TO_WALLET_ADDRESS> <AMOUNT>ufetf --gas auto
```

### Delegate tokens to validator
```
defundd tx staking delegate <MONIKER> <AMOUNT>ufetf --from $DEFUND_WALLET --chain-id $DEFUND_CHAIN_ID --gas auto
```

### Vote for proposal
#### Yes
```
defundd tx gov vote <PROPOSAL_NUMBER> yes --from $DEFUND_WALLET --chain-id $DEFUND_CHAIN_ID
```
#### No
```
defundd tx gov vote <PROPOSAL_NUMBER> no --from $DEFUND_WALLET --chain-id $DEFUND_CHAIN_ID
```
