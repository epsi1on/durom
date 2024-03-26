# Client side

## Syncing mechanism
Triggered by user each time or on automatic basis. each time cliend give the last block id to server to see if there is new block since that.


## UI user frontend

tables:

Forum_Posts
Forum_Thread
Forum_Categories
Forum_PrivateMessages
Forum_Members

consist of two parts:
- Database abstraction layer: A http server (behind 127.0.0.1) for api (like backend), hosted on localhost (127.0.0.1)
- UI layer: a js code, or C# WF or WPF for frontent.



## API

/thread/{thread-id}
