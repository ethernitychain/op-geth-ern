## Go Ethereum

Golang execution layer implementation of the Ethereum protocol.

[![API Reference](
https://pkg.go.dev/badge/github.com/ethereum/go-ethereum
)](https://pkg.go.dev/github.com/ethereum/go-ethereum?tab=doc)
[![Go Report Card](https://goreportcard.com/badge/github.com/ethereum/go-ethereum)](https://goreportcard.com/report/github.com/ethereum/go-ethereum)
[![Travis](https://app.travis-ci.com/ethereum/go-ethereum.svg?branch=master)](https://app.travis-ci.com/github/ethereum/go-ethereum)
[![Discord](https://img.shields.io/badge/discord-join%20chat-blue.svg)](https://discord.gg/nthXNEv)

Automated builds are available for stable releases and the unstable master branch. Binary
archives are published at https://geth.ethereum.org/downloads/.

## Building the source

For prerequisites and detailed build instructions please read the [Installation Instructions](https://geth.ethereum.org/docs/getting-started/installing-geth).

Building `geth` requires both a Go (version 1.19 or later) and a C compiler. You can install
them using your favourite package manager. Once the dependencies are installed, run

```shell
make geth
```

or, to build the full suite of utilities:

```shell
make all
```

## Executables

The go-ethereum project comes with several wrappers/executables found in the `cmd`
directory.

|  Command   | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| :--------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`geth`** | Our main Ethereum CLI client. It is the entry point into the Ethereum network (main-, test- or private net), capable of running as a full node (default), archive node (retaining all historical state) or a light node (retrieving data live). It can be used by other processes as a gateway into the Ethereum network via JSON RPC endpoints exposed on top of HTTP, WebSocket and/or IPC transports. `geth --help` and the [CLI page](https://geth.ethereum.org/docs/fundamentals/command-line-options) for command line options. |
|   `clef`   | Stand-alone signing tool, which can be used as a backend signer for `geth`.                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|  `devp2p`  | Utilities to interact with nodes on the networking layer, without running a full blockchain.                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|  `abigen`  | Source code generator to convert Ethereum contract definitions into easy-to-use, compile-time type-safe Go packages. It operates on plain [Ethereum contract ABIs](https://docs.soliditylang.org/en/develop/abi-spec.html) with expanded functionality if the contract bytecode is also available. However, it also accepts Solidity source files, making development much more streamlined. Please see our [Native DApps](https://geth.ethereum.org/docs/developers/dapp-developer/native-bindings) page for details.                                  |
| `bootnode` | Stripped down version of our Ethereum client implementation that only takes part in the network node discovery protocol, but does not run any of the higher level application protocols. It can be used as a lightweight bootstrap node to aid in finding peers in private networks.                                                                                                                                                                                                                                               |
|   `evm`    | Developer utility version of the EVM (Ethereum Virtual Machine) that is capable of running bytecode snippets within a configurable environment and execution mode. Its purpose is to allow isolated, fine-grained debugging of EVM opcodes (e.g. `evm --code 60ff60ff --debug run`).                                                                                                                                                                                                                                               |
| `rlpdump`  | Developer utility tool to convert binary RLP ([Recursive Length Prefix](https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp)) dumps (data encoding used by the Ethereum protocol both network as well as consensus wise) to user-friendlier hierarchical representation (e.g. `rlpdump --hex CE0183FFFFFFC4C304050583616263`).                                                                                                                                                                                |

## Running `geth`

Going through all the possible command line flags is out of scope here (please consult our
[CLI Wiki page](https://geth.ethereum.org/docs/fundamentals/command-line-options)),
but we've enumerated a few common parameter combos to get you up to speed quickly
on how you can run your own `geth` instance.

### Hardware Requirements

Minimum:

* CPU with 2+ cores
* 4GB RAM
* 1TB free storage space to sync the Mainnet
* 8 MBit/sec download Internet service

Recommended:

* Fast CPU with 4+ cores
* 16GB+ RAM
* High-performance SSD with at least 1TB of free space
* 25+ MBit/sec download Internet service

### Full node on the main Ethereum network

By far the most common scenario is people wanting to simply interact with the Ethereum
network: create accounts; transfer funds; deploy and interact with contracts. For this
particular use case, the user doesn't care about years-old historical data, so we can
sync quickly to the current state of the network. To do so:

```shell
$ geth console
```

This command will:
 * Start `geth` in snap sync mode (default, can be changed with the `--syncmode` flag),
   causing it to download more data in exchange for avoiding processing the entire history
   of the Ethereum network, which is very CPU intensive.
 * Start the built-in interactive [JavaScript console](https://geth.ethereum.org/docs/interacting-with-geth/javascript-console),
   (via the trailing `console` subcommand) through which you can interact using [`web3` methods](https://github.com/ChainSafe/web3.js/blob/0.20.7/DOCUMENTATION.md) 
   (note: the `web3` version bundled within `geth` is very old, and not up to date with official docs),
   as well as `geth`'s own [management APIs](https://geth.ethereum.org/docs/interacting-with-geth/rpc).
   This tool is optional and if you leave it out you can always attach it to an already running
   `geth` instance with `geth attach`.

### A Full node on the Görli test network

Transitioning towards developers, if you'd like to play around with creating Ethereum
contracts, you almost certainly would like to do that without any real money involved until
you get the hang of the entire system. In other words, instead of attaching to the main
network, you want to join the **test** network with your node, which is fully equivalent to
the main network, but with play-Ether only.

```shell
$ geth --goerli console
```

The `console` subcommand has the same meaning as above and is equally
useful on the testnet too.

Specifying the `--goerli` flag, however, will reconfigure your `geth` instance a bit:

 * Instead of connecting to the main Ethereum network, the client will connect to the Görli
   test network, which uses different P2P bootnodes, different network IDs and genesis
   states.
 * Instead of using the default data directory (`~/.ethereum` on Linux for example), `geth`
   will nest itself one level deeper into a `goerli` subfolder (`~/.ethereum/goerli` on
   Linux). Note, on OSX and Linux this also means that attaching to a running testnet node
   requires the use of a custom endpoint since `geth attach` will try to attach to a
   production node endpoint by default, e.g.,
   `geth attach <datadir>/goerli/geth.ipc`. Windows users are not affected by
   this.

*Note: Although some internal protective measures prevent transactions from
crossing over between the main network and test network, you should always
use separate accounts for play and real money. Unless you manually move
accounts, `geth` will by default correctly separate the two networks and will not make any
accounts available between them.*

### Configuration

As an alternative to passing the numerous flags to the `geth` binary, you can also pass a
configuration file via:

```shell
$ geth --config /path/to/your_config.toml
```

To get an idea of how the file should look like you can use the `dumpconfig` subcommand to
export your existing configuration:

```shell
$ geth --your-favourite-flags dumpconfig
```

*Note: This works only with `geth` v1.6.0 and above.*

#### Docker quick start

One of the quickest ways to get Ethereum up and running on your machine is by using
Docker:

```shell
docker run -d --name ethereum-node -v /Users/alice/ethereum:/root \
           -p 8545:8545 -p 30303:30303 \
           ethereum/client-go
```

This will start `geth` in snap-sync mode with a DB memory allowance of 1GB, as the
above command does.  It will also create a persistent volume in your home directory for
saving your blockchain as well as map the default ports. There is also an `alpine` tag
available for a slim version of the image.

Do not forget `--http.addr 0.0.0.0`, if you want to access RPC from other containers
and/or hosts. By default, `geth` binds to the local interface and RPC endpoints are not
accessible from the outside.

### Programmatically interfacing `geth` nodes

As a developer, sooner rather than later you'll want to start interacting with `geth` and the
Ethereum network via your own programs and not manually through the console. To aid
this, `geth` has built-in support for a JSON-RPC based APIs ([standard APIs](https://ethereum.github.io/execution-apis/api-documentation/)
and [`geth` specific APIs](https://geth.ethereum.org/docs/interacting-with-geth/rpc)).
These can be exposed via HTTP, WebSockets and IPC (UNIX sockets on UNIX based
platforms, and named pipes on Windows).

The IPC interface is enabled by default and exposes all the APIs supported by `geth`,
whereas the HTTP and WS interfaces need to manually be enabled and only expose a
subset of APIs due to security reasons. These can be turned on/off and configured as
you'd expect.

HTTP based JSON-RPC API options:

  * `--http` Enable the HTTP-RPC server
  * `--http.addr` HTTP-RPC server listening interface (default: `localhost`)
  * `--http.port` HTTP-RPC server listening port (default: `8545`)
  * `--http.api` API's offered over the HTTP-RPC interface (default: `eth,net,web3`)
  * `--http.corsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
  * `--ws` Enable the WS-RPC server
  * `--ws.addr` WS-RPC server listening interface (default: `localhost`)
  * `--ws.port` WS-RPC server listening port (default: `8546`)
  * `--ws.api` API's offered over the WS-RPC interface (default: `eth,net,web3`)
  * `--ws.origins` Origins from which to accept WebSocket requests
  * `--ipcdisable` Disable the IPC-RPC server
  * `--ipcapi` API's offered over the IPC-RPC interface (default: `admin,debug,eth,miner,net,personal,txpool,web3`)
  * `--ipcpath` Filename for IPC socket/pipe within the datadir (explicit paths escape it)

You'll need to use your own programming environments' capabilities (libraries, tools, etc) to
connect via HTTP, WS or IPC to a `geth` node configured with the above flags and you'll
need to speak [JSON-RPC](https://www.jsonrpc.org/specification) on all transports. You
can reuse the same connection for multiple requests!

**Note: Please understand the security implications of opening up an HTTP/WS based
transport before doing so! Hackers on the internet are actively trying to subvert
Ethereum nodes with exposed APIs! Further, all browser tabs can access locally
running web servers, so malicious web pages could try to subvert locally available
APIs!**

### Operating a private network

Maintaining your own private network is more involved as a lot of configurations taken for
granted in the official networks need to be manually set up.

#### Defining the private genesis state

First, you'll need to create the genesis state of your networks, which all nodes need to be
aware of and agree upon. This consists of a small JSON file (e.g. call it `genesis.json`):

```json
{
  "config": {
    "chainId": <arbitrary positive integer>,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "berlinBlock": 0,
    "londonBlock": 0
  },
  "alloc": {},
  "coinbase": "0x0000000000000000000000000000000000000000",
  "difficulty": "0x20000",
  "extraData": "",
  "gasLimit": "0x2fefd8",
  "nonce": "0x0000000000000042",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp": "0x00"
}
```

The above fields should be fine for most purposes, although we'd recommend changing
the `nonce` to some random value so you prevent unknown remote nodes from being able
to connect to you. If you'd like to pre-fund some accounts for easier testing, create
the accounts and populate the `alloc` field with their addresses.

```json
"alloc": {
  "0x0000000000000000000000000000000000000001": {
    "balance": "111111111"
  },
  "0x0000000000000000000000000000000000000002": {
    "balance": "222222222"
  }
}
```

With the genesis state defined in the above JSON file, you'll need to initialize **every**
`geth` node with it prior to starting it up to ensure all blockchain parameters are correctly
set:

```shell
$ geth init path/to/genesis.json
```

#### Creating the rendezvous point

With all nodes that you want to run initialized to the desired genesis state, you'll need to
start a bootstrap node that others can use to find each other in your network and/or over
the internet. The clean way is to configure and run a dedicated bootnode:

```shell
$ bootnode --genkey=boot.key
$ bootnode --nodekey=boot.key
```

With the bootnode online, it will display an [`enode` URL](https://ethereum.org/en/developers/docs/networking-layer/network-addresses/#enode)
that other nodes can use to connect to it and exchange peer information. Make sure to
replace the displayed IP address information (most probably `[::]`) with your externally
accessible IP to get the actual `enode` URL.

*Note: You could also use a full-fledged `geth` node as a bootnode, but it's the less
recommended way.*

#### Starting up your member nodes

With the bootnode operational and externally reachable (you can try
`telnet <ip> <port>` to ensure it's indeed reachable), start every subsequent `geth`
node pointed to the bootnode for peer discovery via the `--bootnodes` flag. It will
probably also be desirable to keep the data directory of your private network separated, so
do also specify a custom `--datadir` flag.

```shell
$ geth --datadir=path/to/custom/data/folder --bootnodes=<bootnode-enode-url-from-above>
```

*Note: Since your network will be completely cut off from the main and test networks, you'll
also need to configure a miner to process transactions and create new blocks for you.*

#### Running a private miner


In a private network setting a single CPU miner instance is more than enough for
practical purposes as it can produce a stable stream of blocks at the correct intervals
without needing heavy resources (consider running on a single thread, no need for multiple
ones either). To start a `geth` instance for mining, run it with all your usual flags, extended
by:

```shell
$ geth <usual-flags> --mine --miner.threads=1 --miner.etherbase=0x0000000000000000000000000000000000000000
```

Which will start mining blocks and transactions on a single CPU thread, crediting all
proceedings to the account specified by `--miner.etherbase`. You can further tune the mining
by changing the default gas limit blocks converge to (`--miner.targetgaslimit`) and the price
transactions are accepted at (`--miner.gasprice`).


## URI Verification

The modified OP-Geth node processes blockchain transactions, decodes transaction input data, and sends specific URIs & TokenIDs to a defined endpoint. The modifications involve additional functionality to decode transaction input data based on contract ABI and handle specific methods related to URIs.


## Modifications

The following modifications were made to the OP-Geth node:

- Modified the `writeBlockAndSetHead` function to process transactions and decode input data in Blockchain.go
- Implemented `decodeTransactionInputData` to decode transaction input data using the contract ABI in Blockchain.go.
- Added `postDecodedInput` function to send the extracted Input data to a configured endpoint  in Blockchain.go.


## How It Works

### Reading ABI File

It reads the ABI (Application Binary Interface) file to understand the contract's methods and input parameters. The path to the ABI file is set using the `ABI_FILE_PATH` environment variable. If this environment variable is not set, it will not be able to decode the transaction data.

### Decoding Input Data

1. **Transaction Input Data Extraction:**
   It extracts the first 4 bytes of the transaction input data to identify the method signature.

2. **Method Identification:**
   Using the method signature, it identifies the corresponding method in the ABI.

3. **Parameter Unpacking:**
   The input parameters of the identified method are unpacked into a map for further processing.

### Code Modifications

1. **writeBlockAndSetHead Function:**
    ```go
    func (bc *BlockChain) writeBlockAndSetHead(block *types.Block, receipts []*types.Receipt, logs []*types.Log, state *state.StateDB, emitHeadEvent bool) (status WriteStatus, err error) {
        for _, tx := range block.Transactions() {
            toAddress := tx.To()
            if toAddress != nil {
                decodeTransactionInputData(toAddress, tx.Data())
            }
            fmt.Printf("Transaction input data: %x\n  tx addr %s\n", tx.Data(), tx.To())
        }
        
        Rest of the code
    ```

2. **decodeTransactionInputData Function:**
    ```go
       
   func decodeTransactionInputData(addr *common.Address, data []byte) {
   	defer func() {
   		if r := recover(); r != nil {
   			log.Warn("Recovered from panic in decodeTransactionInputData")
   		}
   	}()
   
   	if addr == nil {
   		log.Error("Invalid TX: addr is nil")
   		return
   	}
   	if len(data) < 4 {
   		log.Warn("Data length less than 4 bytes")
   		return
   	}
   
   	// Get the file path from the environment variable
   	abiPath := os.Getenv("ABI_FILE_PATH")
   	if abiPath == "" {
   		log.Error("ABI_FILE_PATH environment variable is not set")
   		return
   	}
   	reader, err := os.Open(abiPath)
   	if err != nil {
   		log.Error("Error decoding ABI", "error", err)
   		return
   	}
   	defer reader.Close()
   
   	contractABI, err := abi.JSON(reader)
   	if err != nil {
   		log.Error("Error decoding ABI:", "error", err)
   		return
   	}
   
   	methodSigData := data[:4]
   
   	method, err := contractABI.MethodById(methodSigData)
   	if err != nil {
   		log.Warn("Method not found in ABI")
   		return
   	}
   
   	inputsSigData := data[4:]
   	inputsMap := make(map[string]interface{})
   
   	err = method.Inputs.UnpackIntoMap(inputsMap, inputsSigData)
   	if err != nil {
   		log.Error("Error unpacking inputs", "error", err)
   		return
   	}
   
   	// Handle different methods based on their signatures and expected parameters
   	switch {
   	case bytes.Equal(methodSigData, []byte{0x02, 0xfe, 0x53, 0x05}):
   		// Decode setURI(string memory newuri) -- Signature: 0x02fe5305
   		if uri, ok := inputsMap["newuri"].(string); ok {
   			postDecodedInput(*addr, method.Name, uri, "")
   		}
   
   	case bytes.Equal(methodSigData, []byte{0xd2, 0x04, 0xc4, 0x5e}):
   		// Decode safeMint(address to, string memory uri) -- Signature: 0xd204c45e
   		if uri, ok := inputsMap["uri"].(string); ok {
   			postDecodedInput(*addr, method.Name, uri, "")
   		}
   
   	case bytes.Equal(methodSigData, []byte{0xcd, 0x27, 0x9c, 0x7c}):
   		// Decode safeMint(address to, uint256 tokenId, string memory uri) -- Signature: 0xcd279c7c
   		tokenIDStr := ""
   		if tokenID, ok := inputsMap["tokenId"]; ok {
   
   			tokenIDStr = fmt.Sprintf("%v", tokenID)
   		}
   		if uri, ok := inputsMap["uri"].(string); ok {
   			postDecodedInput(*addr, method.Name, uri, tokenIDStr)
   		}
   
   	case bytes.Equal(methodSigData, []byte{0x15, 0x6e, 0x29, 0xf6}):
   		// Decode mint(address account, uint256 id, uint256 amount) -- Signature: 0x156e29f6
   		if tokenID, ok := inputsMap["id"]; ok {
   			fmt.Printf("Type of tokenID: %T\n", tokenID)
   			tokenIDStr := fmt.Sprintf("%v", tokenID)
   			postDecodedInput(*addr, method.Name, "", tokenIDStr)
   		}
   
   	case bytes.Equal(methodSigData, []byte{0x73, 0x11, 0x33, 0xe9}):
   		// Decode mint(address account, uint256 id, uint256 amount, bytes memory data) -- Signature: 0x731133e9
   		if ids, ok := inputsMap["ids"]; ok {
   			tokenIDStr := fmt.Sprintf("%v", ids)
   			postDecodedInput(*addr, method.Name, "", tokenIDStr)
   		}
   
   	case bytes.Equal(methodSigData, []byte{0x1f, 0x7f, 0xdf, 0xfa}):
   		// Decode mintBatch(address to, uint256[] memory ids, uint256[] memory amounts, bytes memory data) -- Signature: 0x1f7fdffa
   		tokenIDStr := "[]"
   		if ids, ok := inputsMap["ids"]; ok {
   			tokenIDStr = fmt.Sprintf("%v", ids)
   			postDecodedInput(*addr, method.Name, "", tokenIDStr)
   		}
   
   	case bytes.Equal(methodSigData, []byte{0xd8, 0x1d, 0x0a, 0x15}):
   		// Decode mintBatch(address to, uint256[] memory ids, uint256[] memory amounts) -- Signature: 0xd81d0a15
   		tokenIDStr := "[]"
   		if ids, ok := inputsMap["ids"]; ok {
   			tokenIDStr = fmt.Sprintf("%v", ids)
   			postDecodedInput(*addr, method.Name, "", tokenIDStr)
   		}
   	}
   }

    ```

3. **postDecodedInput Function:**
    ```go
   func postDecodedInput(addr common.Address, method string, uri string, tokenID string) {
   	origin := os.Getenv("ORIGIN")
   	if origin == "" {
   		origin = "op-geth"
   	}
   
   	payload := fmt.Sprintf(`{"contract":"%s","uri":"%s","method":"%s","origin":"%s","token_id":%s}`,
   		addr.Hex(),
   		uri,
   		method,
   		origin,
   		tokenID,
   	)
   
   	uriEndpoint := os.Getenv("URI_ENDPOINT")
   	if uriEndpoint == "" {
   		log.Error("URI_ENDPOINT environment variable is not set")
   		return
   	}
   
   	resp, err := http.Post(uriEndpoint, "application/json", bytes.NewBuffer([]byte(payload)))
   	if err != nil {
   		log.Error("Error sending URI", "error", err)
   		return
   	}
   	defer resp.Body.Close()
   }
    ```

## Workings

1. **Reading ABI File:**
   The ABI file path is set using the `ABI_FILE_PATH` environment variable. The application reads the ABI file to understand the contract's methods and inputs.

2. **Decoding Input Data:**
   It extracts the first 4 bytes of the transaction input data to identify the method signature. It then decodes the input parameters of the method using the ABI.

3. **Posting Decoded Input:**
   For specific methods, the extracted Payload is sent to the endpoint specified by the `URI_ENDPOINT` environment variable.

### Posting Decoded Input

For specific methods related to URIs, it extracts the URI and sends it to a defined endpoint. The endpoint is configured using the `URI_ENDPOINT` environment variable. If this environment variable is not set, it will not send the URI.

### Example Workflow

First make sure that `ABI_FILE_PATH` & `URI_ENDPOINT` is set. To decode and process a `safeMint(address to, uint256 tokenId, string memory uri)` transaction:

1. **Method Signature Identification:**
   The method signature `0xcd279c7c` is extracted from the transaction input data.

2. **Input Parameter Extraction:**
   The `uri` & `tokenId` parameter is extracted from the transaction input data.

4. **Payload Posting:**
   The extracted Payload is sent to the endpoint specified by the `URI_ENDPOINT` environment variable.


## Usage

1. Set the required environment variables 

## Environment Variables

It relies on the following environment variables:

- `ABI_FILE_PATH`: Path to the ABI file used for decoding transaction input data.
- `URI_ENDPOINT`: Endpoint to which the extracted URIs will be sent.
- `ORIGIN`: If it wasn't set, the default will be used, `op-geth`

## Setting Environment Variables

Set these environment variables in your terminal or in a `.env` file:

```bash
export ABI_FILE_PATH="/path/to/your/contract.abi"
export URI_ENDPOINT="https://your-endpoint.com/uri"
export ORIGIN=origin
```



## Contribution

Thank you for considering helping out with the source code! We welcome contributions
from anyone on the internet, and are grateful for even the smallest of fixes!

If you'd like to contribute to go-ethereum, please fork, fix, commit and send a pull request
for the maintainers to review and merge into the main code base. If you wish to submit
more complex changes though, please check up with the core devs first on [our Discord Server](https://discord.gg/invite/nthXNEv)
to ensure those changes are in line with the general philosophy of the project and/or get
some early feedback which can make both your efforts much lighter as well as our review
and merge procedures quick and simple.

Please make sure your contributions adhere to our coding guidelines:

 * Code must adhere to the official Go [formatting](https://golang.org/doc/effective_go.html#formatting)
   guidelines (i.e. uses [gofmt](https://golang.org/cmd/gofmt/)).
 * Code must be documented adhering to the official Go [commentary](https://golang.org/doc/effective_go.html#commentary)
   guidelines.
 * Pull requests need to be based on and opened against the `master` branch.
 * Commit messages should be prefixed with the package(s) they modify.
   * E.g. "eth, rpc: make trace configs optional"

Please see the [Developers' Guide](https://geth.ethereum.org/docs/developers/geth-developer/dev-guide)
for more details on configuring your environment, managing project dependencies, and
testing procedures.

### Contributing to geth.ethereum.org

For contributions to the [go-ethereum website](https://geth.ethereum.org), please checkout and raise pull requests against the `website` branch.
For more detailed instructions please see the `website` branch [README](https://github.com/ethereum/go-ethereum/tree/website#readme) or the 
[contributing](https://geth.ethereum.org/docs/developers/geth-developer/contributing) page of the website.

## License

The go-ethereum library (i.e. all code outside of the `cmd` directory) is licensed under the
[GNU Lesser General Public License v3.0](https://www.gnu.org/licenses/lgpl-3.0.en.html),
also included in our repository in the `COPYING.LESSER` file.

The go-ethereum binaries (i.e. all code inside of the `cmd` directory) are licensed under the
[GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html), also
included in our repository in the `COPYING` file.
