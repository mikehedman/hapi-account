# account user client

The account user client is a generic front-end API, that wraps all
user-specific RESTful API following the [account REST API](http://docs.accountrestapi.apiary.io/)
specifications. There is a separate [account admin client](admin).


## At a glance

```js
var account = new Account({
  // base URL of REST API
  url: 'https://example.com/account/api',
  validate: function(options) {
    // custom validations, must return undefined or an array of errors
    
    //For me, this needs to be explained more.  Is this a function that will be called when the user signs up, like to validate the password length, etc.?  If so, then it should have a different name than the one listed 13 lines below "account.validate" which seems to have many more capabilities.
  }
})

account.username
account.isSignedIn()

account.signUp()
account.confirm()
//funny how github markdown decided that 'confirm' needs to be blue, but all the others are black
account.signIn()
account.signOut()
account.request()
account.get()
account.fetch()
account.update()
account.validate()
```


## Properties

### account.username

Returns username of currently signed in user, otherwise `undefined`.


## Methods


### account.isSignedIn()

Returns `true` / `false`, depending on whether user is signed in or not

```js
if (account.isSignedIn()) {
  console.log("Hey there, ", account.username)
} else {
  console.log("Welcome, visitor!")
}
```


### account.signUp()

Arguments:

- options (object) _required_
  - options.username (string) _required_
  - options.password (string) _required_
  - options.profile (object)

Returns `Promise`:

- resolves with `username`
- rejects with
  - http request error
  - username taken error
  - validation error (e.g. password to short)
//I don't think "schema" is the right word here. To me, schema implies database.

Simple example

```js
account.signUp({
  username: 'john@example.com',
  password: 'password'
})
```

With custom profile properties

```js
account.signUp({
  username: 'john@example.com',
  password: 'password',
  profile: {
    name: 'John Doe',
    birthday: '1984-05-09'
  }
})
```


### account.confirm()

Arguments:

- token (string) _required_

Returns `Promise`:

- resolves with `username`
//We should think through the user flow here, since this is the client api.  Specifically, would the user be signed in when this function is called?  If so, then they signed in and got a reminder: "Hey, you've signed up, but please check your mail for your confirmation token.", and then they would call this function. In this case, returning the username makes sense.
But if they have not yet signed in, then to me it's a bit more problematic to return the username because it *feels* like the confirm process also signed them in...but they didn't supply their password.  
My thought is that calling confirm() when not signed in is probably not a needed use case.
This is probably going to force us to consider what *exactly* to do when a user signs in but they are unconfirmed.  

- rejects with
  - http request error
  - token invalid error

```js
account.confirm('token123')
```


### account.signIn()

Arguments:

- options (object) _required_
  - options.username (string) _required_
  - options.password (string) _required_

Returns `Promise`:

- resolves with `username`
- rejects with
  - http request error
  - username not found error
  - invalid password error
// and perhaps an "unconfirmed" option as well

```js
account.signIn({
  username: 'john@example.com',
  password: 'password'
})
```


### account.signOut()

Returns `Promise`:

- resolves with `username`
- rejects with http request error


### account.request()

Arguments

- what (string) _required_
  (`what` is case insensitive and ignores all non-chars, so `password reset`, `password:reset`, `PASSWORDRESET` are all the same)
- options (object)

Returns `Promise`:

- resolves with request response
- rejects with
  - http request error
  - unknown request error
  - invalid request error

Request a password reset.

```js
account.request('password reset', { email: 'joe@example.com' })
```

Request username reminder

```js
account.request('username reminder', { email: 'joe@example.com' })
```

Request an account upgrade
//todo - give list of 'stock' actions (like password:reset), and in another doc need to specify how to define developer supplied actions (like 'upgrade')

```js
account.request('upgrade', { plan: 'pro' })
```


### Account / Profile properties

Every user account has an id, a username and a password. Custom
properties can be added to the account profile. Custon Account and Profile properties
can be restricted to read-only or write-only, or to secure read / write.

User data can be passed to `account.signUp({profile: { ... }})`, or read /
fetched / updated via `account.get('profile')`. `account.fetch('profile')`
and `account.update({ profile: { ... }})`.

#### read-only

Example: a `"plan"` property can be set in the back-end, after
a user signed up for a paid plan. The user can read it, but not
change it.

#### write-only

A property that shall behave like `"password"`. It can be set,
but not received.

#### secure read / write

Custom data can be sensitive. For example, an API key for a 3rd
party API should not be readable without passing a password,
even if the current user is signed in.

#### account.get()

- property (string)

Returns `Object` with all properties, or `property` value if `property` is set

Get all locally cached profile properties (does not include secure-read properties)

```js
var account = account.get()
console.log('Greetings to  ' + account.profile.address.city)
```

Get single property from the local cache. Use `.` to get a nested property.

```js
var city = account.get('profile.address.city')
console.log('Greetings to  ' + city')
```

Get selected properties

```js
var properties = account.get(['username', 'profile.address.city'])
console.log('Hey there ' + properties.username + '. What is happening in ' + properties.profile.address.city + '?')
```


#### account.fetch()

- property (string)
- options (object)
  - password (string) _required if property is secure-read_

Returns `Promise`

- resolves with all non-secure-read profile properties, or `property` value if `property` is set
- rejects with
  - http request error
  - unauthenticated error

Fetch profile properties from the server (does not include secure-read properties)

```js
account.fetch()
.then(function (account) {
  console.log('Greetings to  ' + account.profile.address.city)
})
```

Fetch one account property from the server

```js
account.fetch('profile.nickName')
.then(function (nickName) {})
```

Fetch multiple properties, which can be secure-read

```js
account.fetch(['profile.doorPin', 'profile.secretNickName'], {password: 'currentpassword'})
.then(function (properties) {})
```

### account.update()

- changedProperties (object) _required_
- options (object)
//the options argument is not always required, such as when updating a profile field
  - authOptions.token (string) _required if not signed in_
  - authOptions.password (string) _required to change secure-write properties, unless token set_

Returns `Promise`:

- resolves with `undefined`
- rejects with
  - http request error
  - unauthenticated error
  - validation error

Set a new password with a token received from a password reset request

```js
account.update({
  password: 'newpassword'
}, {
  token: 'abc4567'
})
```

Change the current user's password

```js
account.update({
  password: 'newpassword'
}, {
  password: 'currentpassword'
})
```

Change the current user's username

```js
account.update({
  username: 'newusername'
}, {
  password: 'currentpassword'
})
```

Update one property

```js
account.update({profile: {nickname: 'FunkyFoo'}})
.then(function () {})
.catch(function (error) {})
//should this .then ... .catch block be included in the three previous examples as well?  It's in 3, and not in 3 - might be confusing.  My suggestion would be to put it on the first example (to show the complete signature), and then not included it on the remaining examples for brevity.
```

Update multiple properties

```js
account.update({profile: {nickName: 'FunkyFoo', motto: 'Hoodie Woogie'}})
.then(function () {})
.catch(function (error) {})
```

Update secure-write properties

```js
account.update({profile: {keys: {github: 'abc4567'}}}, {password: 'currentpassword'})
.then(function () {})
.catch(function (error) {})
```

### account.validate()

- options (object) _required_
  - username (string)
  - password (string)
  - profile (object)
  - remote (boolean)

If `options.remote` is false, only client side validations will be performed. Returns
- `undefined` if there are no errors
- `[error]` array if there are errors

If `options.remote` is true, server-side validations will also be performed. Returns `Promise` and
- resolves with `property` value or `properties`
- rejects with `[error]` array

Check if username is valid

```
var errors = account.validate({username: 'foo'})
if (errors) {
  console.log('username foo is invalid', errors)
}
```

Check if username is valid, and still available

```
account.validate({username: 'foo', remote: true})
.then(function () {})
.catch(function (errors) {})
```

Check profile properties

```
account.validate({profile: {name: 'Foo'}, remote: true})
.then(function () {})
.catch(function (errors) {})
```
