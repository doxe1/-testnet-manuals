# Validator transfer
![Validator transfer](https://github.com/doxe1/testnet-manuals/blob/main/Server-setup/Git-Transfer.png)
**Using Stride node as example**

Validators take more responsibility by raising a node, because they are responsible for the safety of delegates' funds in addition to their own steak. Validators cannot take direct possession of delegated funds, but their mistakes (when slashing) can cause the loss of some of the delegates' and their own funds. This can happen for two reasons:

### > Poor operation of the validator
When the validator node stops processing blocks and misses enough blocks. In this case, the validator is in jail, but can get out of jail. The amount of jailed funds is reduced by 0.01%.

### > Double signature
When the validator signs two identical blocks at the same height. In this case, the validator is imprisoned and can't get out of it. The share of the jailed funds is reduced by 5 percent. Double signing can happen if one `priv_validator_key.json` file is used on different servers, which is categorically not allowed.

## Let's get started
We need 2 servers, **1 server** is already running validator, which is in the active network. Initially we need to download from **first server** `priv_validator_key.json` file to our computer and have seed phrase to restore the wallet.

Next, we need to prepare **2 server** and run a node on it without validator and wallet (in our example Stride), to do this:

> Validator setup instructions: https://github.com/doxe1/testnet-manuals/tree/main/Stride

**Important! After creating the service file, we just wait. Node on 2 server should catch up completely with the blockchain**
```
strided status 2>&1 | jq ."SyncInfo"."latest_block_height"
```
After the node on server2 is fully synchronized, restore the wallet:
```
strided keys add <name_wallet> --recover --keyring-backend os
```
Now the most important step, which must be done very quickly. We will need to stop the nodes on both servers and replace the `priv_validator_key.json` file on second server. 

:heavy_exclamation_mark:**Again, stop the service on first server and on the second server**:heavy_exclamation_mark:

Stopping a node on 1 server
```
sudo systemctl stop strided
```
Stopping a node on 2 server
```
sudo systemctl stop strided
```
Delete `priv_validator_key.json` from on 2 server manually or with the command
```
cd $HOME/.stride/config/ && rm -vf priv_validator_key.json
```
Then transfer the previously copied `priv_validator_key.json` file from 1 server on 2 server and run the node on server 2.
```
sudo systemctl restart strided && sudo journalctl -u strided -f -o cat
```
If everything is done correctly, the validator will continue to process blocks, which you can see in explorer

![image](https://user-images.githubusercontent.com/84589100/182164202-ab5f15c7-ef7c-4307-b957-17663f2a0474.png)

You can also use the following command to check if a validator is in jail
```
strided query staking validator <stridevaloper1p6w...>
```
If you're in jail, you can get out like this
```
strided tx slashing unjail --from <name_wallet> --fees 500ustrd -y
```
After you have succeeded, you can delete server1 completely, or at least the priv_validator_key.json file from it, to avoid double signature!
