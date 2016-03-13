## Offline Queue
When a command can't be processed by Redis (being sent before the `ready` event), by default, it's added to the offline queue and will be
executed when it can be processed. You can disable this feature by setting the `enableOfflineQueue`
option to `false`:

```javascript
var redis = new Redis({ enableOfflineQueue: false });
```
