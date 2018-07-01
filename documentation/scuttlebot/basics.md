# Scuttlebot - Basics

* [Install the database](#install-the-database)
  - [Start the server](#start-the-server)
* [Open a client](#open-a-client)
* [Publish a message](#publish-a-message)
* [Read your log](#read-your-log)
* [Create secondary users](#create-secondary-users)
* [Encrypt Messages](#encrypt-messages)
  - [Publish an encrypted message](#publish-an-encrypted-message)
  - [Decrypt a message](#decrypt-a-message)
  - [Is a message encrypted?](#is-a-message-encrypted?)
* [Sync via WIFI](#sync-via-wifi)

## Install the database

```bash
  npm install -g scuttlebot
```

Install the database to your device, not to a host. Each user runs their own Scuttlebot.

### Start the server

```bash
  sbot server
```

Scuttlebot server must be running for any of its other commands, or apps, to work.

## Open a client

If you're using Scuttlebot from a JS application, you will need to open a client connection.

> If you are using the CLI, you can [skip this step](#)

```js


var ssbClient = require('ssb-client')
ssbClient(function (err, sbot) {
  if (err)
    throw err

  // sbot is now ready. when done:
  sbot.close()
})

```

Currently, Scuttlebot's API client is only available for node.js applications.

See also:
- [SSB-Client API](#)

## Publish a message

Every user has an append-only log of JSON messages, called their feed.

Use this command to publish your first message:

```bash
sbot publish --type post --text "Hello, world!"
```

You can do the same with javascript:

```js
  sbot.publish({
    type: 'post',
    text: 'Hello, world!'
  }, function (err, msg) {
    // 'msg' includes the hash-id and headers
  })
```

See also:
- [Publish API](#)
- [Post Messages](#)

## Read your log

Use this command to get your ID:
Bash:
```bash
sbot whoami
```

Javascript:
```js
sbot.whoami(function (err, info) {
  // info.id
})
```

Now, use that ID to get your message history:

Bash:
```bash
sbot createUserStream --id {yourId}
```

Javascript:
```js
var pull = require('pull-stream')
pull(
  sbot.createUserStream({ id: yourId }),
  pull.collect(err, msgs) {
    if (err)
      throw err
    console.log(msgs)
  })
)
```

The output will look something like this:

```json
{
  "key": "%vhP8tyeB+7cVLbIHd6wEd36AVEcUsZgwTYigpcx6Qn0=.sha256",
  "value": {
    "previous": "%NA/4By9K3L0OmVS2eD8le05uUW94ukDNbX16V3ZApi8=.sha256",
    "author": "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519",
    "sequence": 2,
    "timestamp": 1439158769923,
    "hash": "sha256",
    "content": {
      "type": "post",
      "text": "Hello, world!"
    },
    "signature": "EQngCchOejwfWAcZ2Xgr5QR6iquBQVlF++1/ZOLlRJQfyj4TxHk6MHRUKV/o7L35h2zfL1K+Il991JxrxCT+BA==.sig.ed25519"
  }
}
```

Notice that your messages are wrapped in an envelope with various header values, including your ID, your signature, and a timestamp.

The `key` value is a sha256 hash of your message. You can `sbot.get()` the message using that key.

See also:
- [createUserStream API](#)
- [Pull streams](#)

## Create secondary users

By default, Scuttlebot uses a "master" identity/feed, which sbot.publish() will append new messages to. If you want to publish to additional feeds, you can load the keypair and then use this library to do so.

```js
var ssbFeed = require('ssb-feed')
var ssbKeys = require('ssb-keys')

// create the new feed
var alice = ssbFeed(sbot, ssbKeys.generate())

// Post to alice's feed
alice.publish({
  type: 'post',
  text: 'hello world, I am alice.'
}, function (err) { ... })
```

See also:
- [SSB - Feed API](#)
- [SSB - Keys API](#)

## Encrypt Messages

Scuttlebot uses a [gossip protocol](#), which means that users download and reshare messages on each other' behalf. As a result, standard connection-based encryption (like HTTPS) is not enough to hide information from non-recipients.

Instead, Scuttlebot uses [end-to-end encryption](#). This is exposed in the JS-only [Scuttlebot.Private API](#).

> You can read the [details of the encryption protocol here](#)

### Publish an encrypted message

Private publishing works just like the usual publish method, except that you specify a list of recipient IDs after the message content. Be sure to include your own ID, so that you'll be able to read the message back.

```js
sbot.private.publish(
  // message:
  {
    type: 'post',
    text: 'Hello, friend!'
  },
  // recipient PKs:
  [
    '@hxGxqPrplLjRG2vtj...wQpS730nNwE=.ed25519',
    '@EMovhfIrFk4NihAKn...8pTxJNgvCCY=.ed25519'
  ],
  // cb:
  function (err, privateMsg) {
    // privateMsg.value.content is
    // an encrypted string
  }
)
```

The message will be encrypted and then published on your log, like any other message. However, only the recipients will be able to decrypt it.

Here's an example of what it will look like:

```json
{
  "key": "%elZpRx8X4LJfc06p4lEpagtC0pZ6JGMYnSchrqro+1o=.sha256",
  "value": {
    "previous": "%KaDR/KnV/pssiekn8wE5Qh/Lqfuecx6lnCJbWaCE3h8=.sha256",
    "author": "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519",
    "sequence": 3290,
    "timestamp": 1458405498947,
    "hash": "sha256",
    "content": "/TDSbpL+OXU0sbHBIZDaXdDe6ATeyv0krt9H47GD6pV41X++tGsaQwbFLvsnCsuInBfE4IToVJ73upO55ObK2trGF6tY+/8MQtDg1sHN3xYDz/GEV56qZKdEV1fEKcyxqhvW/phHEmXBWRkUQnDoUBlIW...Mmj0rRwj.box",
    "signature": "g3S2BPJSX+uiIEawcEDBhgNEMtjkrHDVHamxBeOVrow1rfsJEmX2CZFRyshwjaQecr8EjBDCL2T8Lxllc+gbAA==.sig.ed25519"
  }
}
```

### Decrypt a message

To decrypt a message, you just send its content-string through the unbox method. If you were a recipient, you'll get the content-object back:

```js
sbot.private.unbox(
  privateMsg.value.content,
  function (err, content) {
    // 'content' is now an object
    // (if you were a recipient)
  }
)
```

### Is a message encrypted?

You may have noticed, the encrypted message had a string for its `content` attribute, whereas other messages will usually have an object. This gives a simple heuristic for checking if a message is encrypted or not -- just check the `.content` value type:

```js
function isPlaintext (msg) {
  return (typeof msg.value.content == 'object')
}

function isEncrypted (msg) {
  return (typeof msg.value.content == 'string')
}
```

See also:
- [Private box protocol](#)
- [Scuttlebot.Private API](#)

## Sync via WIFI

Once you turn the Scuttlebot on, it will  automatically detect other Scuttlebot peers on the LAN and starts downloading their messages.

> If you're having trouble with Wifi sync, check if your computer is configured to allow LAN traffic. For instance, in OSX, the Security & Privacy Firewall will block Scuttlebot's LAN-discovery.
