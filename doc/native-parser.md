## Native Parser
If [hiredis](https://github.com/redis/hiredis-node) is installed (by `npm install hiredis`),
ioredis will use it by default. Otherwise, a pure JavaScript parser will be used.
Typically, there's not much difference between them in terms of performance.
