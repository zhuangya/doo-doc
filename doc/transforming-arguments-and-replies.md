## Transforming Arguments & Replies
Most Redis commands take one or more Strings as arguments,
and replies are sent back as a single String or an Array of Strings. However, sometimes
you may want something different. For instance, it would be more convenient if the `HGETALL`
command returns a hash (e.g. `{ key: val1, key2: v2 }`) rather than an array of key values (e.g. `[key1, val1, key2, val2]`).

ioredis has a flexible system for transforming arguments and replies. There are two types
of transformers, argument transformer and reply transformer:

```javascript
var Redis = require('ioredis');

// Here's the built-in argument transformer converting
// hmset('key', { k1: 'v1', k2: 'v2' })
// or
// hmset('key', new Map([['k1', 'v1'], ['k2', 'v2']]))
// into
// hmset('key', 'k1', 'v1', 'k2', 'v2')
Redis.Command.setArgumentTransformer('hmset', function (args) {
  if (args.length === 2) {
    if (typeof Map !== 'undefined' && args[1] instanceof Map) {
      // utils is a internal module of ioredis
      return [args[0]].concat(utils.convertMapToArray(args[1]));
    }
    if ( typeof args[1] === 'object' && args[1] !== null) {
      return [args[0]].concat(utils.convertObjectToArray(args[1]));
    }
  }
  return args;
});

// Here's the built-in reply transformer converting the HGETALL reply
// ['k1', 'v1', 'k2', 'v2']
// into
// { k1: 'v1', 'k2': 'v2' }
Redis.Command.setReplyTransformer('hgetall', function (result) {
  if (Array.isArray(result)) {
    var obj = {};
    for (var i = 0; i < result.length; i += 2) {
      obj[result[i]] = result[i + 1];
    }
    return obj;
  }
  return result;
});
```

There are three built-in transformers, two argument transformers for `hmset` & `mset` and
a reply transformer for `hgetall`. Transformers for `hmset` and `hgetall` were mentioned
above, and the transformer for `mset` is similar to the one for `hmset`:

```javascript
redis.mset({ k1: 'v1', k2: 'v2' });
redis.get('k1', function (err, result) {
  // result === 'v1';
});

redis.mset(new Map([['k3', 'v3'], ['k4', 'v4']]));
redis.get('k3', function (err, result) {
  // result === 'v3';
});
```
