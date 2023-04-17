<p style="font-size:14px" align="right">
<a href="https://t.me/L0vd_staking" target="_blank">Join our telegram <img src="https://raw.githubusercontent.com/L0vd/screenshots/main/Telegram_logo.png" width="30"/></a>
<a href="https://l0vd.com/" target="_blank">Visit our website <img src="https://raw.githubusercontent.com/L0vd/screenshots/main/L0vd.png" width="30"/></a>
</p>

# DeFund Snapshot

## Install dependencies, if needed
```
sudo apt update
sudo apt install lz4 -y
```

## Sync from Snapshot  
| Height  | Size | Pruning | Indexer | Creation Time (UTC+3) |
| --------- | --------- | --------- | --------- | --------- |
| 3613389  | 67GB  | custom/100/0/10 | null | 2023-04-17_22:06:12 |

```
sudo systemctl stop defundd

cp $HOME/.defund/data/priv_validator_state.json $HOME/.defund/priv_validator_state.json.backup
defundd tendermint unsafe-reset-all --home $HOME/.defund --keep-addr-book

rm -rf $HOME/.defund/data 

SNAP_NAME=$(curl -s http://snapshots.l0vd.com/defund/ | egrep -o ">orbit-alpha-1.*\.tar.lz4" | tr -d ">")
curl http://snapshots.l0vd.com/defund/${SNAP_NAME} | lz4 -dc - | tar -xf - -C $HOME/.defund

mv $HOME/.defund/priv_validator_state.json.backup $HOME/.defund/data/priv_validator_state.json

sudo systemctl restart defundd
sudo journalctl -u defundd -f --no-hostname -o cat
```
