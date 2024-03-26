# Intro

It is a social network, with almost minimum features, with keeping the privacy in mind. 
The decentralized forum uses a blockchain to prevent any kind of censorship and management over other people's posts.
each group of users can have their own network, in other words there is no meant to be a single network, like bitcoin.
Since the nodes are connected with HTTP protocol, then is it simple to hide node behind http servers, along with many other sites on hostings. there is no need to have a single valid IP for each node anymore altough it can work with IP instead of hostname.

A usual forum software, like phpbb, vbulletin or nodebb, have a database with at least several tables like Posts, Users, Threads etc.
each time a user is sending a post, a new record to the `Posts` table is added. or when users edits his/her own profile and appropriated field in the `Users` table is edited. in both cases, a change to to database happens. we can abstract this change as a `DbQuery`. At its simplest for the DbQuery is a single string like this:

``` Insert into Posts (Content, UserId) values ("Hello world" , 1)```

We could make a blockchain with these requests. Then chain the consequetive rows of blockchain in order to prevent edits and tampers and finally we have a blockchain. of course the blockchain will not be simple as above, in next we do talk about the details.

So the block chain is a single table, which holds the requests. The nodes only and only keep that single table. The app on the client want to see the threads, then it need to grab whole blockchain, create empty database locally with like SqLite, apply each query to DB and finally it will have the posts and users in it's own database. 

So there are two types of devices, nodes and clients. each client can be managed centrally with a single person, that way users can trust that website or app, and use that website. very much like brokers (like binance) role in bitcoin network.

# Decentralized Discussion Forum
Designed for anonimity, on highly censored areas. even IPs are not recorded...

- Anonimity: High
- App Features: MID

It is not planned to have only one decentralized forum, there could be multiple forums for multiple group of users completely separated. but as we use pub key hash for user ids, then a single user can have same account in multiple forums.
Nodes could have firewall, i.e. network is limited to known nodes who know eachother.

# Considerations

Note that need to minimize data on servers, also clients.
A client maybe do not want to see whole data, maybe only want to see a specific category, or even a single topic. How to do that? is it even possible?


## Client Side
Here is database tables. note that this DB is on the client PC. i.e. every client have its own copy. Client side tables are build and filled on each instalation. the blockchain is setted as a layer on top of the client.


### Forum_Posts (actions: add, edit, remove)
posts in threads by users

fields:

- ID: BlockId is used as ID
- ThreadID: ID of thread
- Owner: ID of owner (or sender)
- TimeStamp: epoch with milisecs, in UTC
- Content-Type: two types, local and remote. local is post body, remote is like http or https link
- Content: could be post body or link to an http
- Remote-Content-Length: if the content is remote, the length of remote content, used for dead link or edited content detection
- Remote-Content-Hash: if the content is remote, the length of remote content, used for edited content detection

### Forum_Thread (actions: add, edit, remove)

threads (title and owner)

fields:

- ID
- Owner
- Title (the title of thread)
- (moved to v2) IsEncrypted (if it is encrypted - placeholder moved to V2)
- TimeStamp

### Forum_Categories (action: add?, edit)

Categories (sub forums)

fields:

- ID
- Owner
- Title
- Description
- Parent

### Forum_PrivateMessages (actions: add, edit, remove)

