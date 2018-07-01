# Scuttlebot - Social

* [Update your profile](#update-your-profile)
  - [Get your ID](#get-your-id)
  - [Set your username](#set-your-username)
  - [Set a profile pic](#set-a-profile-pic)
* [View a profile](#view-a-profile)
* [Follow users](#follow-users)
  - [Unfollow users](#unfollow-users)
* [Query the social graph](#query-the-social-graph)
  - [List degrees-of-connection between users](#list-degrees-of-connection-between-users)
* [Join a pub](#join-a-pub)
  - [Pubs](#pubs)
  - [Joining a pub](#joining-a-pub)

## Update your profile

Each user has a profile, comprised of data published by themself and others. The profile content is controlled by [About messages](#).

### Get your ID

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

### Set your username

There is no global registry, so you can choose any name you want. (It's recommended you stay alphanumeric.)

Bash:

```bash
sbot publish --type about --about {yourId} --name {name}
```

Javascript:

```js
sbot.publish({
  type: 'about',
  about: yourId,
  name: name
}, cb)
```

Replace `yourId` and `name` with your data.

Here's an example (don't copy this directly):

Bash:

```bash
sbot publish \
  --type about \
  --about "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519" \
  --name "bob"
```

Javascript:

```js
sbot.publish({
  type: 'about',
  about: '@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519',
  name: 'bob'
}, cb)
```

### Set a profile pic

First, [add the image file to the blobstore](#). Then, publish this message with the outputted file-ID:

Bash:

```bash
sbot publish \
  --type about \
  --about {yourId} \
  --image.link {fileId} \       # required
  --image.width {widthInPx} \   # optional, but recommended
  --image.height {heightInPx} \ # optional, but recommended
  --image.name {fileName} \     # optional, but recommended
  --image.size {sizeInBytes} \  # optional, but recommended
  --image.type {mimeType}       # optional, but recommended
```

Javascript:

```js
sbot.publish({
  type: 'about',
  about: yourId,
  image: {
    link: fileID,       // required
    width: widthInPx,   // optional, but recommended
    height: heightInPx, // optional, but recommended
    name: fileName,     // optional, but recommended
    size: sizeInBytes,  // optional, but recommended
    type: mimeType      // optional, but recommended
  }
}, cb)
```

Here's an example (don't copy this directly):

Bash:

```bash
sbot publish \
  --type about \
  --about "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519" \
  --image.link "&NfP4H4NZCfiPQ6AZ6fEmilbFL8Hz3wTQVeaxbCnNEt4=.sha256" \
  --image.size 347856 \
  --image.type "image/png" \
  --image.width 512 \
  --image.height 512
```

Javascript:

```js
sbot.publish({
  'type': 'about',
  'about': '@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519',
  'image': {
    'link': '&NfP4H4NZCfiPQ6AZ6fEmilbFL8Hz3wTQVeaxbCnNEt4=.sha256',
    'size': 347856,
    'type': 'image/png',
    'width': 512,
    'height': 512
  }
})
```

See also:
- [About messages](#)
- [Links](#)

## View a profile

To see somebody's profile, you fetch all of their 'about' messages. You can then accept the last `name` and `image` value they published.

This is how you get the self-published about messages:

Bash:
```bash
sbot links \
  --source {userId} \
  --dest {userId} \
  --rel about \
  --values
```

Javascript:
```js
var pull = require('pull-stream')
pull(
  sbot.links({
    source: userId,
    dest: userId,
    rel: 'about',
    values: true
  }),
  pull.collect(function (err, msgs) { ... })
)
```

This query will get all of the 'about' messages that the user has published about themselves. If you want to get 'about' messages published by everyone, remove the `source` parameter.

Here's an example usage (don't copy this directly):

Bash:

```bash
sbot links \
  --source "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519" \
  --dest "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519" \
  --rel about \
  --values
```

Javascript:

```js
var pull = require('pull-stream')
pull(
  sbot.links({
    source: '@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519',
    dest: '@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519',
    rel: 'about',
    values: true
  }),
  pull.collect(function (err, msgs) { ... })
)
```

And some of the output:

```json
{
  "source": "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519",
  "rel": "about",
  "dest": "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519",
  "key": "%NatgQ6Y0iHIGogGgUN5wHlYH9afOWFJY8hW4SRlsGaE=.sha256",
  "value": {
    "previous": "%hDNjq+IeXWuP4Qeju38q2CgF+oTqRG5k16FqVGaYg3s=.sha256",
    "author": "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519",
    "sequence": 257,
    "timestamp": 1443544832442,
    "hash": "sha256",
    "content": {
      "type": "about",
      "about": "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519",
      "name": "bob",
      "image": {
        "link": "&NfP4H4NZCfiPQ6AZ6fEmilbFL8Hz3wTQVeaxbCnNEt4=.sha256",
        "size": 347856,
        "type": "image/png",
        "width": 512,
        "height": 512
      }
    },
    "signature": "Rp8E4H8fz4hALwN6PuiRURJZVWMHxe0fAlV+M3kW0JMV049+Ga72BpYQgQcNOAEZqhtvmXaSS9uHuGl6/RyZBQ==.sig.ed25519"
  }
}
```

See also:
- [About messages](#)
- [Links API](#)

## Follow users

To stay in sync with somebody, you follow their feed. Scuttlebot will search the network for new messages by your followed users.

Following is controlled by [Contact messages](#) that are published on your feed.

Bash:

```bash
sbot publish --type contact --contact {userId} --following
```

Javascript:

```js
sbot.publish({
  type: 'contact',
  contact: userId,
  following: true
}, cb)
```

Example usage (don't copy this directly):

Bash:

```bash
sbot publish \
  --type contact \
  --contact "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519" \
  --following
```

Javascript:

```js
sbot.publish({
  type: 'contact',
  contact: '@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519',
  following: true
}, cb)
```

### Unfollow users

Bash:

```bash
sbot publish --type contact --contact {userId} --no-following
```

Javascript:

```js
sbot.publish({
  type: 'contact',
  contact: userId,
  following: false
}, cb)
```

Example usage (don't copy this directly):

Bash:

```bash
sbot publish \
  --type contact \
  --contact "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519" \
  --no-following
```

Javascript:

```js
sbot.publish({
  type: 'contact',
  contact: '@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519',
  following: false
}, cb)
```

See also:
- [Contact messages](#)
- [Links](#)

## Query the social graph

You can see who follows whom by calling the `friends.all` method. It will provide an object of `{ userId => [followedIds] }`.

Bash:

```bash
sbot friends.all
```

Javascript:

```js
sbot.friends.all(function (err, graph) {
  // ...
})
```

Example output:

```json
{
  "@ptAi6iWwMtqkbpO6YEsLXhucEZkqDgmffCCE0HZYXmk=.ed25519": {
    "@8HsIHUvTaWg8IXHpsb8dmDtKH8qLOrSNwNm298OkGoY=.ed25519": true,
    "@uRECWB4KIeKoNMis2UYWyB2aQPvWmS3OePQvBj2zClg=.ed25519": true,
    "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519": true,
    "@ye+QM09iPcDJD6YvQYjoQc7sLF/IFhmNbEqgdzQo3lQ=.ed25519": true
  },
  "@N8xBJlwMzP/eKNz50i0Eq81qtg/S70YkifNA0mBzbKE=.ed25519": {
    "@D0GsAaMyt96Ze3q1YiiuzWhPkyou2fVTUgw8Xr+G7Jo=.ed25519": true
  },
  "@QgHY6P7NGW9qFJVjAMl36EkapanG3iyjCBo+8/ZwXTY=.ed25519": {
    "@uRECWB4KIeKoNMis2UYWyB2aQPvWmS3OePQvBj2zClg=.ed25519": true
  }
}
```

### List degrees-of-connection between users

If you want to find out how connected you (or somebody else) is, you can use the `friends.hops` method. This will list all known user-ids, and how many social-connections away they are.

For instance:
  - Somebody you follow will be 1 hop away.
  - Somebody followed by someone you follow will be 2 hops away.(_a friend of a friend_)
  - etc

Bash:

```bash
sbot friends.hops {userId}
```

Javascript:

```js
sbot.friends.hops(userId, function (err, list) {
  // ...
})
```

Example output:

```json
{
  "@hxGxqPrplLjRG2vtjQL87abX4QKqeLgCwQpS730nNwE=.ed25519": 0,
  "@jpq43ybzjW+u2lABZCPD8OVVvp85CKeY4qSlXTA31j4=.ed25519": 1,
  "@V/0ZcEFyPagUhs6Cqc9Su98rpzna4zBRXXbAZOtF6p4=.ed25519": 1,
  "@Tc/1cgullpQYaQk/JKNwDAsfniVADcU9lulwYXrGsos=.ed25519": 1,
  "@GLH9VPzvvU2KcnnUu2n5oxOqaTUtzw+Rk6fd/Kb9Si0=.ed25519": 1,
  "@6ilZq3kN0F+dXFHAPjAwMm87JEb/VdB+LC9eIMW3sa0=.ed25519": 1,
  "@9sCXwCJZJ9doPcx7oZ1gm7HNZapO2Z9iZ0FJHJdROio=.ed25519": 2,
  "@GslEFXx7Oz3ooO6z4+6JRzdtKpzeNqb4aYv8M9wGqoU=.ed25519": 2,
  "@yARQ14OOZXgI8OU8UsgQhZSSmzPbuwCgC5gXxmRAhkE=.ed25519": 2,
  "@bCbTcYMmSwYbbiDBUnSbhobzYnayKesq0LNmTDSvR6Y=.ed25519": 2,
  "@v4pOKqdvSpv/Rm/Eb+g5gCfsBXVVzKbmMT/F6IW7lds=.ed25519": 2,
  "@+gpZcBi1p1E2omC9L3bXUWduYWfrUZqwWcednMxy+Rw=.ed25519": 3,
  "@OweKXqiKl49z9pzp764z0xvzOfHqTacYxuLubLzrCgQ=.ed25519": 3
}
```

See also:
- [Scuttlebot.Friends API](#)
- [Follow users](#)

## Join a pub

If you want to connect with users across the net, you need to be followed by a Pub server. 

### Pubs

 Some homes and offices can't accept peer-to-peer connections over the Internet. You can solve this with ["NAT Hole-Punching"](https://en.wikipedia.org/wiki/Hole_punching_(networking)), but it can be slow and sometimes fails. Instead, Scuttlebot solves this with Pubs.

Pubs are bot-users that have public IP addresses, and who follow real users. They dont have any special authority, but they do keep your data backed up and easy-to-access. You can have more than one, and change pubs any time, but you need at least one to reach friends over the net.

> If you want to only sync by Wifi, or dont want to sync with anyone at all, you can skip this step.

### Joining a pub

First get an invite-code from a Pub operator, or [create your own Pub](#). You can ask for invites via the #scuttlebutt channel, on Freenode.

To use the invite, follow this command:

Bash:

```bash
sbot invite.accept {code}
```

Javascript:

```js
sbot.invite.accept(code, function (err) {
  // ...
})
```

Your Scuttlebot will now connect to, and sync with, the Pub. Other users can then sync with the pub to receive your feed.

See also:
- [Scuttlebot.Invite API](#)
