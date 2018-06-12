loopchain JSON RPC API V2 document
========

<!-- TOC depthFrom:1 depthTo:1 withLinks:1 updateOnSave:1 orderedList:undefined -->

- [Endpoint](#endpoint)
- [icx_sendTransaction](#icxsendtransaction)
- [icx_getTransactionResult](#icxgettransactionresult)
- [icx_getBalance](#icxgetbalance)
- [icx_getBlockByHeight](#icxgetblockbyheight)
- [icx_getBlockByHash](#icxgetblockbyhash)
- [icx_getLastBlock](#icxgetlastblock)

<!-- /TOC -->

# Endpoint
 * Mainnet: https://wallet.icon.foundation/api/v2
 * Testnet: https://testwallet.icon.foundation/api/v2

# icx_sendTransaction

Transfer ICX value from one wallet to another with the fee.

## Request

### JSON RPC data

* ``` jsonrpc```: Fixed as "2.0"

* ``` method```: "icx_sendTransaction"

* ``` id```: An identifier established by the Client that MUST contain a String, Number, or NULL value if included. If it is not included it is assumed to be a notification. The value SHOULD normally not be Null and Numbers SHOULD NOT contain fractional parts.

### Params

* ```from```

    * Wallet address of the sender

    * Type: String

    * Format: ‘hx’ +  40 digit hex string

* ```to```

    * Wallet address of the recipient

    * Type: String

    * Format: ‘hx’ +  40 digit hex string

* ```value```

    * The transfer amount(ICX)

    * Type: String

    * Format: ```0x``` + Hex string

    * Unit: 1/10<sup>18</sup> icx

        * To transfer 1.0 icx, the value must be the hex value of 1.0 X 10<sup>18</sup>.

        * Ex) 1 icx = 10<sup>18</sup> loop = "0xDE0B6B3A7640000"

* ```fee```

    * The fee for the transaction

    * Type: String

    * Format: 0x’ + Hex string string

    * Unit: 1/10<sup>18</sup> icx

    * Currently, it’s 0.01 icx (Jan 25, 2018)

        * Ex) 0.01icx = 10<sup>16</sup> = "0x2386f26fc10000"

* ```timestamp```

    * UNIX epoch time (Begin from 1970.1.1 00:00:00)

    * Type: string

    * Unit: microseconds

    * Meaning: creation time of this transaction to avoid ‘replay attack’.

* ```nonce``` (optional)

    * Type: string

    * Integer value increased by request to avoid ‘replay attack’

* ```tx_hash```

    * See ‘Additional information.’

* ```signature```

    * See ‘Additional information.’

### Additional information

#### Generate tx_hash

```tx_hash``` has bundled the parameters of the transaction. Pseudo code to generate ```tx_hash``` is as follows:

```python
def create_dictionary_with_key( from, to, value, fee):
    temp_dict = dict()
    temp_dict["from"] = from
    temp_dict[“to”] = to
    temp_dict[“value”] = value
    temp_dict[“time”] = get_UNIX_time_in_string()
    temp_dict[“nonce”] = get_nonce_value()
    return temp_dict


def get_dump_string(key_sorted_data):
    str_tmp = “icx_sendTransaction”
    for key, value in key_sorted_data.item():
           str_tmp += “.”
           str_tmp += str(key)
           str_tmp += “.”
           str_tmp += str(value)

    return str_tmp


tx_tmp = create_dictionary_with_key(from, to, value, fee)
key_sorted_data = get_dictionary_with_sorted_by_key_in_unicode(tx_tmp)
dumped_string = get_dump_string(key_sorted_data)
tx_hash = hashlib.sha3_256(dumped_string.encode()).hexdigest()
```

#### Generate signature

The ```signature``` is electronic signature data derived from the wallet’s private key of the sender. Pseudo code to generate the signature is as follows:

``` python
signature = SECP256k1.ecdsa_sign_recoverable(msg=binascii.unhexlify(tx_hash),
                                                   raw=True,
                                                   digest=hashlib.sha3_256)
serialized_sig = SECP256k1.ecdsa_recoverable_serialize(signature)
sig_message = b''.join([serialized_sig[0], bytes([serialized_sig[1]])])
signature = base64.b64encode(sig_message).decode()
```


### Example of request

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 2,
    "params": {
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "hxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32",
        "value": "0xde0b6b3a7640000",
        "fee": "0x2386f26fc10000",
        "timestamp": "1516942975500598",
        "nonce": "8367273",
        "tx_hash": "4bf74e6aeeb43bde5dc8d5b62537a33ac8eb7605ebbdb51b015c1881b45b3aed",
        "signature": "VAia7YZ2Ji6igKWzjR2YsGa2m53nKPrfK7uXYW78QLE+ATehAVZPC40szvAiA6NEU5gCYB4c4qaQzqDh2ugcHgA="
    }
}
```

## Response

* ```response_code```: JSON RPC error code.
* ```tx_hash```: Hash data of the result. Use icx_getTransactionResult to get the result.
* ```id```: It MUST be the same as the value of the id member in the Request Object.

    * If there was an error in detecting the id in the Request object (e.g. Parse error/Invalid Request), it MUST be Null.

### Successful case

``` json
{
    "jsonrpc": "2.0",
    "result": {
        "response_code": 0,
        "tx_hash": "4bf74e6aeeb43bde5dc8d5b62537a33ac8eb7605ebbdb51b015c1881b45b3aed"
    },
    "id":2
}
```



### Unsuccessful case

```json
{
    "jsonrpc": "2.0",
    "result": {
        "message": "create tx message",
        "response_code": -11
    },
    "id": 2
}
```


# icx_getTransactionResult

 Request the result of previous transaction.

## Request

### JSON RPC data

* ```jsonrpc```: Fixed as "2.0"
* ```method```: "icx_getTransactionResult"
* ```id```: An identifier established by the Client that MUST contain a String, Number, or NULL value if included. If it is not included it is assumed to be a notification. The value SHOULD normally not be Null and Numbers SHOULD NOT contain fractional parts.

### Params

* ```tx_hash```: Hash string from the result of icx_sendTransaction

#### Example of request

``` json
{
    "jsonrpc" : "2.0",
    "method": "icx_getTransactionResult",
    "id": 2,
    "params": {
        "tx_hash": "e670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
    }
}
```


## Response

 * ```id```: It MUST be the same as the value of the id member in the Request Object.
    * If there was an error in detecting the id in the Request object (e.g. Parse error/Invalid Request), it MUST be Null.
* ``` response_code```: JSON RPC error code.
* ``` response```: Code or message. See the following explanation for code.
     * SUCCESS = 0
     * EXCEPTION = 90
     * NOT_INVOKED = 2  # means pending
     * NOT_EXIST = 3  # possibly means failure
     * SCORE_CONTAINER_EXCEPTION = 9100

### Successful case

```json
{
    "jsonrpc": "2.0",
    "result": {
        "response_code": 0,
        "response": {
            "code": 0
        }
    },
    "id": 2
}
```


### Unsuccessful case

```json
{
    "jsonrpc": "2.0",
    "result": {
        "response_code": -6,
        "message": "Invalid transaction hash."
    },
    "id": 2
}
```

# icx_getBalance

Get the balance of the wallet address.

## Request

### JSON RPC data

* ```jsonrpc```: Fixed as "2.0"
* ```method```:  "icx_getBalance"
* ```id```: An identifier established by the Client that MUST contain a String, Number, or NULL value if included. If it is not included it is assumed to be a notification. The value SHOULD normally not be Null and Numbers SHOULD NOT contain fractional parts.

### Params
* ```address```: Address of wallet.

### Example of request

```json
{
    "jsonrpc" : "2.0",
    "method": "icx_getBalance",
    "id": 1234,
    "params": {
        "address": "hxaa688d74eb5f98b577883ca203535d2aa4f0838c"
    }
}
```


## Response

* ```id```: It MUST be the same as the value of the id member in the Request Object.
    * If there was an error in detecting the id in the Request object (e.g. Parse error/Invalid Request), it MUST be Null.
* ``` response_code```: JSON RPC error code.
* ```response```: code or message. See the following explanation for code.
	* SUCCESS = 0

### Successful case

```json
{
    "jsonrpc": "2.0",
    "result": {
        "response": "0x0",
        "response_code": 0
    },
    "id": 1234
}
```


### Unsuccessful case

```json
{
    "jsonrpc": "2.0",
    "id": 123,
    "result": {
        "code": -32602,
        "message": "Invalid params"
    }
}
```


# icx_getBlockByHeight

 Get block information and transactions by block height

## Request
### JSON RPC data

* ```jsonrpc```: Fixed as "2.0"
* ```method```:  "icx_getBlockByHeight"
* ```id```: An identifier established by the Client that MUST contain a String, Number, or NULL value if included. If it is not included it is assumed to be a notification. The value SHOULD normally not be Null and Numbers SHOULD NOT contain fractional parts.

### Params
* ```height```: Block height

### Example of request

```json
{
    "jsonrpc" : "2.0",
    "method": "icx_getBlockByHeight",
    "id": 1234,
    "params": {
        "height": "1234"
    }
}
```

## Response

* ```id```: It MUST be the same as the value of the id member in the Request Object.
    * If there was an error in detecting the id in the Request object (e.g. Parse error/Invalid Request), it MUST be Null.
* ``` response_code```: JSON RPC error code.
* ```block```: Block information
  - ```version``` : Version of block structure
  - ```height```: Height of current block
  - ```prev_block_hash```: Block hash of previous block
  - ```merkle_tree_root_hash```:Merkle tree root hash of transactions in this block.
  - ```time_stamp``` : UNIX epoch time (Begin from 1970.1.1 00:00:00)
    * Unit: microseconds
  - ```block_hash```: Hash of block
  - ```peer_id```: Id of node
  - ```signature```: See [this](#request) section
  - ```confirmed_transaction_list```: List of transactions in this block.
    - ```from``` : See [this](#request) section
    - ```to``` : See [this](#request) section
    - ```value``` : See [this](#request) section
    - ```nonce``` : See [this](#request) section
    - ```tx_hash``` : See [this](#request) section
    - ```signature``` : See [this](#request) section
    - ```method```: Fixed as "icx_sendTransaction"

### Successful case

```json
{
  "jsonrpc": "2.0",
  "id": 1234,
  "result": {
    "response_code": 0,
    "block": {
      "version": "0.1a",
      "prev_block_hash": "48757af881f76c858890fb41934bee228ad50a71707154a482826c39b8560d4b",
      "merkle_tree_root_hash": "fabc1884932cf52f657475b6d62adcbce5661754ff1a9d50f13f0c49c7d48c0c",
      "time_stamp": 1516498781094429,
      "block_hash": "1fcf7c34dc875681761bdaa5d75d770e78e8166b5c4f06c226c53300cbe85f57",
      "height": 3,
      "peer_id": "e07212ee-fe4b-11e7-8c7b-acbc32865d5f",
      "signature": "MEQCICT8mTIL6pRwMWsJjSBHcl4QYiSgG8+0H3U32+05mO9HAiBOhIfBdHNm71WpAZYwJWwQbPVVXFJ8clXGKT3ScDWcvw==",
      "confirmed_transaction_list": [
        {
          "from": "hx63fac3fc777ad647d2c3a72cf0fc42d420a2ba81",
          "to": "hx5f8bfd603f1712ccd335d7648fbc989f63251354",
          "value": "0xde0b6b3a7640000",
          "fee": "0x2386f26fc10000",
          "nonce": "0x3",
          "tx_hash": "fabc1884932cf52f657475b6d62adcbce5661754ff1a9d50f13f0c49c7d48c0c",
          "signature": "cpSevyvPKC4OpAyywnoNyf0gamHylHOeuSPnLjkyILl1n9Xo4ygezzxda8LpcQ6K1rmo4JU+mXdh+Beh+/mhBgA=",
          "method": "icx_sendTransaction"
        }
      ]
    }
  }
}
```
### TIP : To get the total block height of current block loopchain
 Request ```height``` as ```-1```. Then it returns the top block in blockchain. You can get the total block height by using ```result.height.height```.

# icx_getBlockByHash

Get block information and transactions by block hash.

## Request
### JSON RPC data

* ```jsonrpc```: Fixed as "2.0"
* ```method```:  "icx_getTransactionByAddress"
* ```id```: An identifier established by the Client that MUST contain a String, Number, or NULL value if included. If it is not included it is assumed to be a notification. The value SHOULD normally not be Null and Numbers SHOULD NOT contain fractional parts.

### Params
* ```hash```: Hash of block

### Example of request

```json
{
   "jsonrpc" : "2.0",
   "method": "icx_getTransactionByAddress",
   "id": 1234,
   "params": {
       "hash": "af5570f5a1810b7af78caf4bc70a660f0df51e42baf91d4de5b2328de0e83dfc"
   }
}
```

## Response

* ```id```: It MUST be the same as the value of the id member in the Request Object.
   * If there was an error in detecting the id in the Request object (e.g. Parse error/Invalid Request), it MUST be Null.
* ``` response_code```: JSON RPC error code.
* ```block```: Block information
 - ```version``` : Version of block structure
 - ```height```: Height of current block
 - ```prev_block_hash```: Block hash of previous block
 - ```merkle_tree_root_hash```:Merkle tree root hash of transactions in this block.
 - ```time_stamp``` : UNIX epoch time (Begin from 1970.1.1 00:00:00)
   * Unit: microseconds
 - ```block_hash```: Hash of block
 - ```peer_id```: Id of node
 - ```signature```: See [this](#request) section
 - ```confirmed_transaction_list```: List of transactions in this block.
   - ```from``` : See [this](#request) section
   - ```to``` : See [this](#request) section
   - ```value``` : See [this](#request) section
   - ```nonce``` : See [this](#request) section
   - ```tx_hash``` : See [this](#request) section
   - ```signature``` : See [this](#request) section
   - ```method```: Fixed as "icx_sendTransaction"

### Successful case

 ```json
 {
   "jsonrpc": "2.0",
   "id": 1234,
   "result": {
     "response_code": 0,
     "block": {
       "version": "0.1a",
       "prev_block_hash": "48757af881f76c858890fb41934bee228ad50a71707154a482826c39b8560d4b",
       "merkle_tree_root_hash": "fabc1884932cf52f657475b6d62adcbce5661754ff1a9d50f13f0c49c7d48c0c",
       "time_stamp": 1516498781094429,
       "block_hash": "1fcf7c34dc875681761bdaa5d75d770e78e8166b5c4f06c226c53300cbe85f57",
       "height": 3,
       "peer_id": "e07212ee-fe4b-11e7-8c7b-acbc32865d5f",
       "signature": "MEQCICT8mTIL6pRwMWsJjSBHcl4QYiSgG8+0H3U32+05mO9HAiBOhIfBdHNm71WpAZYwJWwQbPVVXFJ8clXGKT3ScDWcvw==",
       "confirmed_transaction_list": [
         {
           "from": "hx63fac3fc777ad647d2c3a72cf0fc42d420a2ba81",
           "to": "hx5f8bfd603f1712ccd335d7648fbc989f63251354",
           "value": "0xde0b6b3a7640000",
           "fee": "0x2386f26fc10000",
           "nonce": "0x3",
           "tx_hash": "fabc1884932cf52f657475b6d62adcbce5661754ff1a9d50f13f0c49c7d48c0c",
           "signature": "cpSevyvPKC4OpAyywnoNyf0gamHylHOeuSPnLjkyILl1n9Xo4ygezzxda8LpcQ6K1rmo4JU+mXdh+Beh+/mhBgA=",
           "method": "icx_sendTransaction"
         }
       ]
     }
   }
 }
 ```

# icx_getLastBlock

 Get last block information and transactions.

## Request
### JSON RPC data

 * ```jsonrpc```: Fixed as "2.0"
 * ```method```:  "icx_getTransactionByAddress"
 * ```id```: An identifier established by the Client that MUST contain a String, Number, or NULL value if included. If it is not included it is assumed to be a notification. The value SHOULD normally not be Null and Numbers SHOULD NOT contain fractional parts.

### Params
 * ```hash```: Hash of block

### Example of request

 ```json
 {
	 "jsonrpc" : "2.0",
 	 "id": 1234,
   "method": "icx_getLastBlock"
 }
 ```

## Response

 * ```id```: It MUST be the same as the value of the id member in the Request Object.
    * If there was an error in detecting the id in the Request object (e.g. Parse error/Invalid Request), it MUST be Null.
 * ```result```: Block information
	 - ```response_code```:JSON RPC error code.
	 - ```block_data_json``` : Block data
		 - ```prev_block_hash```: The hash of previous block hash
		 - ```height```: The height of this block
		 - ```block_hash```: The hash of last block.

### Successful case

  ```json
  {
		"jsonrpc": "2.0",
		"id": 1234,
    "result": {
        "response_code": 0,
        "block_data_json": {
            "prev_block_hash": "",
            "height": "0",
            "block_hash": "af5570f5a1810b7af78caf4bc70a660f0df51e42baf91d4de5b2328de0e83dfc"
        }
    }
  }
  ```
