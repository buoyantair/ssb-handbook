# Scuttlebot - Message Types

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

## Posts

A post is a text-based message, for a public or private audience. It can be a reply to other posts.

There's not a limit on the length of the text, but all Scuttlebot messages (including their headers) must be less than 8KB.

Javascript:

```js
{
  type: 'post',
  text: String,
  channel: String?,
  root: MsgLink?,
  branch: MsgLink? | MsgLinks?,
  recps: FeedLinks?,
  mentions: Links?
}
```

### Text

`text` is a markdown string that includes the body of the post. See [SSB-Markdown](#) for some Scuttlebot-specific markdown rules.

### Channel

`channel` is optionally used for categorization. It's similar to subreddits or chat channels.

### Root and branch

`root` and `branch` are for replies. `root` should point to the topmost message in the thread. `branch` should point to the message or set of messages in the thread which is being replied to.

In the first reply of a thread, `root === branch`, and both should be included. `root` and `branch` should only point to `type: post` messages. If the post is about another message-type, use `mentions`.

### Mentions

`mentions` is a generic reference to other feeds, entities, or blobs.
- User mentions: for when a user is referenced in the `text`.
- Blob mentions: for when a file is referected in the `text`.
- Message mentions: for when a message is referenced in the `text`.

### Recps

`recps` is a list of user-links specifying who the message is for. This is typically used for encrypted messages, to specify who the message was encrypted for, but it can be used in unencrypted messages as well.

See also:
- [Message Schemas](#)
- [Linking message](#)
- [SSB-Msg-Schemas API](#)

## About

About-messages set attributes about someone or something. They can be used to set a name or picture for users, files, or messages. However, they're most commonly published about users.

About messages do not have to have both `name` and `image`, and, in fact, it's recommended that they only have one or the other. Why? Because then a `vote` on the message can be more specific.

Javascript:

```js
{ type: 'about', about: Link, name: String, image: BlobLink }
```

### Names

There's no global registry of names. Each user can choose a name for themselves and for others, and conflicts are allowed.

Its up to your application to decide which names to use. Generally speaking, names by followed users are trusted, and the local user gets to override all decisions.

A common rule for names is to allow `A-z0-9._-`, and not allow a trailing `.`.

### Images

For images, a common size is 512x512. Square images are recommended but not required.

Some recommended (but not required) attributes on the `image` link:

* `width` in pixels
* `height` in pixels
* `name` a string filename
* `size` in bytes
* `type` a mimetype

See also:

* [Update your profile](#)
* [Message schemas](#)
* [Linking messages](#)
* [SSB-Msg-Schemas API](#)

## Contact

Contact-messages determine who you are following or blocking.

Javascript:

```js
{
  type: 'contact',
  contact: FeedLink,
  following: Bool,
  blocking: Bool
}
```

See also:

* [Message Schemas](#)
* [Linking messages](#)
* [SSB-Msg-Schemas API](#)

## Vote

Vote-messages signal approval about someone or something. Votes can be on users, messages, or blobs.

The `value` should be negative, 0, or positive. If the value is non-numeric, it is invalid.

The `reason` is an optional string to explain the vote.

Javascript:

```js
{
  type: 'vote',
  vote: {
    link: Link,
    value: -1|0|1,
    reason: String?
  }
}
```

### Voting on users

A positive vote on a user signals trust in that user. It's generally used to "confirm" that you think that user publishes good information.

A negative vote on a user is a "flag." You can flag a user for publishing bad information, making false claims, or being abusive. You can also flag a user if the owner lost the keys.

Common values for `reason` in downvotes on users:

* `dead`: Dead Account / Lost Keys
* `spam`: Spammer
* `abuse`: Abusive behavior

### Voting on messages/blobs

A positive vote on a message or blobs signals that you like the content of the message/blobs.

A negative vote on a message is a "flag" that signals that you think the message/blob content should not be used or seen.

See also:

* [Message Schemas](#)
* [Linking messages](#)
* [SSB-Msg-Schemas API](#)

## Pub

Pub-messages announce the ID, address, and port of public Pub users. They are automatically published by Scuttlebot after successfully using an invite to a Pub.

When Scuttlebot sees a pub-message, it will add the link/host/port triple to its peers table, and connect to the peer in the future to sync messages.

Javascript:

```js
{
  type: 'pub',
  pub: {
    link: FeedLink,
    host: String,
    port: Number
  }
}
```

See also:

* [Message Schemas](#)
* [Linking messages](#)
* [SSB-Msg-Schemas API](#)

## Custom Types

Message schemas are interpretted according to the `type` attribute, as demonstrated in the other pages of this section.

There is no restriction on which types applications use. A `type` simply must be a string between 3 and 52 characters long. You are free to create new types, with their own schemas, as you need them.

Likewise, there is no restriction on message schemas, so long as the content is an object, and the total message size (including headers) is less than 8kb.

### Interoperation

Applications should interpret messages "defensively." There's nothing enforcing a schema, so (as with any input) applications must be prepared for malformed `content` objects in messages.

Applications should endeavor to interpret messages the same way. Otherwise, they won't be able to interoperate, and may introduce unexpected behaviors.

There is no official mechanism for making sure message-types interoperate, except for the documentation which you're reading here. As it becomes clear that new types are coming into common use, we'll add them to this site.

### Namespacing

To avoid accidental collisions with other applications, it's a good idea to add your org or application name (or both!) to the message type. We recommend: `{org}-{app}-{type}`, or `{app}-{type}` if you think your app has a very unique name.

See also:

* [Message Schemas](#)
* [Linking messages](#)
* [SSB-Msg-Schemas API](#)

## Configure Scuttlebot

Scuttlebot keeps all data and configuration in the `~/.ssb` directory.

Configuration is stored in `~/.ssb/config`, which is a JSON file:

JSON:

```js
{
  "host": "",
  "port": 8008,
  "timeout": 30000,
  "pub": true,
  "local": true,
  "friends": {
    "dunbar": 150,
    "hops": 3
  },
  "gossip": {
    "connections": 2
  },
  "master": [],
  "logging": {
    "level": "notice"
  }
}
```

You can also specify config flags in the `sbot` server call. For example:

Bash:

```bash
$ sbot server --port 1234 --timeout 500
```

The options:

* `host` (string) The domain or ip address for `sbot`. Defaults to your public ip address.
* `port` (string|number) The port for `sbot`. Defaults to `8008`.
* `timeout`: (number) Number of milliseconds a replication stream can idle before it's automatically disconnected. Defaults to `30000`.
* `pub` (boolean) Replicate with pub servers. Defaults to `true`.
* `local` (boolean) Replicate with local servers found on the same network via `udp`. Defaults to `true`.
* `friends.dunbar` (number) `Dunbar's number`. Number of nodes your instance will replicate. Defaults to `150`.
* `friends.hops` (number) How many friend of friend hops to replicate. Defaults to `3`.
* `gossip.connections` (number) How many other nodes to connect with at one time. Defaults to `2`.
* `master` (array) Pubkeys of users who, if they connect to the Scuttlebot instance, are allowed to command the primary user with full rights. Useful for remotely operating a pub. Defaults to `[]`.
* `logging.level` (string) How verbose should the logging be. Possible values are error, warning, notice, and info. Defaults to `notice`.