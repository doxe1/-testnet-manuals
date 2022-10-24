![image](https://user-images.githubusercontent.com/84589100/197594943-f9bd7b2d-705c-4c38-9e80-7c402f958890.png)

# Bifrost | week 2

## Update

Stop bifrost-node (change the name if you have another name container)
```
docker stop bifrost
docker rm bifrost
```
Download bifrost-node:v1.0.1
```
docker pull thebifrost/bifrost-node:v1.0.1
```
Change bifrost-node version on run command
```
docker run -d --network host -v "/var/lib/bifrost-data:/data" --name "bifrost" thebifrost/bifrost-node:v1.0.1 \
  --base-path /data \
  --chain /specs/bifrost-testnet.json \
  --port 30333 \
  --validator \
  --state-cache-size 0 \
  --runtime-cache-size 64 \
  --name "YOUR_CONTROLLER_ADDRESS"
```

## Tasks

Add next settings to metamask (also you can attend a [cross swap](https://start.testnet.thebifrost.io/) and it will be auto added to your wallet)
```
- Network name
Bifrost test
- New RPC URL
https://public-01.testnet.thebifrost.io/rpc
- Chain ID
49088
- Currency symbol
BFC
- Block explorer URL
https://explorer.testnet.thebifrost.io/
```

1) Make 50 transactions to `0x9ab5574cff766b71033fc7e132d1c022ba911b55`

2) Make 50 transactions to `0x64be7d6a8355054a9e789f5f9f3e60dfdaad51df`

3) Transfer (verify if you has the same addresses in your task list, just to be sure):
```
Goerli : 0x7d208fE5CB10FbCc42be52AAA84bFcD5948E1178 2ETH
Mumbai : 0xF0AB8b1Ae7e35965f549a44870d4D41c64A8c061 2MATIC
BNB-Testnet : 0xa22F0ae4039a02A0965639E76e523d2A358d0Bf8 2BNB
```

4) Go to [cross swap](https://start.testnet.thebifrost.io/) and make 30 crosschain transfers with 0.2 BFC or more

![image](https://user-images.githubusercontent.com/84589100/197594702-c45f89d7-c3a6-4c4d-bc6b-223852c63b7d.png)

Then go to the profile and claim all your done tasks! Have a great day!
