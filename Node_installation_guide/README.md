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
  ver="1.19.3"
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
git checkout v0.2.6
make install
```


## Setting up vars
You should replace values in <> <br />
<YOUR_MONIKER> Here you should put name of your moniker (validator) that will be visible in explorer <br />
<YOUR_WALLET> Here you shoud put the name of your wallet

```
echo "export DEFUND_WALLET="<YOUR_WALLET_NAME>"" >> $HOME/.bash_profile
echo "export DEFUND_NODENAME="<YOUR_MONIKER>"" >> $HOME/.bash_profile
echo "export DEFUND_CHAIN_ID="orbit-alpha-1"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```


## Configure your node
```
defundd config chain-id $DEFUND_CHAIN_ID
```

## Initialize your node
```
defundd init $DEFUND_NODENAME --chain-id $DEFUND_CHAIN_ID
```

## Download genesis
```
wget -O $HOME/.defund/config/genesis.json "https://raw.githubusercontent.com/defund-labs/testnet/main/orbit-alpha-1/genesis.json"
```

## Check genesis.json file
```
# check genesis sha sum
sha256sum ~/.defund/config/genesis.json
# output must be: 58916f9c7c4c4b381f55b6274bce9b8b8d482bfb15362099814ff7d0c1496658
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
SEEDS="f902d7562b7687000334369c491654e176afd26d@170.187.157.19:26656,2b76e96658f5e5a5130bc96d63f016073579b72d@rpc-1.defund.nodes.guru:45656"
PEERS="f902d7562b7687000334369c491654e176afd26d@170.187.157.19:26656,f8093378e2e5e8fc313f9285e96e70a11e4b58d5@rpc-2.defund.nodes.guru:45656,878c7b70a38f041d49928dc02418619f85eecbf6@rpc-3.defund.nodes.guru:45656,3594b1f46c6321d9f99cda8ad5ef5a367ce06ccf@199.247.16.116:26656"
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

### [State Sync guide](https://github.com/L0vd/DeFund/tree/main/State_Sync)


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
  --chain-id=$DEFUND_CHAIN_ID\
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
defundd tx bank send <FROM_WALLET_ADDRESS> <TO_WALLET_ADDRESS> <AMOUNT>ufetf --fees 0ufetf
```

### Delegate tokens to validator
```
defundd tx staking delegate <MONIKER> <AMOUNT>ufetf --from $DEFUND_WALLET --chain-id $DEFUND_CHAIN_ID --fees 0ufetf
```

### Vote for proposal
#### Yes
```
defundd tx gov vote <PROPOSAL_NUMBER> yes --from $DEFUND_WALLET --chain-id $DEFUND_CHAIN_ID --fees 0ufetf
```
#### No
```
defundd tx gov vote <PROPOSAL_NUMBER> no --from $DEFUND_WALLET --chain-id $DEFUND_CHAIN_ID --fees 0ufetf
```
