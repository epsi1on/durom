# Server side

## Program Architecture
Server side is a single console software, http server. a single executable file.
Starting point for any possible action is simply an http request from a client.

## Configuration
Each node should know the list of other nodes. It is a secret key (optional) and a http(s) address of the server, or a IP and a host (if host do not have DNS record). also ability to set the custom http headers.

keys in config.json for server

```
{
  "listen-ip": "0.0.0.0",
  "listen-port" : 8080,
  "listen-secret" : "abc123",
  "neighbors":
  [
    {"ip":"1.2.3.4", "port": "2240", "hostname": "abc.com", "secret": "abc123"},
    {"ip":"10.20.30.40", "port": "2240", "hostname": "def.com", "secret": "def123"}
  ]
}
```


## http API

webserver api, for client to sync itself:

### ver
the Version
parameters: none


### stat
The statistics of whole blockchain
no params

### block
gets a single block
params: id


### blocks
gets a range of blocks
param: from, count

### blockInfo

the number of block in blockchain (i'th), total size (byte) before block, total size (byte) after block, total count before block, total count after block
param: blockId

### broadcast

Client or another server want to broadcast a new block to network.

The whole block. details in the other place.

All methods have local changes, just `/broadcast` have both local and network changes. `broadcast` first check for validity and authentication and authorization, if all go well the block should spread to other nodes of network.


## Database
dont know if we use builtin file that store whole database (invented wheel) or simply use things like sqlite or redis.
combination of C# dictionary and a file, plus catching will have highest performance. as long as there is a single table with primary key.


There is a single table database


```
Hash        (64 byte) - primary key

TargetTable (byte)
Action      (byte)
Params      (string-json)
Owner       (128 - byte)
Nonce       (64 byte)
Signature   (64 byte)
```

better to use a custom file format to store whole blocks. indexes also in the runtime as a dictionary.

