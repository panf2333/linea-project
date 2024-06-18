# Process

1. forge init linea-project
2. cd linea-project
3. mkdir .env and configure
>path linea-project/.env   
PRIVATE_KEY=YOUR_PRIVATE_KEY    <font color="red"> your wallet address private key</font>  
INFURA_API_KEY=YOUR_INFURA_API_KEY  [infura api key](https://support.infura.io/blockchain-networks/linea/connecting-to-linea)   
LINEASCAN_API_KEY=YOUR_LINEASCAN_API_KEY [lineascanAPIKey](https://lineascan.build/myapikey)

4. <font color="red">source .env</font>
5. touch foundry.toml and configure
```
[rpc_endpoints]   
linea-testnet = "https://linea-sepolia.infura.io/v3/${INFURA_API_KEY}"   
linea-mainnet = "https://linea-mainnet.infura.io/v3/${INFURA_API_KEY}"  

[etherscan]
linea-testnet = { key = "${LINEASCAN_API_KEY}", url = "https://api-testnet.lineascan.build/api" }
linea-mainnet = { key = "${LINEASCAN_API_KEY}", url = "https://api.lineascan.build/api" }
```  
6. Deploy NFT
    1. forge create --rpc-url linea-testnet src/SimpleCryptoKitties.sol:SimpleCryptoKitties --private-key $PRIVATE_KEY
    >Error (6275): Source "@openzeppelin/contracts@4.9.3/token/ERC721/ERC721.sol" not found: File not found. Searched the following locations: "/home/panf233/lineaProject/linea-project". 
    2. forge install OpenZeppelin/openzeppelin-contracts
    3. forge create --rpc-url linea-testnet src/SimpleCryptoKitties.sol:SimpleCryptoKitties --private-key $PRIVATE_KEY
    >Error:   
server returned an error response: error code -32009: Gas price below configured minimum gas price


    4. query gas 
    ```python
    import requests
    import json
    def get_gas_price():
        infura_url = "https://linea-sepolia.infura.io/v3/98b178401d334654979b262accef0282"
        headers = {
            "Content-Type": "application/json",
        }
        payload = {
            "jsonrpc": "2.0",
            "method": "eth_gasPrice",
            "params": [],
            "id": 1
        }

        response = requests.post(infura_url, headers=headers, data=json.dumps(payload))
        gas_price_hex = response.json()["result"]
        gas_price_wei = int(gas_price_hex, 16)

        return gas_price_wei

        gas_price_wei = get_gas_price()
        gas_price_gwei = gas_price_wei / 10**9
        print(f"Current Gas Price: {gas_price_gwei} Gwei {gas_price_wei} wei")

    ```
    5. add --gas-price {gas amount}
    >forge create --rpc-url linea-testnet src/SimpleCryptoKitties.sol:SimpleCryptoKitties --private-key $PRIVATE_KEY --gas-price 55050010
    return   
    Deployer: 0x50F1e3510F6e4a8E83F195B7071B50Fa5dF0bB87        
    Deployed to: 0x0078bFae1C5bf0Cd62D2EDEc7C4E2c0f2b8f5CCf         
    Transaction hash: 0x1031b76fefe58e791953893ecdfb5ec0ce2278adfc03d93ce4072a4cc8b6d3e7

    https://sepolia.lineascan.build/address/0x0078bFae1C5bf0Cd62D2EDEc7C4E2c0f2b8f5CCf


7. Deploy DutchAuction.sol
>  forge create --rpc-url linea-testnet src/DutchAuction.sol:DutchAuction --private-key $PRIVATE_KEY --constructor-args 1718701460335 1 0x0078bFae1C5bf0Cd62D2EDEc7C4E2c0f2b8f5CCf 1 --gas-price 55050010    
>Deployer: 0x50F1e3510F6e4a8E83F195B7071B50Fa5dF0bB87   
Deployed to: 0xd2810924F67e24675d7246Db4103fC4cdBB0Fa33   
Transaction hash: 0xa58de95e7810c40d6a2c74ff686b36da35ab9abbaf00fe69a7612a598ef3b361 


8. veirfy the contract
> [verify-smart-contract](https://docs.linea.build/developers/quickstart/verify-smart-contract/foundry)
>forge verify-contract --etherscan-api-key $LINEASCAN_API_KEY \
 --verifier-url https://api-sepolia.lineascan.build/api 0xd2810924F67e24675d7246Db4103fC4cdBB0Fa33 src/DutchAuction.sol:DutchAuction \
 --constructor-args $(cast abi-encode "constructor(uint256,uint256,address,uint256)" 1718701460335 1 0x0078bFae1C5bf0Cd62D2EDEc7C4E2c0f2b8f5CCf 1) \
 --watch

>recommand verify when create：  forge create --rpc-url https://linea-sepolia.infura.io/v3/INFURA_API_KEY src/Counter.sol:Counter --private-key YOUR_PRIVATE_KEY --verify --verifier-url https://api-sepolia.lineascan.build/api --etherscan-api-key LINEASCAN_API_KEY


> forge verify-contract --etherscan-api-key $LINEASCAN_API_KEY \
 --verifier-url https://api-sepolia.lineascan.build/api 0xd2810924F67e24675d7246Db4103fC4cdBB0Fa33 src/DutchAuction.sol:DutchAuction \
 --constructor-args-path constructor-args.txt \
 --watch