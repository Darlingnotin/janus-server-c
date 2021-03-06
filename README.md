# janus-server-c
Janus Presence Server, C++ implementation

### Building

Build with Qt 5.  Requires linking with the QtNetwork module (included in a typical Qt SDK install).

On Windows/Linux where Qt Creator is installed: open the project file (janus-server-c.pro) and start a release build.  On Linux where no GUI is available (assumes Qt5 SDK is installed):

> qmake -makefile

> make

and a `janus-server-c` binary will be created.  Run with:

> ./janus-server-c

### General Features

| Feature | Description |
| ---------------------- | ---- |
|**Default Listening Port**|5566 (TCP) 5568 (UDP)|
|**Protocol**|Ordinary socket, UTF-8 encoded|
|**Encoding**|UTF-8|
|**Message Format**|JSON objects formatted onto a single line|
|**Message Structure**|<ul><li>Every packet must contain a "method" field</li><li>Every packet can also optionally contain a "data" field</li></ul>|

### Example

You can test that the server is up by using telnet.

> telnet localhost 5566

Note that disabling UDP connections can be done by specifying a udpport of 0:

> ./janus-server-c -udpport 0

Copy and paste the examples below to see how it works (which also serve as protocol documentation):

=================================
### 1. Client -> Server Methods:
=================================

-----------------------
#### 1.1 "logon" Method
-----------------------

Client -> Server Message Example:

This is an example of a message to logon with if the userId "LL" has not been registered:

```json
{"method":"logon","data":{"userId":"LL", "version":"23.4","roomId":"345678354764987457"}}
```

Real example from JanusVR 40.3

```json
{"method":"logon","data":{"userId":"ProudMinna333","version":"40.3","roomId":"e562b2e1339fc08d635d28481121857c"}}
```

This is an example of a message to logon with if the userId "LL" HAS been registered and therefore requires a password (recommend that this is not used until security tightened up in release):

```json
{"method":"logon","data":{"userId":"LL", "version":"23.4","roomId":"345678354764987457","password":"MyPassword"}}
```

version = the client version

roomId = MD5 hash of the room's URL

password = password associated with userId

Server -> Client Response Example:

If everything is OK and you logged in then you will receive: 

```json
{"method":"okay"}
```

and optionally, if the server supports UDP it provides an incoming port to use for UDP datagram packets, udpport (unsigned 16-bit) which defaults to 5568:

```json
{"method":"okay", "data":{"udp":udpport}}
```

If no roomId was found in the logon request:

```json
{"method":"error", "data":{"message":"Missing roomId in data packet"}}
```

If no userId was found in the logon request:

```json
{"method":"okay"} if everything was okay or a {"method":"error", "data":{"message": "Some error string"}}
```

```json
{"method":"error", "data":{"message": "Missing userId in data packet"}}
```

If the userId is already in use:

```json
{"method":"error", "data":{"message": "User name is already in use"}}
```

TODO: Reject incompatable clients  

----------------------------
#### 1.2 "enter_room" Method
----------------------------

When you pass through a portal:

```json
{"method":"enter_room", "data": {"userId":"LL", "roomId": "345678354764987457" }}
```

Real example from JanusVR 40.3:

```json
{"method":"enter_room", "data": {"userId":"LL", "roomId":"e562b2e1339fc08d635d28481121857c"}}
```

----------------------
#### 1.3 "move" Method
----------------------

When the user position has moved:

```json
{"method":"move", "data": {"userId":"LL", "roomId": "345678354764987457", ...} }
```

Data can be anything you like, but must include the userId string.

Real example from JanusVR 40.3:

JanusVR submits extra information in first move call and every so often after the first call.  This is an example of a move call with extra information:

