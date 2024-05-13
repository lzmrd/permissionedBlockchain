# permissionedBlockchain
This walkthrough will help you to create a permissioned EVM blockchain using Geth.
The instructions will allow you to build a blockchain from scratch with 3 nodes: 
- one miner/validator node
- one rpc node
- one archieve node  
  
The walktrough was thought to be realized on 3 different terminals and to start the blockcahin locally, but for production purposes you can deploy each node on different virtual machines or on different containers.

### Requirements:
- Ubuntu 22.04.3 LTS
- geth version 1.13.14-stable-2bd6bd01 [Geth Download](https://geth.ethereum.org/docs/getting-started/installing-geth#ubuntu-via-ppas)

## Walkthrough:

1. Open the first terminal
2. Create a new directory to host the project and run the followiing commands:  
 `mkdir blockchain`  
 `cd blockchain`  
 `geth --datadir miner-node account new`  
3. You'll need to enter a password that will unlock the account (Remember the password!)
Repeat this step for the other node as well  
  `geth --datadir rpc-node account new`  
4. In the terminal logs, you can see the account addresses of the newly created accounts, which you have to insert into the `genesis.json` file we'll create shortly
5. Still from the project root, create a new directory to host the archive node with command:  
`mkdir archive-node`  
6.   In all three directories just created (miner-node, rpc-node, archive-node), create the genesis.json file, which will have a structure like this:    

    
```
{
"config": {
"chainId": 666,
"homesteadBlock": 0,
"eip150Block": 0,
"eip155Block": 0,
"eip158Block": 0,
"byzantiumBlock": 0,
"constantinopleBlock": 0,
"petersburgBlock": 0,
"istanbulBlock": 0,
"muirGlacierBlock": 0,
"berlinBlock": 0,
"londonBlock": 0,
"arrowGlacierBlock": 0,
"grayGlacierBlock": 0,
"clique": {
"period": 5,
"epoch": 30000
}
},
"difficulty": "1",
"gasLimit": "800000000",
"extradata":
"0x0000000000000000000000000000000000000000000000000000000000000000XXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXX00000000000000000000000000000000000000000
0000000000000000000000000000000000000000000000000000000000000000000000000
0000000000000000",
"alloc": {
"XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX": { "balance":
"10000000000000000000000000000000000000000000000" },
"YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY": { "balance":
"1000000000000000000000000000000000000000" }
}
}
```

  
If this example doesn't work, you can find another one in the official Geth documentation, in the "Clique Example" section: [Private Networks | go-ethereum](https://geth.ethereum.org/docs/fundamentals/private-network)  

7. In the `extradata` field, replace the `XXXXXXXX` with the account address of your miner, but without the `0x` prefix.
8. In the `alloc` field, instead, insert (in place of `XXXXX` and `YYYYYY`) the account addresses of both accounts created earlier, always without the prefix.  
9.  Enter the miner's directory (cd miner-node) and run the command  
`geth init --datadir . genesis.json`  
10. Move to the other 2 directories and run the same command  
`geth init --datadir . genesis.json`  
11. Now moving to the `miner-node` directory we can start the first node with the command:  
`geth --datadir . --syncmode full --port 30313 --networkid 666 --unlock 0xC763Be5Dc0A7C18Ea925F52681863C262843f4De --allow-insecure-unlock --authrpc.port 8553 --mine --miner.etherbase 0xC763Be5Dc0A7C18Ea925F52681863C262843f4De`  
you'll be prompted to enter the password you chose for the account in the terminal.
Replace the account addresses in the above command with that of the miner created earlier.
12. Now open a new terminal and move to the `rpc-node` directory, where you can start the second node with the command:  
`geth --datadir . --syncmode full --port 30314 --http --http.port 8556 --http.api "eth,net ,web3,admin,personal,txpool,miner" --http.corsdomain "*" --networkid 666 --bootnodes enode://28beb915e25a15f6ccc059f46a7a878e92d1c3c6f34026343da938dddaa76d7a44a18d589fdddf534ea9042120e34a01a9557df5f5b87321ad63fa81505eaf08@127. 0.0.1:30313 --unlock 0x64CE8eeE42A93a1988D658c8A75cbB5965038919 --allow-insecure-unlock --gcmode archive --authrpc.port 8554`  
you'll be prompted to enter the password you chose for the account in the terminal.
Replace the enode with the one generated when starting the first node, which you can find in its logs, and the account address with the one generated earlier.
13. Now open a new terminal and move to the `archive-node` directory and start the third node with the command:  
`geth --datadir . --syncmode full --networkid 666 --vmdebug --gcmode archive --txlookuplimit 100000 --bootnodes enode://28beb915e25a15f6ccc059f46a7a878e92d1c3c6f34026343da938dddaa76d7 a44a18d589fdddf534ea9042120e34a01a9557df5f5b87321ad63fa81505eaf08@127. 0.0.1:30313 --authrpc.port 8558`  
Again, replace the enode with the one generated when starting the first node that you can find in its logs.
14. In a fourth terminal, connect to the newly created blockchain with the command
`geth attach http://localhost:8556` which will start a JS console
with which you can interact with the blockchain. For more info: [JavaScript Console | go-ethereum](https://geth.ethereum.org/docs/interacting-with-geth/javascript-console#main-content)

## Verify blockchain functioinality:

To ensure that the blockchain functions as expected, connect the JS console to the node with the command  
`geth attach http://localhost:8556`

If the command `eth.syncing` returns `false`, the blockchain is synchronized and you can execute a transaction with the command:   
  
  `eth.sendTransaction({from: '0x0000000000000', to: '0x11111111111', value: 55555 })`   
    
  replacing the `from` and `to` fields with the account addresses generated initially and present in the `genesis.json` file.

The console should return the transaction hash (e.g. "0x82e36d54814fc36c2e95e8861651023e78dad7ae09c19a87a95097311128f297"). To perform a final check that the transaction was successful, verify if the balance of the accounts is consistent with the transaction just made using the command `eth.getBalance('0x0000000000000')` within parentheses, insert the account address whose balance you want to check.
