[![Build Status](https://travis-ci.org/peers/peerjs-server.png?branch=master)](https://travis-ci.org/peers/peerjs-server)

# PeerServer: A server for PeerJS #

PeerServer helps broker connections between PeerJS clients. Data is not proxied through the server.

##[http://peerjs.com](http://peerjs.com)

**If you prefer to use a cloud hosted PeerServer instead of running your own, [sign up for a free API key here](http://peerjs.com/peerserver)**

or

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://www.heroku.com/deploy/?template=https://github.com/peers/peerjs-server)

### Run PeerServer

Install the library:

```bash
$> npm install peer
```

Run the server:

```bash
$> peerjs --port 9000 --key peerjs
```

Or, create a custom server:

```javascript
var PeerServer = require('peer').PeerServer;
var server = PeerServer({port: 9000, path: '/myapp'});
```

Connecting to the server from PeerJS:

```html
<script>
    // No API key required when not using cloud server
    var peer = new Peer('someid', {host: 'localhost', port: 9000, path: '/myapp'});
</script>
```

Using HTTPS: Simply pass in PEM-encoded certificate and key.

```javascript
var fs = require('fs');
var PeerServer = require('peer').PeerServer;

var server = PeerServer({
  port: 9000,
  ssl: {
    key: fs.readFileSync('/path/to/your/ssl/key/here.key'),
    cert: fs.readFileSync('/path/to/your/ssl/certificate/here.crt')
  }
});
```

#### Running PeerServer behind a reverse proxy

Make sure to set the `proxied` option, otherwise IP based limiting will fail.
The option is passed verbatim to the
[expressjs `trust proxy` setting](http://expressjs.com/4x/api.html#app-settings)
if it is truthy.

```javascript
var PeerServer = require('peer').PeerServer;
var server = PeerServer({port: 9000, path: '/myapp', proxied: true});
```

### Combining with existing express app

```javascript
var express = require('express');
var app = express();
var ExpressPeerServer = require('peer').ExpressPeerServer;

app.get('/', function(req, res, next) { res.send('Hello world!'); });

var server = app.listen(9000);

var options = {
    debug: true
}

app.use('/api', ExpressPeerServer(server, options));

// OR

var server = require('http').createServer(app);

app.use('/peerjs', ExpressPeerServer(server, options));

server.listen(9000);
```
### Authentification

Create a custom server:

```javascript
var users = {
	oshimin : "ae3bc5f6f22f15",
	badlee  : "be3ff5c6d52a16"
};
var PeerServer = require('peer').PeerServer;
var server = PeerServer({
	port: 9000, path: '/myapp', proxied: true,
	auth : function(info,next){
		if(info.id in users && users[info.id] === info.token)
			next()
		else
			next("Invalid ID");
	}
	// or list the users in a array : [[userName, token],[userName, token][...]]
	//, auth : [["oshimin","ae3bc5f6f22f15"],["badlee","be3ff5c6d52a16"]]
});

```
Connecting to the server from PeerJS:

```html
<script>
    var peer = new Peer('oshimin', {host: 'localhost', port: 9000, path: '/myapp',token:"ae3bc5f6f22f15"});
</script>
```

### Rooms Validation

Create a custom server:

```javascript
var users = {
	oshimin : ["ae3bc5f6f22f15","private"],
	badlee  : ["be3ff5c6d52a16"]
	foo  : ["bar"]
};
var rooms = ['room','room2','private'];

var PeerServer = require('peer').PeerServer;
var server = PeerServer({
	port: 9000, path: '/myapp', proxied: true,
	auth : function(info,next){
		if(info.id in users && users[info.id][0] === info.token){
			if(users[info.id][1] && users[info.id][1] != info.key)
				next('Invalid Key');
			else
				next();

		}else
			next('Invalid ID')
		
	},
	key : function(info,next){
		if(rooms.some(r=>r == info.key))
			next()
		else
			next("Invalid Key");
	}
	// or list the valid keys in a array [room1,room2,room3[...]]
	//, key : ['room','room2','private']
});

```
Connecting to the server from PeerJS:

```js
// client 1 - oshimin can only use the room 'private'
var peer1 = new Peer('oshimin', {host: 'localhost', port: 9000, path: '/myapp',token:"ae3bc5f6f22f15",key:"private"});

// client 2 - badlee can call oshimin because they are in the same room
var peer2 = new Peer('badlee', {host: 'localhost', port: 9000, path: '/myapp',token:"ae3bc5f6f22f15",key:"private"});

// cllient 3 - foo can't call badlee or oshimin because they aren't is the same room
var peer3 = new Peer('foo', {host: 'localhost', port: 9000, path: '/myapp',token:"bar",key:'room'});
```

### Events

The `'connection'` event is emitted when a peer connects to the server.

```javascript
server.on('connection', function(id,key) { ... });
```

The `'disconnect'` event is emitted when a peer disconnects from the server or
when the peer can no longer be reached.

```javascript
server.on('disconnect', function(id,key) { ... });
```

## Problems?

Discuss PeerJS on our Google Group:
https://groups.google.com/forum/?fromgroups#!forum/peerjs

Please post any bugs as a Github issue.
