# Rebus
![Rebus Network Guide](https://github.com/doxe1/testnet-manuals/blob/main/archive/rebus/rebus-logo-white.png)

Official documentation:

> Validator setup instructions:

https://docs.rebuschain.com/validators/joining-the-testnets/

> Explorer:

https://exp.nodeist.net/Rebus/staking

https://rebus.explorers.guru/validators

> Network Chain ID

reb_3333-1

> Denom: 

arebus

## Hardware Requirements

Like any Cosmos-SDK chain, the hardware requirements are pretty easy.

`Recommended Hardware Requirements`

- 4x CPUs; the faster clock speed the better
- 8GB RAM
- 200GB of storage (SSD or NVME)
- Permanent Internet connection (traffic will be minimal during mainnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

## Server setup

Prerequisite: [server-setup](https://github.com/doxe1/testnet-manuals/tree/main/server-setup)

## Node start-up

:heavy_exclamation_mark:IMPORTANT - in the commands below, change everything in <> to your value and remove <>:heavy_exclamation_mark:

```
cd $HOME
git clone https://github.com/rebuschain/rebus.core.git 
cd rebus.core && git checkout v0.0.3
make install
```
Check version
```
rebusd version --long | head
```
Initialize a node to create the necessary configuration files
```
rebusd init <name_moniker> --chain-id reb_3333-1
```
Download `genesis file`
```
wget -O $HOME/.rebusd/config/genesis.json "https://raw.githubusercontent.com/rebuschain/rebus.testnet/master/rebus_3333-1/genesis.json"
```
Check version of a `genesis file`
```
sha256sum ~/.rebusd/config/genesis.json
```
Check the state of the validator blocks at the initial stage
```
cd && cat .rebusd/data/priv_validator_state.json
```
`{
  "height": "0",
  "round": 0,
  "step": 0
}`

If the values above are not equal to zero, then execute the command, but if they are equal to zero, then skip this step
```
rebusd tendermint unsafe-reset-all --home $HOME/.rebusd
```
Download the `addrbook file`
```
wget -O $HOME/.teritorid/config/addrbook.json "https://raw.githubusercontent.com/doxe1/testnet-manuals/main/actual-nodes/rebus/addrbook.json"
```
## Setting up the node configuration
Edit the config so that we no longer use the `chain-id` flag for each CLI command in client.toml
```
rebusd config chain-id reb_3333-1
```
If necessary, configure the keyring-backend in `client.toml`
```
rebusd config keyring-backend os
```
Setting the minimum price for gas in `app.toml`
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025arebus\"/;" ~/.rebusd/config/app.toml
```
Add seeds/bpeers/peers to `config.toml`
```
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.rebusd/config/config.toml
```
```
peers="40b7f1fe0cf85c4ab3f5e2aa9731dbf07aa421b3@65.109.17.86:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.rebusd/config/config.toml
```
```
seeds="a6d710cd9baac9e95a55525d548850c91f140cd9@3.211.101.169:26656,c296ee829f137cfe020ff293b6fc7d7c3f5eeead@54.157.52.47:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.rebusd/config/config.toml
```
**(Optional)** Configure pruning with one command `app.toml`
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.rebusd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.rebusd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.rebusd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.rebusd/config/app.toml
```
**(Optional)** Turn off indexing in `config.toml`
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.rebusd/config/config.toml
```
**(Optional)** On/off snapshots in `app.toml`
```
snapshot_interval=1000 && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.rebusd/config/app.toml
```
By default snapshots are disabled `snapshot-interval=0`
## Create a service file
```
sudo tee /etc/systemd/system/rebusd.service > /dev/null <<EOF
[Unit]
Description=rebusd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which rebusd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload && \
sudo systemctl enable rebusd && \
sudo systemctl restart rebusd && sudo journalctl -u rebusd -f -o cat
```
:heavy_exclamation_mark:If after start the node cannot connect to peers for a long time, then look for new peers or ask for addrbook.json in discord and so on:heavy_exclamation_mark:

Stop the node, delete the address book and reset node
```
sudo systemctl stop rebusd
rm $HOME/.rebusd/config/addrbook.json
rebusd tendermint unsafe-reset-all --home $HOME/.rebusd
```
Restart the node
```
sudo systemctl restart rebusd && journalctl -u rebusd -f -o cat
```
Create or restore the wallet and save the memnomics and other information as needed

:heavy_exclamation_mark:Don't forget to save yourself a memo, because you are responsible for your wallet:heavy_exclamation_mark:

Set up a wallet
```
rebusd keys add <name_wallet> --keyring-backend os
```
Recover a wallet
```
rebusd keys add <name_wallet> --recover --keyring-backend os
```
Connect ledger wallet
```
rebusd keys add <name_wallet> --ledger 
```
Create a validator

:heavy_exclamation_mark:Don't forget to save `priv_validator_key.json` 
```
rebusd tx staking create-validator \
--chain-id reb_3333-1 \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000arebus \
--pubkey $(rebusd tendermint show-validator) \
--moniker "<name_moniker>" \
--from <name_wallet> \
--fees 555arebus
```
## Useful Commands

Check block heights
```
rebusd status 2>&1 | jq ."SyncInfo"."latest_block_height"
```
Check logs (all logs and the last 100 lines)
```
sudo journalctl -u rebusd -f -o cat
sudo journalctl --lines=100 --follow --unit rebusd
```
Check status
```
curl localhost:26657/status
```
Check a balance of the wallet
```
rebusd q bank balances <address>
```
Collect commissions + rewards
```
rebusd tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees 500arebus --commission -y
```
Delegate some more coins to yourself (for example, 1 coin is sent)
```
rebusd tx staking delegate <valoper_address> 1000000arebus --from <name_wallet> --fees 500arebus -y
```
Send coins to another address
```
rebusd tx bank send <name_wallet> <address> 1000000arebus --fees 500arebus -y
```
Get out of jail
```
rebusd tx slashing unjail --from <name_wallet> --fees 500arebus -y
```
Display a list of wallets
```
rebusd keys list
```
Vote for the proposal 
```
rebusd tx gov vote 1 yes --from <name_wallet> --fees 555arebus
```
Delete a node
```
sudo systemctl stop rebusd && \
sudo systemctl disable rebusd && \
rm /etc/systemd/system/rebusd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .rebusd rebus.core && \
rm -rf $(which rebusd)
```
