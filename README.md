# hapi-account

> Hapi server with friendly RESTful and front-end APIs

## Scope

1. Expose a RESTful API for all common account functionality for both, users and admins
2. Expose a client API, wrapping the RESTful API.
3. Allow both to be configured at once (e.g. validation)

## Current state

This is all Dream Code at this point, nothing is implemented yet. Let us know what you think!
At at least the [front-end API](client) needs to be fixed by August 1st, when we start
with the implementation.

## User client

To load the `hapi-account` front-end client, add `<script src="/account.js"></script>` to
your HTML page. The `account` object becomes available, with the following properties and
methods:

```js
account.username
account.isSignedIn()

account.signUp()
account.confirm()
account.signIn()
account.signOut()
account.request()
account.get()
account.fetch()
account.update()
account.validate()
```

Full specifications:

- [account user client](client)
- [account admin client](client/admin)

## Server API

```js
var Hapi = require('hapi')
var hapiAccount = require('hapi-account')

var options = {
  adapter: {
    couchDb: 'http://admin:secret@localhost:5984'
  }
})

server.register({register: hapiAccount}, options, function (error) {
  // server is ready
});

server.connection({
  port: 8000
});

server.start(function () {
  console.log('Server running at %s', server.info.uri);
});
```

Full specifications:

- [account server](server)
