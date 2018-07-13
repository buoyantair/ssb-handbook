# Scuttlebot - Advanced

* [Links](#links)
  - [Content-Hash Linking](#content-hash-linking)
* [Publish a file](#publish-a-file)
  - [Add a file to the blobstore](#add-a-file-to-the-blobstore)
  - [Publish the file to the network](#publish-the-file-to-the-network)
* [Read a file](#read-a-file)
  - [Wait for the file to arrive](#wait-for-the-file-to-arrive)
  - [Fetch and read the file](#fetch-and-read-the-file)
* [Feed (send time sort)](#feed-send-time-sort)
  - [Messages ordered by time stamp](#messages-ordered-by-time-stamp)
* [Feed (receive time sort)](#feed-receive-time-sort)
  - [Messages ordered by receive-time](#messages-ordered-by-receive-time)
* [Messages by type](#messages-by-type)
* [Messages by user](#messages-by-user)
* [Watch for messages](#watch-for-messages)
  - [Watch for messages by all users](#watch-for-messages-by-all-users)
  - [Watch for messages by a single user](#watch-for-messages-by-a-single-user)
* [Votes on a message](#votes-on-a-message)
* [Files referenced by a user](#files-referenced-by-a-user)
* [Links from one user to another](#links-from-one-user-to-another)
* [Post threads](#post-threads)

## Links

### Content-Hash Linking

Messages, feeds, and blobs are addressable by specially-formatted identifiers. Message and blob IDs are content-hashes, while feed IDs are public keys.

To indicate the type of ID, a "sigil" is prepended to the string. They are:

- `@` for feeds
- `%` for messages
- `&` for blobs

Additionally, each ID has a "tag" appended to indicate the hash or key algorithm. Some example IDs:

- A feed: `@LA9HYf5rnUJFHHTklKXLLRyrEytayjbFZRo76Aj/qKs=.ed25519`
- A message: `%MPB9vxHO0pvi2ve2wh6Do05ZrV7P6ZjUQ+IEYnzLfTs=.sha256`
- A blob: `&Pe5kTo/V/w4MToasp1IuyMrMcCkQwDOdyzbyD5fy4ac=.sha256`

---

When IDs are found in the messages, they may be treated as links, with the keyname acting as a "relation" type. An example of this:

Bash:

```bash
$ sbot publish --type post \
  --root "%MPB9vxHO0pvi2ve2wh6Do05ZrV7P6ZjUQ+IEYnzLfTs=.sha256" \
  --branch "%kRi8MzGDWw2iKNmZak5STshtzJ1D8G/sAj8pa4bVXLI=.sha256" \
  --text "this is a reply!"
```

Javascript:

```js
sbot.publish({
  type: "post",
  root: "%MPB9vxHO0pvi2ve2wh6Do05ZrV7P6ZjUQ+IEYnzLfTs=.sha256",
  branch: "%kRi8MzGDWw2iKNmZak5STshtzJ1D8G/sAj8pa4bVXLI=.sha256",
  text: "this is a reply!"
})
```

In this example, the `root` and `branch` keys are the relations. SSB automatically builds an index based on these links, to allow queries such as "all messages with a `root` link to this message."

If you want to include data in the link object, you can specify an object with the id in the `link` subattribute:

Bash:

```bash
$ sbot publish --type post \
  --mentions.link "@LA9HYf5rnUJFHHTklKXLLRyrEytayjbFZRo76Aj/qKs=.ed25519" \
  --mentions.name bob \
  --text "hello, @bob"
```

Javascript:

```js
sbot.publish({
  type: "post",
  mentions: { 
    link: "@LA9HYf5rnUJFHHTklKXLLRyrEytayjbFZRo76Aj/qKs=.ed25519",
    name: "bob"
  },
  text: "hello, @bob"
})
```

To query the link-graph, use [the links method](#):

Bash:

```bash
$ sbot links [--source id|filter] [--dest id|filter] [--rel value]
```

Javascript:

```js
pull(sbot.links({ source:, dest:, rel: }), pull.drain(...))
```

You can provide either the source or the destination. Both can be set to a sigil to filter; for instance, using `'&'` will filter to blobs, as `&` is the sigil that precedes blob IDs. You can also include a relation-type filter.

Here are some example queries:

Bash:

```bash
# all links pointing to this message
$ sbot links \
  --dest %6sHHKhwjVTFVADme55JVW3j9DoWbSlUmemVA6E42bf8=.sha256

# all "about" links pointing to this user
$ sbot links \
  --rel about \
  --dest @hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519

# all blob links from this user
$ sbot links \
  --dest "&" \
  --source @hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519
```

Javascript:

```js
// all links pointing to this message
pull(
  sbot.links({
    dest: '%6sHHKhwjVTFVADme55JVW3j9DoWbSlUmemVA6E42bf8=.sha256'
  }),
  pull.drain(...)
)

// all "about" links pointing to this user
pull(
  sbot.links({
    rel: 'about',
    dest: '@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519'
  }),
  pull.drain(...)
)

// all blob links from this user
pull(
  sbot.links({
    dest: '&',
    source: '@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519'
  }),
  pull.drain(...)
)
```

A common pattern is to recursively fetch the links that point to a message, creating a tree. This is useful for creating comment-threads, for instance.

You can do that easily in scuttlebot with [relatedMessages](#).

Bash:

```bash
$ sbot relatedMessages --id {id}
```

Javascript:

```js
sbot.relatedMessages({ id: id }, cb)
```

## Publish a file

To publish a file, you must first add it to Scuttlebot's blob-store. Then, you announce the file by publishing its hash-ID in a message.

### Add a file to the blobstore

Scuttlebot maintains a "blobstore," which is a space of files that are named by their content-hash. Add a file by streaming its content in the `blobs.add` method.

Bash:

```bash
$ cat ./hello.txt | sbot blobs.add &hT/5N2Kgbdv3IsTr6d3WbY9j3a6pf1IcPswg2nyXYCA=.sha256
```

### Publish the file to the network

Adding a file will only affect the local blobstore. To distribute the file, you must also link to it in an unencrypted message.

Bash:

```bash
$ sbot publish --type post --text "checkout this file!" \
  --mentions.link "&hT/5N2Kgbdv3IsT...swg2nyXYCA=.sha256" \
  --mentions.name "hello.txt" \
  --mentions.size 12 \
  --mentions.type "text/plain"
```

Javascript:

```js
sbot.publish({
  type: 'post',
  text: 'checkout [this file!]('+hash+')',
  mentions: [{
    link: hash,        // the hash given by blobs.add
    name: 'hello.txt', // optional, but recommended
    size: 12,          // optional, but recommended
    type: 'text/plain' // optional, but recommended
  }]
}, function (err, msg) {
  // ...
})
```

When peers see the hash-ID in your message, they'll request the blob from your device, and then reshare it to other peers.

See also:
- [Scuttlebot.Blobs API](#)
- [Linking messages](#)
- [Pull streams](#)

## Read a file

When Scuttlebot finds the hash-ID of a file in a message, it will request its contents from the network. You can then read the file from the blobstore.

### Wait for the file to arrive

Files always arrive after the message that first announces it. So, it's a good idea to call `want`, which will return when the file has arrived:

Bash:

```bash
$ sbot blobs.want "&hT/5N2Kgbdv3...XYCA=.sha256"
```

Javascript:

```js
// want will not cb() until the file arrives
sbot.blobs.want(hash, function (err) {
  // ready to read
})
```

### Fetch and read the file

Once the file is available, you can fetch it using 'get':

Bash:

```bash
$ sbot blobs.get "&hT/5N2Kgbdv3...XYCA=.sha256"
hello, world
```

See also:

- [Scuttlebot.Blobs API](#)
- [Linking messages](#)
- [Pull streams](#)

## Feed (send time sort)

### Messages ordered by timestamp

You can fetch all of the messages in your Scuttlebot DB, merged into one stream. To fetch them ordered by the timestamp at time-of-publish, use 'createFeedStream'.

Bash:

```bash
$ sbot feed --reverse
```

Javascript:

```js
var pull = require('pull-stream')
pull(
  sbot.createFeedStream({ reverse: true }),
  pull.collect(function (err, msgs) { ... })
)
```

**Important note**: message timestamps are set by the wall clock of the publisher, and may be incorrect. The "feed" stream should only be used when order is not important.

[-> createFeedStream API](#)

See also:

- [Pull stream API](#)
- [Scuttlebot API](#)

## Feed (receive time sort)

### Messages ordered by receive-time

You can fetch all of the messages in your Scuttlebot DB, merged into one stream. To fetch them ordered by the timestamp at time-of-download, use 'createLogStream'.

Bash:

```bash
$ sbot log --reverse
```

Javascript:

```js
var pull = require('pull-stream')
pull(
  sbot.createLogStream({ reverse: true }),
  pull.collect(function (err, msgs) { ... })
)
```

The receive time of messages will differ for each user. The "log" stream's order will never be the same between two devices.

[-> createLogStream API](#)

See also:

- [Pull stream API](#)
- [Scuttlebot API](#)

## Messages by type

Bash:

```bash
$ sbot logt --type post
```

Javascript:

```js
var pull = require('pull-stream')
pull(
  sbot.messagesByType({ type: 'post' }),
  pull.collect(function (err, msgs) { ... })
)
```

The ordering in `messagesByType` will be the same as the ordering in `createLogStream`.

[→ messagesByType API](#)

See also:

- [Pull stream API](#)
- [Scuttlebot API](#)

## Messages by user

Bash:

```bash
$ sbot createUserStream --id {userId}
```

Javascript:

```js
var pull = require('pull-stream')
pull(
  sbot.createUserStream({ id: userId }),
  pull.collect(function (err, msgs) { ... })
)
```

The order of messages in an individual user's feed will be the same for all devices, and can be trusted.

[-> createUserStream API](#)

See also:
- [Pull stream API](#)
- [Scuttlebot API](#)






## Watch for messages

### Watch for messages by all users

Bash:

```bash
$ sbot log --live
```

Javascript:

```js
var pull = require('pull-stream')
pull(
  sbot.createLogStream({ live: true }),
  pull.drain(function (msg) { ... })
)
```

[-> createLogStream API](#)

### Watch for messages by a single user

Bash:

```bash
$ sbot createUserStream --id {userId} --live
```

Javascript:

```js
var pull = require('pull-stream')
pull(
  sbot.createUserStream({ id: userId, live: true }),
  pull.drain(function (msg) { ... })
)
```

Notice that `pull.drain` is used instead of `pull.collect`, so that new messages are handled immediately.

[-> createUserStream API](#)

See also:
- [Pull stream API](#)
- [Scuttlebot API](#)

## Votes on a message

Bash:

```bash
$ sbot links --dest {messageId} --rel vote --values
```

Javascript:

```js
var pull = require('pull-stream')
pull(
  sbot.links({ dest: messageId, rel: 'vote' }),
  pull.collect(function (err, msgs) { ... })
)
```

[Vote messages](#) put the value and reason data in the link, so we'll get those attributes in the output of `links` without including the full message value. That's why `values: true` is not set.

[-> Links API](#)

See also:
- [Pull stream API](#)
- [Scuttlebot API](#)

## Files referenced by a user

Bash:

```bash
$ sbot links --source {userId} --dest &
```

Javascript:

```js
var pull = require('pull-stream')
pull(
  sbot.links({ source: userId, dest: '&' }),
  pull.collect(function (err, msgs) { ... })
)
```

)

All IDs have a "sigil" character defining what type of object the ID references. (Blobs start with "&", users start with "@", and messages start with "%").

In `links`, you can use the sigil to filter the source or dest by the ID type.

[-> Links API](#)

See also:
- [Pull stream API](#)
- [Scuttlebot API](#)

## Links from one user to another

Bash:

```bash
$ sbot links --source {userId1} --dest {userId2}
```

Javascript:

```js
var pull = require('pull-stream')
pull(
  sbot.links({ source: userId1, dest: userId2 }),
  pull.collect(function (err, msgs) { ... })
)
```

[-> Links API](#)

See also:
- [Pull stream API](#)
- [Scuttlebot API](#)

## Post threads

Bash:

```bash
$ sbot relatedMessages --id {messageId}
```

Javascript:

```js
sbot.relatedMessages({ id: messageId }, function (err, thread) {
  ...
})
```

This will provide a tree-structure showing all messages that link to the given message and its children.

[-> relatedMessages API](#)

See also:
- [Scuttlebot API](#)
