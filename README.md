
# theloop-fabric
This is a Node.js library for easier interaction Hyperledger Fabric chaincode.
 https://github.com/hyperledger/fabric

All documentation is on this page.

Table Of Contents:

***

## Installation

```
npm install theloop-fabric
```

***

## Usage Steps!

1. Require this module
1. Pass network + chaincode parameters to loop.load(options, my_cb):
1. Receive chaincode object from callback to loop.load(). ie: my_cb(e, chaincode)
1. You can now deploy your chaincode (if needed) with chaincode.deploy(func, args, null, cb)
1. Use dot notation on chaincode to call any of your chaincode functions ie:

```js
		// The functions below need to exist in your actual chaincode GoLang file(s) 
		chaincode.query.getRow(['a'], cb);              //will read variable "a" from current chaincode state
		chaincode.invoke.write(['a', 'test'], cb);    //will write to variable "a"
		chaincode.invoke.remove(['a'], cb);           //will delete variable "a"
		chaincode.invoke.add([ARGS], cb);    //calls my custom chaincode function init_marbles() and passes it ARGS
```

## Example

```js
	// Step 1 ==================================
	var loopf = require('theloop-fabric');
	var loop = new loopf(/*logger*/);             //you can pass a logger such as winston here - optional
	var chaincode = {};

	// ==================================
	// configure ibc-js sdk
	// ==================================
	var options = 	{
		network:{
			peers:   [{
				"api_host": "xxx.xxx.xxx.xxx",
				"api_port": "xxxxx",
				"api_port_tls": "xxx",
				"id": "xxxxxx-xxxx-xxx-xxx-xxxxxxxxxxxx_vpx"
			}],
			users:  [{
				"enrollID": "user1",
				"enrollSecret": "xxxxxxxx"
			}],
			options: {							//this is optional
				quiet: true, 
				timeout: 60000
			}
		},
		chaincode:{
            git_url: 'gitlab.yellofg.com/popldo/theloop-demo/chaincode/theloopfirst',
            deploy_name: 'd0231ed3cdd8b0f349809606d74559b934f7ea7fdcc6791c97fbffb67d9c119b9578bf713ab89303982e5d346f202fe7494b2eefebc17d107fe9f7e8ed69efb4',
            invoke :["init","createAccount","AddBalance","createAsset","transferAsset","Invoke"],
            query :["GetAccount","GetAsset","GetAllAssets","GetAllAccountList","GetTransactionLog","query"],
            version : "github.com/hyperledger/fabric/core/chaincode/shim"
		}
	};
	
	// Step 2 ==================================
	loop.load(options, cb_ready);

	// Step 3 ==================================
	function cb_ready(err, cc){								//response has chaincode functions
		app1.setup(ibc, cc);
		app2.setup(ibc, cc);
	
	// Step 4 ==================================
		if(cc.details.deployed_name === ""){				//decide if I need to deploy or not
			cc.deploy('init', ['99'], null, cb_deployed);
		}
		else{
			console.log('chaincode summary file indicates chaincode has been previously deployed');
			cb_deployed();
		}
	}

	// Step 5 ==================================
	function cb_deployed(err){
		console.log('sdk has deployed code and waited');
		chaincode.query.read(['a']);
	}
```
	
** Winston **
```js

var winston = require('winston');
var logger = new (winston.Logger)({
    transports: [
        new (winston.transports.Console)(),
        new (winston.transports.File)({ filename: 'var/log/console.log' })
    ]
});

var Ibc1 = require('theloop-fabric');
var ibc = new Ibc1(logger);

```


***

## <a name="loopjs"></a>theloop-fabric Documentation
### Usage

Example with standard console logging:
```js
	var loopjs = require('theloop-fabric');
	var loop = new loopjs();
```

### loop.load_chaincode(options, [callback])
Load the chaincode you want to use. 
It will be downloaded and parsed. 
The callback will receive (e, obj) where `e` is the error format and `obj` is the chaincode object.
"e" is null when there are no errors.
The chaincode object will have dot notation to the functions in the your chaincode. 

Ex:

