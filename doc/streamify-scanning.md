## Streamify Scanning
Redis 2.8 added the `SCAN` command to incrementally iterate through the keys in the database. It's different from `KEYS` in that
`SCAN` only returns a small number of elements each call, so it can be used in production without the downside
of blocking the server for a long time. However, it requires recording the cursor on the client side each time
the `SCAN` command is called in order to iterate through all the keys correctly. Since it's a relatively common use case, ioredis
provides a streaming interface for the `SCAN` command to make things much easier. A readable stream can be created by calling `scanStream`:

```javascript
var redis = new Redis();
// Create a readable stream (object mode)
var stream = redis.scanStream();
var keys = [];
stream.on('data', function (resultKeys) {
  // `resultKeys` is an array of strings representing key names
  for (var i = 0; i < resultKeys.length; i++) {
    keys.push(resultKeys[i]);
  }
});
stream.on('end', function () {
  console.log('done with the keys: ', keys);
});
```

`scanStream` accepts an option, with which you can specify the `MATCH` pattern and the `COUNT` argument:

```javascript
var stream = redis.scanStream({
  // only returns keys following the pattern of `user:*`
  match: 'user:*',
  // returns approximately 100 elements per call
  count: 100
});
```

Just like other commands, `scanStream` has a binary version `scanBufferStream`, which returns an array of buffers. It's useful when
the key names are not utf8 strings.

There are also `hscanStream`, `zscanStream` and `sscanStream` to iterate through elements in a hash, zset and set. The interface of each is
similar to `scanStream` except the first argument is the key name:

```javascript
var stream = redis.hscanStream('myhash', {
  match: 'age:??'
});
```

You can learn more from the [Redis documentation](http://redis.io/commands/scan).
