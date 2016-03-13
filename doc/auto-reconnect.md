## Auto-reconnect
By default, ioredis will try to reconnect when the connection to Redis is lost
except when the connection is closed manually by `redis.disconnect()` or `redis.quit()`.

It's very flexible to control how long to wait to reconnect after disconnection
using the `retryStrategy` option:

```javascript
var redis = new Redis({
  // This is the default value of `retryStrategy`
  retryStrategy: function (times) {
    var delay = Math.min(times * 2, 2000);
    return delay;
  }
});
```

`retryStrategy` is a function that will be called when the connection is lost.
The argument `times` means this is the nth reconnection being made and
the return value represents how long (in ms) to wait to reconnect. When the
return value isn't a number, ioredis will stop trying to reconnect, and the connection
will be lost forever if the user doesn't call `redis.connect()` manually.

When reconnected, the client will auto subscribe to channels that the previous connection subscribed to.
This behavior can be disabled by setting the `autoResubscribe` option to `false`.

And if the previous connection has some unfulfilled commands (most likely blocking commands such as `brpop` and `blpop`),
the client will resend them when reconnected. This behavior can be disabled by setting the `autoResendUnfulfilledCommands` option to `false`.

### Reconnect on error

Besides auto-reconnect when the connection is closed, ioredis supports reconnecting on the specified errors by the `reconnectOnError` option. Here's an example that will reconnect when receiving `READONLY` error:

```javascript
var redis = new Redis({
  reconnectOnError: function (err) {
    var targetError = 'READONLY';
    if (err.message.slice(0, targetError.length) === targetError) {
      // Only reconnect when the error starts with "READONLY"
      return true; // or `return 1;`
    }
  }
});
```

This feature is useful when using Amazon ElastiCache. Once failover happens, Amazon ElastiCache will switch the master we currently connected with to a slave, leading to the following writes fails with the error `READONLY`. Using `reconnectOnError`, we can force the connection to reconnect on this error in order to connect to the new master.

Furthermore, if the `reconnectOnError` returns `2`, ioredis will resend the failed command after reconnecting.