```js
	var options = 	{
        git_url: 'gitlab.yellofg.com/popldo/theloop-demo/chaincode/theloopfirst',
        invoke :["init","createAccount","AddBalance","createAsset","transferAsset","Invoke"],
        query :["GetAccount","GetAsset","GetAllAssets","GetAllAccountList","GetTransactionLog","query"],
        version : "github.com/hyperledger/fabric/core/chaincode/shim",
		deployed_name: null   //[optional] this is the hashed name of a deployed chaincode.  if you want to run with chaincode that is already deployed set it now, else it will be set when you deploy with the sdk
	};
	loop.load_chaincode(options, cb_ready);
```

### loop.network(arrayPeers, [options])
Set the information about the peers in the network.
This should be an array of peer objects. 
The optional options parameter should be an object with the field `quiet` and/or `timeout`.
- quiet = boolean - when true will print out only minimal HTTP debug information. Defaults `true`.
- timeout = integer - time in ms to wait for a http response. Defaults `60000`.
- tls = boolean - when `false` will use HTTP instead of HTTPS. Defaults `true`.

Ex:

```js
	var peers = [
		{
			"api_host": "xxx.xxx.xxx.xxx",
			"api_port": "xxxxx",
			"api_port_tls": "xxx",
			"id": "xxxxxx-xxxx-xxx-xxx-xxxxxxxxxxxx_vpx"
		}
	]
	loop.network(peers, {quiet: false, timeout: 120000});
```