```json
{"method":"move", "data":{"userId":"LL", "pos":"8.38889 -0.267704 -5.83333","dir":"-1 -1.33e-06 9.42e-07","view_dir":"-1 -1.33e-06 9.42e-07","up_dir":"-1.33e-06 1 1.25e-12","head_pos":"0 0 0","avatar":"<FireBoxRoom><Assets><AssetObject id=^head^ src=^http://avatars.vrsites.com/chibii/head_male.obj^ mtl=^http://avatars.vrsites.com/chibii/mtls/head_male3.mtl^ /><AssetObject id=^body^ src=^http://avatars.vrsites.com/chibii/body_male.obj^ mtl=^http://avatars.vrsites.com/chibii/mtls/body_male3.mtl^ /></Assets><Room><Ghost id=^ProudMinna333^ js_id=^3^ scale=^1.700000 1.700000 1.700000^ head_id=^head^ head_pos=^0.000000 0.750000 0.000000^ body_id=^body^ /></Room></FireBoxRoom>"}}
```

Most JanusVR calls only provide a limited data set:

```json
{"method":"move", "data":{"userId":"LL", "pos":"8.38889 -0.267704 -5.83333","dir":"-1 -1.33e-06 9.42e-07","view_dir":"-1 -1.33e-06 9.42e-07","up_dir":"-1.33e-06 1 1.25e-12","head_pos":"0 0 0"}}
```

-----------------------
#### 1.4 "chat" Method
-----------------------

When the user wants to send a text message:

```json
{"method":"chat", "data": {"userId":"LL", "message":"The message"}}
```

The above message will be broadcasted to all clients in the same room as the user, they will all receive a user_chat notification.

Clients may optionally include a "toUserId" string in the data, which specifies the userId to send a private message.  Only the user specified will receive a user_chat notification.

```json
{"method":"chat", "data": {"userId":"LL", "toUserId":"james", "message":"A private message for james"}}
```

You can pass anything through the data field and it will be sent to all clients subscribed to the current room.

Real example from JanusVR 40.3:

```json
{"method":"chat", "data": {"userId":"LL", "message":"The message"}}
```

----------------------------
#### 1.5 "subscribe" Method
----------------------------

When you wish to start receiving events about a room (you are in that room or looking through a portal)

```json
{"method":"subscribe", "data": { "userId":"LL", "roomId": "345678354764987457" }}
```

Real example from JanusVR 40.3:

```json
{"method":"subscribe", "data":{"userId":"ProudMinna333","version":"40.3","roomId":"e562b2e1339fc08d635d28481121857c"}}
```

Will receive the following if everything is OK.

```json
{"method":"okay"}
```

------------------------------
#### 1.6 "unsubscribe" Method
------------------------------

When you no longer wish to receive messages from that room because none of its portals are visible

```json
{"method":"unsubscribe", "data": {"userId":"LL", "roomId": "345678354764987457"}}
```

Will receive the following if everything is OK.

```json
{"method":"okay"} 
```

-------------------------
#### 1.7 "portal" Method
-------------------------

When a user creates a new portal:

```json
{"method":"portal", "data":{"userId":"LL", "url":"http://...", "pos":[1,2,4], "fwd":[0,1,0]}}
```

Real example from JanusVR 40.3:

```json
{"method":"portal", "data":{"userId":"LL", "url":"http://www.vrsites.com","pos":"-7.16883 -0.267702 -6.57243","fwd":"0.967686 0 -0.234104"}}
```

Will receive the following if everything is OK.

```json
{"method":"okay"} 
```

--------------------------------
#### 1.8 "users_online" Method
--------------------------------

Get list of connected users.

Defined in config.js: config.maxUserResults = 100.

```json
{"method": "users_online"} 
```

    List all users online up to config.maxUserResults.

```json
{"method": "users_online", "data": {"maxResults": 50}}
```

    List all users online up to ‘maxResults’  or config.maxUserResults, whichever is smaller.

```json
{"method": "users_online", "data": {"roomId": "xyz"}}
```

    List all users in ‘roomId’ up to config.maxUserResults.

