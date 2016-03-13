## Pub/Sub

Here is a simple example of the API for publish/subscribe.
The following program opens two client connections.
It subscribes to a channel with one connection
and publishes to that channel with the other:

```javascript
var Redis = require('ioredis');
var redis = new Redis();
var pub = new Redis();
redis.subscribe('news', 'music', function (err, count) {
  // Now we are subscribed to both the 'news' and 'music' channels.
  // `count` represents the number of channels we are currently subscribed to.

  pub.publish('news', 'Hello world!');
  pub.publish('music', 'Hello again!');
});

redis.on('message', function (channel, message) {
  // Receive message Hello world! from channel news
  // Receive message Hello again! from channel music
  console.log('Receive message %s from channel %s', message, channel);
});

// There's also an event called 'messageBuffer', which is the same as 'message' except
// it returns buffers instead of strings.
redis.on('messageBuffer', function (channel, message) {
  // Both `channel` and `message` are buffers.
});
```

`PSUBSCRIBE` is also supported in a similar way:

```javascript
redis.psubscribe('pat?ern', function (err, count) {});
redis.on('pmessage', function (pattern, channel, message) {});
redis.on('pmessageBuffer', function (pattern, channel, message) {});
```

When a client issues a SUBSCRIBE or PSUBSCRIBE, that connection is put into a "subscriber" mode.
At that point, only commands that modify the subscription set are valid.
When the subscription set is empty, the connection is put back into regular mode.

If you need to send regular commands to Redis while in subscriber mode, just open another connection.
