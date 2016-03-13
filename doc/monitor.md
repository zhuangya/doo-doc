## Monitor
Redis supports the MONITOR command,
which lets you see all commands received by the Redis server across all client connections,
including from other client libraries and other computers.

The `monitor` method returns a monitor instance.
After you send the MONITOR command, no other commands are valid on that connection. ioredis will emit a monitor event for every new monitor message that comes across.
The callback for the monitor event takes a timestamp from the Redis server and an array of command arguments.

Here is a simple example:

```javascript
redis.monitor(function (err, monitor) {
  monitor.on('monitor', function (time, args) {
  });
});
```
