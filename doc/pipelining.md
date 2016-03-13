## Pipelining
If you want to send a batch of commands (e.g. > 5), you can use pipelining to queue
the commands in memory and then send them to Redis all at once. This way the performance improves by 50%~300% (See [benchmark section](#benchmark)).

`redis.pipeline()` creates a `Pipeline` instance. You can call any Redis
commands on it just like the `Redis` instance. The commands are queued in memory
and flushed to Redis by calling the `exec` method:

```javascript
var pipeline = redis.pipeline();
pipeline.set('foo', 'bar');
pipeline.del('cc');
pipeline.exec(function (err, results) {
  // `err` is always null, and `results` is an array of responses
  // corresponding to the sequence of queued commands.
  // Each response follows the format `[err, result]`.
});

// You can even chain the commands:
redis.pipeline().set('foo', 'bar').del('cc').exec(function (err, results) {
});

// `exec` also returns a Promise:
var promise = redis.pipeline().set('foo', 'bar').get('foo').exec();
promise.then(function (result) {
  // result === [[null, 'OK'], [null, 'bar']]
});
```

Each chained command can also have a callback, which will be invoked when the command
gets a reply:

```javascript
redis.pipeline().set('foo', 'bar').get('foo', function (err, result) {
  // result === 'bar'
}).exec(function (err, result) {
  // result[1][1] === 'bar'
});
```

In addition to adding commands to the `pipeline` queue individually, you can also pass an array of commands and arguments to the constructor:

```javascript
redis.pipeline([
  ['set', 'foo', 'bar'],
  ['get', 'foo']
]).exec(function () { /* ... */ });
```
