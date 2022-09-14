# Haqq
![Haqq Network Guide](https://github.com/doxe1/testnet-manuals/blob/main/actual-nodes/haqq/photo_2022-09-11_20-03-15.jpg)
Official documentation:

> Validator setup instructions:

https://github.com/haqq-network/testnets/tree/main/TestEdge

> Explorer:

https://pingpub.explorer.haqq.network/haqq

https://explorer.nodestake.top/haqq-testnet

> Network Chain ID

haqq_54211-2

> Denom: 

aISLM

## Hardware Requirements

Like any Cosmos-SDK chain, the hardware requirements are pretty easy.

`Recommended Hardware Requirements`

- 4x CPUs; the faster clock speed the better
- 8GB RAM
- 200GB of storage (SSD or NVME)
- Permanent Internet connection (traffic will be minimal during mainnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

## Preparing the server

Prerequisite: [server-setup](https://github.com/doxe1/testnet-manuals/tree/main/server-setup)

## Node start-up

:heavy_exclamation_mark:IMPORTANT - in the commands below, change everything in <> to your value and remove <>:heavy_exclamation_mark:

```
cd $HOME && git clone https://github.com/haqq-network/haqq && \
cd haqq && \
git checkout v1.0.3 && \
make install
```
Check version
```
haqqd version --long | head
```
Initialize a node to create the necessary configuration files
```
haqqd init <name_moniker> --chain-id haqq_54211-2
```
Download `genesis file`
```
cd $HOME/.haqqd/config/ && wget https://raw.githubusercontent.com/haqq-network/validators-contest/master/genesis.json
```
Check version of a `genesis file`
```
sha256sum ~/.haqqd/config/genesis.json
```
Check the state of the validator blocks at the initial stage
```
cd && cat .haqqd/data/priv_validator_state.json
```
`{
  "height": "0",
  "round": 0,
  "step": 0
}`

If the values above are not equal to zero, then execute the command, but if they are equal to zero, then skip this step
```
haqqd tendermint unsafe-reset-all --home $HOME/.haqqd
```
Download the `addrbook file`
```
wget -O $HOME/.haqqd/config/addrbook.json "https://raw.githubusercontent.com/doxe1/testnet-manuals/main/actual-nodes/haqq/addrbook.json"
```
## Setting up the node configuration
Edit the config so that we no longer use the `chain-id` flag for each CLI command in client.toml
```
haqqd config chain-id haqq_54211-2
```
If necessary, configure the keyring-backend in `client.toml`
```
haqqd config keyring-backend test
```
Setting the minimum price for gas in `app.toml`
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025aISLM\"/;" ~/.haqqd/config/app.toml
```
Add seeds/bpeers/peers to `config.toml`
```
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.haqqd/config/config.toml
```
```
peers="b3ce1618585a9012c42e9a78bf4a5c1b4bad1123@65.21.170.3:33656,952b9d918037bc8f6d52756c111d0a30a456b3fe@213.239.217.52:29656,85301989752fe0ca934854aecc6379c1ccddf937@65.109.49.111:26556,d648d598c34e0e58ec759aa399fe4534021e8401@109.205.180.81:29956,f2c77f2169b753f93078de2b6b86bfa1ec4a6282@141.95.124.150:20116,eaa6d38517bbc32bdc487e894b6be9477fb9298f@78.107.234.44:45656,37513faac5f48bd043a1be122096c1ea1c973854@65.108.52.192:36656,d2764c55607aa9e8d4cee6e763d3d14e73b83168@66.94.119.47:26656,fc4311f0109d5aed5fcb8656fb6eab29c15d1cf6@65.109.53.53:26656,297bf784ea674e05d36af48e3a951de966f9aa40@65.109.34.133:36656,bc8c24e9d231faf55d4c6c8992a8b187cdd5c214@65.109.17.86:32656,8645831391634afcc0e2e6283b36be6c5f470415@144.126.140.252:26656,99a8389c84625503c2b8d734dfd78035d28e4f15@65.109.30.117:26656,afe8c5af90e2eef4a98bc998366e2e780a927599@65.108.126.46:34656,749ed555c435509908736bdf28910c79b3676207@65.21.227.78:29656,302466457301efc7b12f61561b85ab366ece5659@142.132.248.253:26656,35bb813a902ae1122c146d83f1eaa83a58692a4a@65.108.76.44:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.haqqd/config/config.toml
```
```
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.haqqd/config/config.toml
```
**(Optional)** Configure pruning with one command `app.toml`
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.haqqd/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.haqqd/config/app.toml
```
**(Optional)** Turn off indexing in `config.toml`
```
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.haqqd/config/config.toml
```
**(Optional)** On/off snapshots in `app.toml`
```
snapshot_interval=1000 && \
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.haqqd/config/app.toml
```
By default snapshots are disabled `snapshot-interval=0`
## Create a service file
```
sudo tee /etc/systemd/system/haqqd.service > /dev/null <<EOF
[Unit]
Description=haqqd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which haqqd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload && \
sudo systemctl enable haqqd && \
sudo systemctl restart haqqd && sudo journalctl -u haqqd -f -o cat
```
#### :heavy_exclamation_mark:If after start the node cannot connect to peers for a long time, then look for new peers or ask for addrbook.json in discord and so on:heavy_exclamation_mark:

Stop the node, delete the address book and reset node
```
sudo systemctl stop haqqd
rm $HOME/.haqqd/config/addrbook.json
haqqd tendermint unsafe-reset-all --home $HOME/.haqqd
```
Restart the node
```
sudo systemctl restart haqqd && journalctl -u haqqd -f -o cat
```
Create or restore the wallet and save the memnomics and other information as needed

#### :heavy_exclamation_mark:Don't forget to save yourself a memo, because you are responsible for your wallet:heavy_exclamation_mark:

Set up a wallet
```
haqqd keys add <name_wallet> --keyring-backend test
```
Recover a wallet
```
haqqd keys add <name_wallet> --recover --keyring-backend test
```
Connect ledger wallet
```
haqqd keys add <name_wallet> --ledger 
```
Create a validator

#### :heavy_exclamation_mark:Don't forget to save `priv_validator_key.json`:heavy_exclamation_mark:
```
haqqd tx staking create-validator \
--chain-id haqq_54211-2 \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000aISLM \
--pubkey $(haqqd tendermint show-validator) \
--moniker "<name_moniker>" \
--from <name_wallet> \
--fees 555aISLM
```
## Useful Commands

Check block heights
```
haqqd status 2>&1 | jq ."SyncInfo"."latest_block_height"
```
Check logs (all logs and the last 100 lines)
```
sudo journalctl -u haqqd -f -o cat
sudo journalctl --lines=100 --follow --unit haqqd
```
Check status
```
curl localhost:26657/status
```
Check a balance of the wallet
```
haqqd q bank balances <address>
```
Collect commissions + rewards
```
haqqd tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees 500aISLM --commission -y
```
Delegate some more coins to yourself (for example, 1 coin is sent)
```
haqqd tx staking delegate <valoper_address> 1000000aISLM --from <name_wallet> --fees 500aISLM -y
```
Send coins to another address
```
haqqd tx bank send <name_wallet> <address> 1000000aISLM --fees 500aISLM -y
```
Get out of jail
```
haqqd tx slashing unjail --from <name_wallet> --fees 500aISLM -y
```
Display a list of wallets
```
haqqd keys list
```
Vote for the proposal 
```
haqqd tx gov vote 1 yes --from <name_wallet> --fees 555aISLM
```
Delete a node
```
sudo systemctl stop haqqd && \
sudo systemctl disable haqqd && \
rm /etc/systemd/system/haqqd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .haqqd haqq && \
rm -rf $(which haqqd)
```
