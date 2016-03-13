# Error Handling
All the errors returned by the Redis server are instances of `ReplyError`, which can be accessed via `Redis`:

```javascript
var Redis = require('ioredis');
var redis = new Redis();
// This command causes a reply error since the SET command requires two arguments.
redis.set('foo', function (err) {
  err instanceof Redis.ReplyError
});
```

When a reply error is not handled (no callback is specified, and no `catch` method is chained),
the error will be logged to stderr. For instance:

```javascript
var Redis = require('ioredis');
var redis = new Redis();
redis.set('foo');
```

The following error will be printed:

```
Unhandled rejection ReplyError: ERR wrong number of arguments for 'set' command
    at ReplyParser._parseResult (/app/node_modules/ioredis/lib/parsers/javascript.js:60:14)
    at ReplyParser.execute (/app/node_modules/ioredis/lib/parsers/javascript.js:178:20)
    at Socket.<anonymous> (/app/node_modules/ioredis/lib/redis/event_handler.js:99:22)
    at Socket.emit (events.js:97:17)
    at readableAddChunk (_stream_readable.js:143:16)
    at Socket.Readable.push (_stream_readable.js:106:10)
    at TCP.onread (net.js:509:20)
```

But the error stack doesn't make any sense because the whole stack happens in the ioredis
module itself, not in your code. So it's not easy to find out where the error happens in your code.
ioredis provides an option `showFriendlyErrorStack` to solve the problem. When you enable
`showFriendlyErrorStack`, ioredis will optimize the error stack for you:

```javascript
var Redis = require('ioredis');
var redis = new Redis({ showFriendlyErrorStack: true });
redis.set('foo');
```

And the output will be:

```
Unhandled rejection ReplyError: ERR wrong number of arguments for 'set' command
    at Object.<anonymous> (/app/index.js:3:7)
    at Module._compile (module.js:446:26)
    at Object.Module._extensions..js (module.js:464:10)
    at Module.load (module.js:341:32)
    at Function.Module._load (module.js:296:12)
    at Function.Module.runMain (module.js:487:10)
    at startup (node.js:111:16)
    at node.js:799:3
```

This time the stack tells you that the error happens on the third line in your code. Pretty sweet!
However, it would decrease the performance significantly to optimize the error stack. So by
default, this option is disabled and can only be used for debugging purposes. You **shouldn't** use this feature in a production environment.

If you want to catch all unhandled errors without decreased performance, there's another way:

```javascript
var Redis = require('ioredis');
Redis.Promise.onPossiblyUnhandledRejection(function (error) {
  // you can log the error here.
  // error.command.name is the command name, here is 'set'
  // error.command.args is the command arguments, here is ['foo']
});
var redis = new Redis();
redis.set('foo');
```