### loop.save(path [callback])
Save the [Chaincode Summary File](#ccsf) to a path.

Ex:

```js
	loop.save('./');
```

### loop.clear([callback])
Clear any loaded chaincode files including the downloaded chaincode repo, and [Chaincode Summary File](#ccsf).

Ex:

```js
	loop.clear();
```

### loop.chain_stats([callback])
Get statistics on the network's chain.  

Ex:

```js
	loop.chain_stats(function(e, stats){
		console.log('got some stats', stats);
	});
```

Example Chain Stats:

```js
	{
		"height": 10,
		"currentBlockHash": "n7uMlNMiOSUM8s02cslTRzZQQlVfm8wKT9FtL54o0ywy6BkvPMwSzN5R1tpquvqOwFFHyLSoW44n6rkFyvAsBw==",
		"previousBlockHash": "OESGPzacJO2Xc+5PB2zpmYVM8XlrwnEky0L2Ghok9oK1Lr/DWoxuBo2WwBca5zzJGq0fOeRQ7aOHgCjMupfL+Q=="
	}
```

### loop.block_stats(id, [callback])
Get statistics on a particular block in the chain.  

Ex:

```js
	loop.block_stats(function(e, stats){
		console.log('got some stats', stats);
	});
```

Example Response:

```js
	{
		"transactions": [
			{
				"type": 3,
				"chaincodeID": "EoABNWUzNGJmNWI1MWM1MWZiYzhlMWFmOThkYThhZDg0MGM2OWFjOWM5YTg4ODVlM2U0ZDBlNjNiM2I4MDc0ZWU2NjY2OWFjOTAzNTg4MzE1YTZjOGQ4ODY4M2Y1NjM0MThlMzMwNzQ3ZmVhZmU3ZWYyMGExY2Q1NGZmNzY4NWRhMTk=",
				"payload": "CrABCAESgwESgAE1ZTM0YmY1YjUxYzUxZmJjOGUxYWY5OGRhOGFkODQwYzY5YWM5YzlhODg4NWUzZTRkMGU2M2IzYjgwNzRlZTY2NjY5YWM5MDM1ODgzMTVhNmM4ZDg4NjgzZjU2MzQxOGUzMzA3NDdmZWFmZTdlZjIwYTFjZDU0ZmY3Njg1ZGExORomCgtpbml0X21hcmJsZRIHcng2YXRzcBIFZ3JlZW4SAjM1EgNCb2I=",
				"uuid": "b3da1d08-19b8-4d8c-a116-b46defb07a7c",
				"timestamp": {
					"seconds": 1453997627,
					"nanos": 856894462
				}
			}
		],
		"stateHash": "81ci8IAOeDh0ZwFM6hE/b3SfXt4tnZFemib7sI95cOsNcYMmtRxBWRBA7qnjPOCGU6snBRsFVnAliZXUigQ03w==",
		"previousBlockHash": "tpjUh4sgbaUQFO8wm8S8nrm7yCrBa4rphIiujfaYAlEVfzI8IZ0mjYMf+GiOZ6CZRNWPmf+5bekmGIfr0H6zdw==",
		"nonHashData": {
			"localLedgerCommitTimestamp": {
			"seconds": 1453997627,
			"nanos": 868868790
			}
		}
	}
```

### loop.switchPeer(peerIndex)
The SDK will default to use peer[0].  This function will switch the default peer to another index.  

Ex:

```js
	loop.switchPeer(2);
```
	
### loop.register(peerIndex, enrollID, enrollsecret, maxRetry, [callback])
Only applicable on a network with security enabled. 
`register()` will register against peer[peerIndex] with the provided credentials.
If successful, the peer will now use this `enrollID` to perform any http requests.
- peerIndex = integer - position of peer in peers array (the one you fed loop.networks()) you want to register against.
- enrollID = string - name of secure context user.
- enrollSecret = string - password/secret/api key of secure context user.
- maxRetry = integer - number of times to retry this call before giving up.

Ex:

```js
	loop.register(3, 'user1', 'xxxxxx', 3, my_cb);
```

### loop.monitor_blockheight(callback)
This will call your callback function whenever the block height has changed.
ie. whenever a new block has been written to the chain.
It will also pass you the same response as in `chain_stats()`.

Ex:

```js
	loop.monitor_blockheight(my_callback);
	function my_callback(e, chainstats){
		console.log('got a new block!', chainstats);
	}
```

### loop.get_transaction(udid, [callback])
Get information about a particular transaction ID.

Ex:

```js
	loop.get_transaction('d30a1445-185f-4853-b4d6-ee7b4dfa5534', function(err, data){
		console.log('found trans', err, data);
	});
```

***
***

##<a name="ccfunc"></a>Chaincode Functions
- Chaincode functions are dependent on actually be found inside your Go chaincode
- My advice is to build your chaincode off of the Marble Application one.  This way you get the basic CRUD functions below:

### chaincode.deploy(func, args, [options], [enrollID], [callback])
Deploy the chaincode. 
Call GoLang function named 'func' and feed it 'args'.
Usually "args" is an array of strings.
The `enrollID` parameter should be the desired secure context enrollID that has already been registered against the selected peer. 
If left `null` the SDK will use a known enrollID for the selected peer. (this is only relevant in a permissioned network)

Options: 
- save_path = save the [Chaincode Summary File](#ccsf) to 'save_path'. 
- delay_ms = time in milliseconds to postpone the callback after deploy. Default is `40000`

Ex:

```js
	chaincode.deploy('init', ['99'], {delay_ms: 60000}, cb_deployed);
```

### chaincode.query.CUSTOM_FUNCTION_NAME(args, [enrollID], [callback])
Will invoke your Go function CUSTOM_FUNCTION_NAME and pass it `args`. 
Usually `args` is an array of strings.
The `enrollID` parameter should be the desired secure context enrollID that has already been registered against the selected peer. 
If left `null` the SDK will use a known enrollID for the selected peer. (this is only relevant in a permissioned network)

Ex:

```js
	chaincode.query.read(['abc'], function(err, data){
		console.log('read abc:', data, err);
	});
```

### chaincode.invoke.CUSTOM_FUNCTION_NAME(args, [enrollID], [callback])
Will query your Go function CUSTOM_FUNCTION_NAME and pass it `args`. 
Usually `args` is an array of strings.
The `enrollID` parameter should be the desired secure context enrollID that has already been registered against the selected peer. 
If left `null` the SDK will use a known enrollID for the selected peer. (this is only relevant in a permissioned network)

Ex:

```js
	chaincode.invoke.init_marbles([args], function(err, data){
		console.log('create marble response:', data, err);
	});
```

### chaincode.query.read(name, [enrollID], [callback]) *depreciated 4/1/2016*
*This function is only here to help people transition from ibc v0.0.x to v1.x.x.*
*You should create your own read() function in your chaincode which will overwrite this prebuilt one.*
*This function will put the `name` argument into `args[0]` and set `function` to `query`.*
*These are passed to the chaincode function `Query(stub *shim.ChaincodeStub, function string, args []string)`.*

Read variable named name from chaincode state. 
This will call the `Query()` function in the Go chaincode. 
The `enrollID` parameter should be the desired secure context enrollID that has already been registered against the selected peer. 
If left `null` the SDK will use a known enrollID for the selected peer. (this is only relevant in a permissioned network)

***
***

##<a name="formats"></a>Formats
### Chaincode Object
This is the main guy.
It is returned in the callback to load_chaincode() and contains all your cc functions + some of the setup/input data.

```js
	chaincode = 
		{
			query: {
				CUSTOM_FUNCTION_NAME1: function(args, cb){etc...};	//call chaincode function and pass it args
				CUSTOM_FUNCTION_NAME2: function(args, cb){etc...};
				^^ etc...
			}
			invoke: {
				CUSTOM_FUNCTION_NAME1: function(args, cb){etc...};	//call chaincode function and pass it args
				CUSTOM_FUNCTION_NAME2: function(args, cb){etc...};
				^^ etc...
			}
			deploy: function(func, args, path, cb),     //deploy loaded chaincode
			details:{                                   //input options get stored here, sometimes its handy
						deployed_name: '',              //hash of deployed chaincode
						func: {
							invoke: [],                 //array of function names found
							query: []                   //array of function names found
						},
						git_url: '',
						peers: [],                      //peer list provided in network()
						timestamp: 0,                   //utc unix timestamp in ms of parsing
						users: [],                      //users provided in load()
						unzip_dir: '',
						zip_url: '',
			}
		};
```

### Error Format

```js
	{
		name: "input error",
		code: 400,
		details: {msg: "did not provide git_url"}
	};
```
	
### <a name="ccsf"></a>Chaincode Summary File
This file is used internally. 
It is created in loop.load_chaincode() and updated with chaincode.deploy(). 
A copy can be saved elsewhere with loop.save(path). 
I found it handy in niche cases, but it will probably be unhelpful to most developers. 

```js
	{
        "details":{
        "deployed_name":"9173a0757fca260a7765fc279d10f23148833dd9244d93f7f23aca9449825f7d7a8a015676e91f75c77a7336d066371728e998e455da52cca17d09a35ac35f11",
            "func":{
            "invoke":[
                "init",
                "createAccount",
                "AddBalance",
                "createAsset",
                "transferAsset",
                "Invoke"
            ],
                "query":[
                "GetAccount",
                "GetAsset",
                "GetAllAssets",
                "GetAllAccountList",
                "GetTransactionLog",
                "query"
            ]
        },
        "git_url":"gitlab.yellofg.com/popldo/theloop-demo/chaincode/theloopfirst",
            "options":{
            "quiet":true,
                "timeout":60000,
                "tls":false
        },
        "peers":[
            {
                "name":"vp0-dev_vp0...:5000",
                "api_host":"127.0.0.1",
                "api_port":5000,
                "api_port_tls":5000,
                "id":"dev_vp0",
                "tls":false,
                "enrollID":"admin"
            }
        ],
            "timestamp":1466399970682,
            "users":[
            {
                "enrollId":"admin",
                "enrollSecret":"Xurw3yU9zI0l"
            },
            {
                "enrollId":"WebAppAdmin",
                "enrollSecret":"DJY27pEnl16d"
            }
        ],
            "unzip_dir":"",
            "version":"github.com/hyperledger/fabric/core/chaincode/shim",
            "zip_url":""
    }
```

#FAQ

*loop.load() appears to ignore all of my users for secure context. Then it complains it found "No membership users" and never registers with a Peer!*

- Correct behavior of `loop.load()` is to remove any enrollIDs that do not contain 'type_1' in their name. 
This is to conform to the OBC Peer spec of what enrollIDs a dev's app should use. 
If this is not applicable for your network, you can easily create your own version of `loop.load()` for your needs. 
I would copy the code found in `loop.load()` then modify it to fit your own needs. 
Everything important that `loop.load()` calls is exposed in this module. 


*How exactly do I write chaincode?*

- We have a "hello world" like tutorial for chaincode over at [Learn Chaincode](https://github.com/IBM-Blockchain/learn-chaincode)

*I'm getting error code 2 in my deploy response?*

- Your chaincode has build issues and is not compiling. Manually build it in your local machine to get details.

*I'm getting error code 1!*
- The shim version your chaincode import has is not the same as the shim the peer is running. ie you are probably running 'Hyperledger' peer code and sending it chaincode with a shim pointing to "OBC-Peer". 

*This package is based by Ibm-blockchain-js
 https://github.com/IBM-Blockchain/ibm-blockchain-js

