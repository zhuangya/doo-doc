## Handle Binary Data
Arguments can be buffers:
```javascript
redis.set('foo', new Buffer('bar'));
```

And every command has a method that returns a Buffer (by adding a suffix of "Buffer" to the command name).
To get a buffer instead of a utf8 string:

```javascript
redis.getBuffer('foo', function (err, result) {
  // result is a buffer.
});
```
