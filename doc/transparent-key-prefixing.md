## Transparent Key Prefixing
This feature allows you to specify a string that will automatically be prepended
to all the keys in a command, which makes it easier to manage your key
namespaces.

```javascript
var fooRedis = new Redis({ keyPrefix: 'foo:' });
fooRedis.set('bar', 'baz');  // Actually sends SET foo:bar baz

fooRedis.defineCommand('echo', {
  numberOfKeys: 2,
  lua: 'return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}'
});

// Works well with pipelining/transaction
fooRedis.pipeline()
  // Sends SORT foo:list BY foo:weight_*->fieldname
  .sort('list', 'BY', 'weight_*->fieldname')
  // Supports custom commands
  // Sends EVALSHA xxx foo:k1 foo:k2 a1 a2
  .echo('k1', 'k2', 'a1', 'a2')
  .exec()
```
