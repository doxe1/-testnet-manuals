# Teritori
![Teritori Network Guide](https://github.com/doxe1/testnet-manuals/blob/main/actual-nodes/teritory/1500x500.jpg)
Official documentation:

> Validator setup instructions:

https://github.com/TERITORI/teritori-chain/tree/main/testnet/teritori-testnet-v2

> Explorer:

https://teritori.explorers.guru/

> Network Chain ID

teritori-testnet-v2

> Denom: 

utori

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
git clone https://github.com/TERITORI/teritori-chain
cd teritori-chain
git checkout teritori-testnet-v2
make install
```
Check version
```
teritorid version --long | head
```
Initialize a node to create the necessary configuration files
```
teritorid init <name_moniker> --chain-id teritori-testnet-v2
```
Download `genesis file`
```
wget -O $HOME/.teritorid/config/genesis.json "https://raw.githubusercontent.com/TERITORI/teritori-chain/teritori-testnet-v2/genesis/genesis.json"
```
Check version of a `genesis file`
```
sha256sum ~/.teritorid/config/genesis.json
```
Check the state of the validator blocks at the initial stage
```
cd && cat .teritorid/data/priv_validator_state.json
```
`{
  "height": "0",
  "round": 0,
  "step": 0
}`

If the values above are not equal to zero, then execute the command, but if they are equal to zero, then skip this step
```
teritorid tendermint unsafe-reset-all --home $HOME/.teritorid
```
Download the `addrbook file`
```
wget -O $HOME/.teritorid/config/addrbook.json "https://raw.githubusercontent.com/doxe1/testnet-manuals/main/actual-nodes/teritory/addrbook.json"
```
## Setting up the node configuration
Edit the config so that we no longer use the `chain-id` flag for each CLI command in client.toml
```
teritorid config chain-id teritori-testnet-v2
```
If necessary, configure the keyring-backend in `client.toml`
```
teritorid config keyring-backend os
```
Setting the minimum price for gas in `app.toml`
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025utori\"/;" ~/.teritorid/config/app.toml
```
Add seeds/bpeers/peers to `config.toml`
```
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.teritorid/config/config.toml
```
```
peers="20d21518f3e54e0b8825bc244017c525cb862d17@rpc2.bonded.zone:20956,0d19829b0dd1fc324cfde1f7bc15860c896b7ac1@teritori-testnet.nodejumper.io:27656,1796a1a96e4cb11d59f1988baf435ca21f3bf17a@65.109.17.86:27656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.teritorid/config/config.toml
```
```
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.teritorid/config/config.toml
```
Increase the number of incoming and outgoing peers for the connection, except for persistent peers in `config.toml`
```
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.teritorid/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.teritorid/config/config.toml
```
**(Optional)** Configure pruning with one command `app.toml`
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.teritorid/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.teritorid/config/app.toml
```
**(Optional)** Turn off indexing in `config.toml`
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.teritorid/config/config.toml
```
**(Optional)** On/off snapshots in `app.toml`
```
snapshot_interval=1000 && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.teritorid/config/app.toml
```
By default snapshots are disabled `snapshot-interval=0`
## Create a service file
```
sudo tee /etc/systemd/system/teritorid.service > /dev/null <<EOF
[Unit]
Description=teritorid
After=network-online.target

[Service]
User=$USER
ExecStart=$(which teritorid) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload && \
sudo systemctl enable teritorid && \
sudo systemctl restart teritorid && sudo journalctl -u teritorid -f -o cat
```
:heavy_exclamation_mark:If after start the node cannot connect to peers for a long time, then look for new peers or ask for addrbook.json in discord and so on:heavy_exclamation_mark:

Stop the node, delete the address book and reset node
```
sudo systemctl stop teritorid
rm $HOME/.teritorid/config/addrbook.json
teritorid tendermint unsafe-reset-all --home $HOME/.teritorid
```
Restart the node
```
sudo systemctl restart teritorid && journalctl -u teritorid -f -o cat
```
Create or restore the wallet and save the memnomics and other information as needed

:heavy_exclamation_mark:Don't forget to save yourself a memo, because you are responsible for your wallet:heavy_exclamation_mark:

Set up a wallet
```
teritorid keys add <name_wallet> --keyring-backend os
```
Recover a wallet
```
teritorid keys add <name_wallet> --recover --keyring-backend os
```
Connect ledger wallet
```
teritorid keys add <name_wallet> --ledger 
```
Create a validator

:heavy_exclamation_mark:Don't forget to save `priv_validator_key.json` 
```
teritorid tx staking create-validator \
--chain-id teritori-testnet-v2 \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000utori \
--pubkey $(teritorid tendermint show-validator) \
--moniker "<name_moniker>" \
--from <name_wallet> \
--fees 555utori
```
## Useful Commands

Check block heights
```
teritorid status 2>&1 | jq ."SyncInfo"."latest_block_height"
```
Check logs (all logs and the last 100 lines)
```
sudo journalctl -u teritorid -f -o cat
sudo journalctl --lines=100 --follow --unit teritorid
```
Check status
```
curl localhost:26657/status
```
Check a balance of the wallet
```
teritorid q bank balances <address>
```
Collect commissions + rewards
```
teritorid tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees 500utori --commission -y
```
Delegate some more coins to yourself (for example, 1 coin is sent)
```
teritorid tx staking delegate <valoper_address> 1000000utori --from <name_wallet> --fees 500utori -y
```
Send coins to another address
```
teritorid tx bank send <name_wallet> <address> 1000000utori --fees 500utori -y
```
Get out of jail
```
teritorid tx slashing unjail --from <name_wallet> --fees 500utori -y
```
Display a list of wallets
```
teritorid keys list
```
Vote for the proposal 
```
teritorid tx gov vote 1 yes --from <name_wallet> --fees 555utori
```
Delete a node
```
sudo systemctl stop teritorid && \
sudo systemctl disable teritorid && \
rm /etc/systemd/system/teritorid.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .teritorid teritori-chain && \
rm -rf $(which teritorid)
```
