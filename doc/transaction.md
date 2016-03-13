## Transaction
Most of the time, the transaction commands `multi` & `exec` are used together with pipeline.
Therefore, when `multi` is called, a `Pipeline` instance is created automatically by default,
so you can use `multi` just like `pipeline`:

```javascript
redis.multi().set('foo', 'bar').get('foo').exec(function (err, results) {
  // results === [[null, 'OK'], [null, 'bar']]
});
```
If there's a syntax error in the transaction's command chain (e.g. wrong number of arguments, wrong command name, etc),
then none of the commands would be executed, and an error is returned:

```javascript
redis.multi().set('foo').set('foo', 'new value').exec(function (err, results) {
  // err:
  //  { [ReplyError: EXECABORT Transaction discarded because of previous errors.]
  //    name: 'ReplyError',
  //    message: 'EXECABORT Transaction discarded because of previous errors.',
  //    command: { name: 'exec', args: [] },
  //    previousErrors:
  //     [ { [ReplyError: ERR wrong number of arguments for 'set' command]
  //         name: 'ReplyError',
  //         message: 'ERR wrong number of arguments for \'set\' command',
  //         command: [Object] } ] }
});
```

In terms of the interface, `multi` differs from `pipeline` in that when specifying a callback
to each chained command, the queueing state is passed to the callback instead of the result of the command:

```javascript
redis.multi().set('foo', 'bar', function (err, result) {
  // result === 'QUEUED'
}).exec(/* ... */);
```

If you want to use transaction without pipeline, pass `{ pipeline: false }` to `multi`,
and every command will be sent to Redis immediately without waiting for an `exec` invocation:

```javascript
redis.multi({ pipeline: false });
redis.set('foo', 'bar');
redis.get('foo');
redis.exec(function (err, result) {
  // result === [[null, 'OK'], [null, 'bar']]
});
```

The constructor of `multi` also accepts a batch of commands:

```javascript
redis.multi([
  ['set', 'foo', 'bar'],
  ['get', 'foo']
]).exec(function () { /* ... */ });
```

Inline transactions are supported by pipeline, which means you can group a subset of commands
in the pipeline into a transaction:

```javascript
redis.pipeline().get('foo').multi().set('foo', 'bar').get('foo').exec().get('foo').exec();
```