```json
{"method": "users_online", "data": {"maxResults": 50, "roomId": "xyz"}}
```

    List all users in ‘roomId’ up to ‘maxResults’  or config.maxUserResults, whichever is smaller.

Reply:

```json
{"method": "users_online", "data": {"results": 3, "roomId": "xyz", "users": ["Arthur Dent", "Data", "Lore"]}
```


======================================
### 2. Server -> Client Notifications:
======================================

----------------------------------
#### 2.1 "user_moved" notification
----------------------------------

When a user moves in any room that you are subscribed too, will recive notification about your own movement.

```json
{"method":"user_moved","data":{"roomId":"fgdgd","userId":"LL","position":[0,0,0,0,0,0,0]}}
```

Real example from interaction with JanusVR 40.3:

Some user_moved notifications will contain extra information:

```json
{"method":"user_moved","data":{"roomId":"e562b2e1339fc08d635d28481121857c","userId":"ProudMinna333","position":{"pos":"8.38889 -0.267704 -5.83333","dir":"-1 -1.33e-06 9.42e-07","view_dir":"-1 -1.33e-06 9.42e-07","up_dir":"-1.33e-06 1 1.25e-12","head_pos":"0 0 0","avatar":"<FireBoxRoom><Assets><AssetObject id=^head^ src=^http://avatars.vrsites.com/chibii/head_male.obj^ mtl=^http://avatars.vrsites.com/chibii/mtls/head_male3.mtl^ /><AssetObject id=^body^ src=^http://avatars.vrsites.com/chibii/body_male.obj^ mtl=^http://avatars.vrsites.com/chibii/mtls/body_male3.mtl^ /></Assets><Room><Ghost id=^ProudMinna333^ js_id=^3^ scale=^1.700000 1.700000 1.700000^ head_id=^head^ head_pos=^0.000000 0.750000 0.000000^ body_id=^body^ /></Room></FireBoxRoom>"}}}
```

However, most will only contain something like the following:

```json
{"method":"user_moved","data":{"roomId":"e562b2e1339fc08d635d28481121857c","userId":"ProudMinna333","position":{"pos":"8.38889 -0.267704 -5.83333","dir":"-1 -1.33e-06 9.42e-07","view_dir":"-1 -1.33e-06 9.42e-07","up_dir":"-1.33e-06 1 1.25e-12","head_pos":"0 0 0"}}}
```

----------------------------------
#### 2.2 "user_chat" notification
----------------------------------

When a user says something in text chat.

```json
{"method":"user_chat", "data":{"message":"The message", "userId":"LL"}}
```

Real example from interaction with JanusVR 40.3:

```json
{"method":"user_chat", "data":{"roomId":"69de79e1077103cb59d1a890e96c7ef2","userId":"ProudMinna333","message":"hello!"}}
```

------------------------------------------------
#### 2.3 "user_leave"/"user_enter" notification
------------------------------------------------

When a user changes room:

```json
{"method":"user_leave", "data":{"userId":"LL","roomId":"oldRoomId"}}
```

```json
{"method":"user_enter", "data":{"userId":"LL","roomId":"newRoomId"}}
```

This is followed up with a "user_moved" event, to specify their position in the new room.

-----------------------------------
#### 2.4 "user_portal" notification
-----------------------------------

When a user creates a portal:

```json
{"method":"user_portal", "data":{"userId":"LL","roomId":"345678354764987457","url":"http://...", "pos":[0,0,0], "fwd":[0,1,0]}}
```

Real example from interaction with JanusVR 40.3:

```json
{"method":"user_portal", "data":{"roomId":"e562b2e1339fc08d635d28481121857c","userId":"ProudMinna333","url":"http://www.vrsites.com","pos":"-7.16883 -0.267702 -6.57243","fwd":"0.967686 0 -0.234104"}}
```

-----------------------------------
#### 2.4 "user_disconnected" notification
-----------------------------------

When a user disconnects:

```json
{"method":"user_disconnected","data":{"userId":"LL"}}
```
