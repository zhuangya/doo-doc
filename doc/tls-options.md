## TLS Options
Redis doesn't support TLS natively, however if the redis server you want to connect to is hosted behind a TLS proxy (e.g. [stunnel](https://www.stunnel.org/)) or is offered by a PaaS service that supports TLS connection (e.g. [Redis Labs](https://redislabs.com/)), you can set the `tls` option:

```javascript
var redis = new Redis({
  host: 'localhost',
  tls: {
    // Refer to `tls.connect()` section in
    // https://nodejs.org/api/tls.html
    // for all supported options
    ca: fs.readFileSync('cert.pem')
  }
});
```

<hr>
