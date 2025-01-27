# Web3.DOTS
Another .Net implementation of the Ethereum RPC and Contract Interactions. Designed for Unity DOTS games but also works with 2020x.

Usage:

### Get RPC Data

```cs
        private static async Task GetRpcData()
        {
            var ethereumService = new EthereumService( ProviderUrl);
            var accountBalance = await ethereumService._provider.GetBalance("0x525b19d1cA89c3620b4A12B7D36970E410c8C5f5");
            var blockNumber = await ethereumService._provider.GetBlockNumber();
            var getBlock = await ethereumService._provider.GetBlock();
            var network = await ethereumService._provider.GetNetwork();
            Console.WriteLine($"Network name: {network.Name}");
            Console.WriteLine($"Network chain id: {network.ChainId}");
            Console.WriteLine("Account Balance: " + accountBalance);
            Console.WriteLine("Block Number: " + blockNumber);
            Console.WriteLine("Block Info: " + JsonConvert.SerializeObject(getBlock, Formatting.Indented));
        }
```

### Transfer Ether

```cs
        private static async Task TransferEther()
        {
            var ethereumService = new EthereumService(PrivateKey, ProviderUrl,new HexBigInteger(5));
            var txHash = await ethereumService.TransferEther("0x525b19d1cA89c3620b4A12B7D36970E410c8C5f5", 0.000001m);
            Console.WriteLine($"Hash: {txHash}");
        }
```

## Mint NFT

```cs
        public static async Task Mint()
        {
            // smart contract method to call
            string method = "safeMint";
            var ethereumService = new EthereumService(PrivateKey, ProviderUrl, new HexBigInteger(5));
            // connects to user's wallet to send a transaction
            try
            {
                var contract = new Contract(MintingContractAbi, MintingContractAddress,ethereumService.GetProvider());
                Console.WriteLine("Account: " + ethereumService.GetAddress(PrivateKey));
                var calldata = contract.Calldata(method, new object[]
                {
                    ethereumService.GetAddress(PrivateKey),
                });

                TransactionInput txInput = new TransactionInput
                {
                    To = MintingNftContractAddress,
                    From = ethereumService.GetAddress(PrivateKey),
                    Value = new HexBigInteger(0),
                    Data = calldata,
                    GasPrice = new HexBigInteger(100000),
                    Gas = new HexBigInteger(100000),
                };

                var txHash = await ethereumService.SignAndSendTransactionAsync(txInput);
                Console.WriteLine($"Transaction Hash: {txHash}");
            }
            catch (Exception e)
            {
                Console.Error.WriteLine(e);
            }
        }
```


## Mint NFT / Function Types

```cs
        [Function("safeMint")]
        public class SafeMintFunction : FunctionMessage
        {
            [Parameter("address", "to")] public string To { get; set; }
        }
        public static async Task Mint2()
        {
            var ethereumService = new EthereumService(PrivateKey, ProviderUrl, new HexBigInteger(5));
            var safeMint = new SafeMintFunction()
            {
                To = "0x525b19d1cA89c3620b4A12B7D36970E410c8C5f5",
            };
            safeMint.To = MintingNftContractAddress;
            safeMint.FromAddress = ethereumService.GetAddress(PrivateKey);
            safeMint.Gas = new HexBigInteger(100000);
            safeMint.GasPrice = new HexBigInteger(100000);
            var contractHandler = ethereumService._web3.Eth.GetContractHandler(MintingContractAddress);
            var txHash = await contractHandler.SendRequestAsync(safeMint);
            Console.WriteLine("Transaction Hash: " + txHash);
        }
```

## Sign Message

```cs
        public override Task<string> SignMessage(byte[] message)
        {
            var hash = new Sha3Keccack().CalculateHash(message);
            return Task.FromResult(_signingKey.Sign(new uint256(hash)).ToCompact().ToHex());
        }

        public override Task<string> SignMessage(string message)
        {
            var hash = new Sha3Keccack().CalculateHash(message);
            return Task.FromResult(_signingKey.Sign(new uint256(hash)).ToCompact().ToHex());
        }
```

