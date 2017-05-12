# Socket.io & Clouds

### 12.5.2017 @ Wunderdog Tekkisessio

## Jussi Kinnula

---

# Basics

___

# Realtime communications

Instead of polling data from server with REST/XMLRPC/etc type of interface, realtime communications enable push. In another words in client-server communication server can push some data to the client at any time.

___

# WebSockets

WebSocket is a computer communications protocol, providing full-duplex communication channels over a single TCP connection. The WebSocket protocol was standardized by the IETF as [RFC 6455](https://tools.ietf.org/html/rfc6455) in 2011.

___

# Socket.io

Socket.io is a JavaScript library for realtime web applications. It enables realtime, bi-directional communication between web clients and servers. Socket.io primarily uses the WebSocket protocol with polling as a fallback option.

---

# Let's do some code

___

### Server

```
var path = require('path');
var app = require('express')();
var server = require('http').createServer(app);
var io = require('socket.io')(server);

var routing = { ... };
for (var route in routing) {
  app.get(route, function(req, res) {
    res.sendFile(path.join(
      __dirname,
      routing[req.originalUrl]
    ));
  });
}

io.on('connection', function(socket) {
  console.log('Client connected');

  socket.on('chat message', function(msg) {
    console.log('Received message: '
      + `${msg.message} from ${msg.nickname}`);
    io.emit('chat message', {
      nick: msg.nickname,
      message: msg.message
    });
  });
});

server.listen(PORT);
```

---

# Demo #1

___

# The Problem

When running cloud application in many instances, Socket.io communication is isolated in each instance (e.g. instances cannot broadcast to other instances).

---

# Possible Solutions

___

### Solution #1: Hypervisor

Create a "hypervisor", which is a single instance web service running Socket.io server. Other web services can broadcast through it.

```
var hypervisor = require('socket.io-client')(HYPERVISOR_URL);

io.on('connection', function(socket) {
  socket.on('chat message', function(msg) {
    hypervisor.emit('chat message', msg);
  });
});

hypervisor.on('chat message', function(msg) {
  io.sockets.emit('chat message', msg);
});
```

___

### Solution #1: Hypervisor

## Pros

- Ultra fast (just another socket connection, no databases)
- Can run many hypervisors for different purposes

## Cons

- Cannot scale single hypervisor process to many instances

___

### Solution #2: Database triggers

```
var client = require('pg').Client(DATABASE_URL);
client.connect();
client.query('LISTEN addedrecord');

io.on('connection', function(socket) {
  socket.on('chat message', function(msg) {
    client.query('INSERT INTO table VALUES($1)',
      [msg], function(err, result) {});
  });
});

client.on('notification', function(event) {
  io.sockets.emit('chat message', event.msg);
});

```

___

### Solution #2: Database triggers

## Pros

- Easy with PostgreSQL and especially if the data (which needs to be updated to clients) is anyway put to a database table

## Cons

- Cannot use default Socket.io methods for instances wide communication
- Data architecture is not practical for any type of communication

___

### Solution #3: DIY memory cache

```
var redis = require('redis');
var sub = redis.createClient(REDIS_URL);
var pub = redis.createClient(REDIS_URL);

io.on('connection', function(socket) {
  socket.on('chat message', function(msg) {
    pub.publish('chat message', msg);
  });
});

sub.on('chat message', function(channel, msg) {
  io.sockets.emit('chat message', msg);
});
sub.subscribe('chat message');
```

___

### Solution #3: DIY memory cache

## Pros

- Can limit broadcasted messages between instances to only selected message types
- Can run logic part in worker processes

## Cons

- Cannot use default Socket.io methods for instances wide communication

___

### Solution #4: Socket.io Adapter

```
var redis = require('redis');
var sub = redis.createClient(REDIS_URL);
var pub = redis.createClient(REDIS_URL);
var adapter = require('socket.io-redis');
io.adapter(adapter({ pubClient: pub, subClient: sub }));
```

...or just

```
var adapter = require('socket.io-redis');
io.adapter(adapter(REDIS_URL));
```

___

### Solution #4: Socket.io Adapter

## Pros

- Easy to use (with `socket-io.redis` existing adapter)
- Generally Socket.io adapters can be tailored to fit any project

## Cons

- Easy for simple stuff, hard to design good architecture

---

# Demo #2

---

# Demo #3

---

# Thank you!

GitHub repository (Demo #1 & Demo #2):
https://github.com/jussikinnula/socket-io-and-clouds

GitHub repository (Demo #3):
https://github.com/jussikinnula/ngx-socketio-chat-example

Slides:
https://jussikinnula.github.io/socket-io-and-clouds-wunderdog-tekki-20170512