private messages (encrypted with receiver's public key)

fields:

- ID
- Owner
- Receiver
- ContentEncrypted (encrypted with receiver key)

### Forum_Members (actions: add, edit, remove)
members (member info like name, etc)

fields:

- ID (~removed~ it is useless but let it be)
- Owner
- Avatar
- Bio
- PubKey : no need anymore, Owner field is pub key

There are two columns which are presented on all tables,

- ID: this is the ID of block who created the record, as long as we do not have two blocks with same ID on block chain, no concern for duplicated data.
- Owner: this is the pubkey of block who created the record.

Note: these two columns are not variables, i.e. they are authorized in broadcast phase. not worry for fakes or tampered data.


## Procedure of new installation on new client
A client is just wants to signup in the forum. He simply downloads the blockchain, then the software reads block by block and fills the tables on the client side. Then user can browse the tables and see the posts.
Processing the blocks of blockchain on client side to build the client DB. the blocks are validated, authenticated and authorized in server side, yet we do those again on client side. for details check the appropriated section. after that filtering, do as this:

for each block:
- Do the preValidation, Validation, Authentication and authorization if any failed simply pass the block and do not apply it to the client side DB
- Take BlockId (i.e. the block hash)
- Take SenderId (i.e. the pub key id of sender)

Apply the block

## Broadcast

Broadcasts to network. this is synced with the network.
This table is not edited. only adds are permited. this is heart of censorship prevention. the broadcast is same as Database row, nothing more.

### Example BroadCast

A node send this broadcast into the network. It asks other nodes to add this block to the end of blockchain.

```
TargetTable: Forum_Post
Action: Create
ID: abs123ABC // TODO: what is this id?
Params:
    {'Time':15654948321,
     'ContentType':HttpsUrl, //this is single byte enumerable: 0: unknown, 1: text, 2: http, 3: https, etc
      Content:'sdf.com/file1.bin', //together with ContentType it forms: https://sdf.com/file1.bin
      RemoteContentLength:'1254744', //in cases the content is https or http url, this is content length of remote http file (for example 1000000000 is 1GB, if remote file is 1GB file)
      RemoteContentHash: 'abcds'     //in cases the content is https or http url, this is content hash of remote http file,
      //note: RemoteContentLength and RemoteContentHash are used for end users when viewing the post to ensure the http link is still same as original.
    }
Sender: 1ad6w98q4dq6e13qw1e65146 (pub key of sender)
Nonce: a nonce
LastHash: hash of last row of blockchain
Hash: 1234 (hash of all fields, except signature, for proof of work stuff also as ID of broadcast)

Signature: 123abc (hash of content, signed by pvt key of Sender)
```

receiving node do calculate the hash of record using last hash on table, does the validation, authentication and authorization. if all ok, then adds the record to end of blockchain.


## Encrypted Threads (end to end) -- moved to V2
Topics can be encrypted, and only certain users be able to see the content. 
To do that, a user can create a topic, and mark it as encrypted. then encrypt with a random key, and then send the key to participants with pvt message or email (key is encrypted with receiver pub key so other users cannot see the key, this would be the end to end encryption)

## Syncing Mechanism
with something like http/s and proof of work. or gossip protocol.

# ServerSide
Server only have to take care of the block chain. ~So only and only single table or even a key value list.~
~there are two tables, a `Database` table and an `Owners` table, which the second one is a calculated table.~
there is single table block chain

## BlockChain

There is a single DB table used as blockchain.

Records table (the block chain)

- TargetTable (enumerable: post, rection, etc)
- Action (enumerable: Create, Update, Delete - CRUD minus R)
- Params (binary data: parameters, see the rules in put block section)
- Owner (pubkey of sender)
- Nonce (a nonce to make hash good, 16 byte would be good enough i think)
- Hash: hash of record, look at Hash (BlockId) section below.
- Signature (signature of digest. encrypted with pvt key, decrypted with sender pub key, used to ensure sender is who it claims - authentication)

### Hash (BlockId)

Also known as `BlockID`, the hash of record except Signature: 

Hash( `PreviousHash`, `Table`, `Action`, `Params`, `Owner`, `Nonce`)


This should contain all fields except signature, in order to prevent any edits.
it is safe to exclude signature from hash. it is mandatory to exclude the signature, because validation also signature is calculated from this field. so theorically impossible to add `Signature`. just forget about including signature to this field!

Primary key is `Hash` column (the `BlockID`)

~~it could combine to `Blocks` table, make it have 3 fields.~~

~~Owners table (a calculated table)~~

~~- BlockId: the ID of sent block~~

~~- SenderId: the ID of person who broadcasted the block~~

we do not need OwnersTable i think

## Consistency of blockchain

By word "Consistency" we mean having no tampering in database. To check the consistency of Database it is simple to calculate the Hash of records one by one with previous hash:

```
bool CheckBlockChainConsistency()

var count = select count(*) from BlockChain

var lastHash = 0x'00000000000000' //to be used by genesis block

for i = 0 to count
  var blk = select * from blockchain where rowNumber = i
  var hash = HASH(lastHash,blk.TargetTable,...blk.Nonce)
  if(blk.Hash != hash) return false;
  lashHash = hash

return true;
```

## Comunication

for comunication of nodes together. Prtocol can be over http, maybe like REST. others can get the latest block ID, request for putting a block and get blocks after specific post.
having http can also help hiding in between of other servers when one tries to identify server with port scan and port fingerprint.
many hostings do use shared IP for hosts, so having http instead of IP could be good.
So each node is a HTTP server.

### Put block (for new blocks)
Unlike Bitcoin, each block do not contains multiple transactions. it only contains a single transaction.
Puts a block at the end of blocks

id=abc&message={abc_url_encoded_json}


Checks: 

There are three levels of filtering illegal requests:

- **PreValidation**: ensure no block with same ID exists, and block size to exceet limit
- **Validation**: ensure data in not curropted while transfering
- **Authentication**: ensure data is from who it claims
- **Authorization**: ensure claimed sender is allowed to do the action

if a request pass through all these, then it will be added to blockchain.
if any of these phases failed, then simply exit.


#### PreValidation

##### Check hash of block


There should not be two blocks with same ID on the blockchain. this is important because of the OwnersTable which is a key value pair of BlockId, SenderId for checking on Edit and delete actions. see authorization section below.

pseudo code:

```
bool preValdiate(block)

var r = select count(*) from BlockChain where BlockId == block.BlockID

return r == 0
```

##### Check size of the block

Also another check for testing the size of block in byte. i.e. 100 kb is max block size.
it should be hardcoded, as a node in network can not be differnt than others.


#### Validation

there are two validations:
1- Hash should be true hash of content, i.e. check for `$hash == HASH(content)`
2- it should met the last hash on the block chain.

both can be handled with this code:

```
bool valdiate(block)

var lastHash = select top 1 BlockId from BlockChain
var params = new [] {lastHash, block.ID, block.TargetTable, ..., block.Nonce};
var expectedHash = HASH(params)

return expectedHash == block.Hash

```


#### Authentication
Next we should make sure message came from claiming person (the pub key). to do this we use message signature and check with sender pubkey. 

```
bool authentication(block)

var pub = block.ownerPubkey
var hash = block.BlockId

return hash and pub and block.signature are consistent
```

#### Authorization
Next we know that user with public key of `<pub>` is the rightful sender. Each target table have it's own Authorization for each of CUD operations.

there are 3 actions: CRUD without R = CUD

##### Authorization in Create action
if action is create:

forum_posts: always OK
forum_threads: Always OK
forum_catergories: ??
forum_privateMessages: always OK
forum_members: only if not created yet



##### Authorization in Edit and Delete actions

forum_posts: only OK if sender is owner of target post
forum_threads: only OK if sender is owner of target post
forum_catergories: ??
forum_privateMessages: never
forum_members: only OK if sender is owner of target post


if action is edit or delete, then there is need for an extra level of authorization. we must make sure that creator of the row is sender. i.e. person A is not trying to edit or remove person B's post.
To do this we must know the target ID of record which is going to be edited. then compair the owner of that record with sender of the block.

pseudo code:


```
bool authorize(block)

if block.action is Create:
 return true //all users can

var action = 

if block.action is Edit or Delete:
 var targetRecord = GetTargetRecord(block)// since the params is binary, need to parse the params in differnt method, this is actualy the block ID of existing record
 var r = SQL( "select count(*) from BlockChain where BlockId == targetRecord AND Owner == block.Owner)//we check for a block with ID and owner, again duplicated block IDs are impossible due to preValidation phase
 if r > 1 throw exception //it is impossible, since we check for duplication at each broadcast
 if r = 1 return true
 if r = 0 return false

```



~~Each table have a column named owner, which is the ID of users who created that row. On updates and deletes, sender should be same as owner. for each table, there should be a method for processing the broadCast. e.g:~~

Deprecated code: 

~~
```
public static class AuthorizationUtil
{
    Dictionary<int256,int256> Owners;//hash, owner id

    public static bool IsAuthorized(block blk)
    {
        if(blk.Action == Update || Delete)
        {//we need authorization
           if(!Owners.ContainsKey(blk.Hash)) return false;

           if(blk.Sender != Owners[blk.hash]) return false;
        }

        return true;
    }

    public static void AddOwner(block blk)//adds owner to internal dic
    {
        Owners[blk.Hash] = Hash(blk.Sender) // hash sender pub key
    }
}

```
~~

### Get Block(s)

id=abc&count=1000

Gets the message, starting with id, and count

## Extra Notes:
~~Users should not be able to modify other users posts. so broadcasts should be authorized after authentication by signature.
Also when manipulating DB on client side.~~ this issue covered before

BroadCast are like transaction in bitcoin. any user want to do any change on forum database, it should broadcast it's own request.

Should not every user can create categories. categories can be flooded.

we do not use encrypt decrypt of content, we only use signature. encrypted threads moved to V2

for broadcasting blocks, we will use compression as long as it is HTTP it can be compressed with `Content-Type` mime type

### Hashing mechanism
Lets define `Hash(a,b,c,d,...)` where each a,b,c,d have a unique hash of n bytes.

pseudo code for blending technique


```
function Hash(object[] params)

    var buf = new byte[n];

    foreach(var val in params)
        var hash = Hash(val);
        buf = XOR(hash, buf);
        buf = Hash(buf);

    return buf;
```

### What about economics?

What would users pay for? broadcasting their block. who would get paid? whole network? so users should pay each other.
Econimics will help the security

# Client Side

User is able to do these locally (on data accefted on the local device):

- set nickname for other people.
- flag other people posts (like delete - i do not want to see this person's posts anymore)
- verify other users locally (like the blue tick in twitter). helps user identify friends in a large amount of users.

As there is no usernames, and users are identified with the pubkeys, then we should use colors for posts too. for example posts for each user on it's own device can be yello (i can identify my own posts with yellow background). or posts from friends are green, or have a blue tick.


Client side have a sort of api behind local http server. this will abstract the DB layer to be independent of DB type.

Like:

http://127.0.0.1:8088/api/threads/abc123

Result is in json format

UI also could be html/js behind `http://127.0.0.1:8088/`

