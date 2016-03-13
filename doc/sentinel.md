## Sentinel
ioredis supports Sentinel out of the box. It works transparently as all features that work when
you connect to a single node also work when you connect to a sentinel group. Make sure to run Redis >= 2.8.12 if you want to use this feature.

To connect using Sentinel, use:

```javascript
var redis = new Redis({
  sentinels: [{ host: 'localhost', port: 26379 }, { host: 'localhost', port: 26380 }],
  name: 'mymaster'
});

redis.set('foo', 'bar');
```

The arguments passed to the constructor are different from the ones you use to connect to a single node, where:

* `name` identifies a group of Redis instances composed of a master and one or more slaves (`mymaster` in the example);
* `sentinels` are a list of sentinels to connect to. The list does not need to enumerate all your sentinel instances, but a few so that if one is down the client will try the next one.

ioredis **guarantees** that the node you connected to is always a master even after a failover. When a failover happens, instead of trying to reconnect to the failed node (which will be demoted to slave when it's available again), ioredis will ask sentinels for the new master node and connect to it. All commands sent during the failover are queued and will be executed when the new connection is established so that none of the commands will be lost.

It's possible to connect to a slave instead of a master by specifying the option `role` with the value of `slave`, and ioredis will try to connect to a random slave of the specified master, with the guarantee that the connected node is always a slave. If the current node is promoted to master due to a failover, ioredis will disconnect from it and ask the sentinels for another slave node to connect to.

Besides the `retryStrategy` option, there's also a `sentinelRetryStrategy` in Sentinel mode which will be invoked when all the sentinel nodes are unreachable during connecting. If `sentinelRetryStrategy` returns a valid delay time, ioredis will try to reconnect from scratch. The default value of `sentinelRetryStrategy` is:

```javascript
function (times) {
  var delay = Math.min(times * 10, 1000);
  return delay;
}
```
