# hapi-account server

The `hapi-account` server is a hapi plugin, that exposes a REST-ful API.
It also exposes dynamically bundled & pre-initialised scripts for the
[account user client](../client) & [account admin client](../client/admin)

## RESTful API

See current work in progress here http://docs.accountrestapi.apiary.io/
Comment / send PRs for [apiary.apib](https://github.com/gr2m/account-rest-api/blob/master/apiary.apib).

Have a glance (might be outdated, check links above)

```
# sign in, check session, sign out
PUT /session
GET /session
DELETE /session

# sign up, get / update / destroy account
PUT /session/account
GET /session/account
PATCH /session/account
DELETE /session/account

# get / update profile
GET /session/account/profile
PATCH /session/account/profile

# get / set secured profile properties (requires user password)
PUT /session/account/profile/{property}
GET /session/account/profile/{property}
DELETE /session/account/profile/{property}

# password resets / username reminder, user account confirmation
PUT /password-reset/{contact}
PUT /username-reminder/{contact}
POST /confirmation

# admins only: manage accounts
POST /accounts
GET /accounts
GET /accounts/{username}
PATCH /accounts/{username}
DELETE /accounts/{username}
```
