## Connection Events
The Redis instance will emit some events about the state of the connection to the Redis server.

Event    | Description
:------------- | :-------------
connect  | emits when a connection is established to the Redis server.
ready    | If `enableReadyCheck` is `true`, client will emit `ready` when the server reports that it is ready to receive commands (e.g. finish loading data from disk).<br>Otherwise, `ready` will be emitted immediately right after the `connect` event.
error    | emits when an error occurs while connecting.<br>However, ioredis emits all `error` events silently (only emits when there's at least one listener) so that your application won't crash if you're not listening to the `error` event.
close    | emits when an established Redis server connection has closed.
reconnecting | emits after `close` when a reconnection will be made. The argument of the event is the time (in ms) before reconnecting.
end     | emits after `close` when no more reconnections will be made.

You can also check out the `Redis#status` property to get the current connection status.

Besides the above connection events, there are several other custom events:

Event    | Description
:------------- | :-------------
authError | emits when the password specified in the options is wrong or the server doesn't require a password.
select   | emits when the database changed. The argument is the new db number.
