---
permalink: /http2-ssl
---

# How to use HTTP2 and SSL with MeiliSearch

For those who would like to use HTTP2 we have to warn you that this is only possible if you have launched your server with SSL.

We will therefore see here how to launch a MeiliSearch server with SSL. In this little tutorial I will do it locally, but be aware that you can very well do the same thing on a server.

To start I need the binary of MeiliSearch, we could also use docker, it would be necessary to pass him the parameters in variable of environment aison that the certificates via a volume.

We will also need a tool to generate a certificate. In this example I use [mkcert](https://github.com/FiloSottile/mkcert). We could also use certbot if we were on a server or directly signed certificates by a certificate authority.

Finally, here I would use curl to make my requests because it's easy to specify that we want to make HTTP2 requests with the `--http2` option.

## Test with HTTP2 without SSL

We start by running the binary.

```
./meilisearch
```

And then test a request.

```
curl -kvs --http2 --request GET 'http://localhost:7700/indexes'
```

The answer from the server is:

```
*   Trying ::1...
* TCP_NODELAY set
* Connection failed
* connect to ::1 port 7700 failed: Connection refused
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 7700 (#0)
> GET /indexes HTTP/1.1
> Host: localhost:7700
> User-Agent: curl/7.64.1
> Accept: */*
> Connection: Upgrade, HTTP2-Settings
> Upgrade: h2c
> HTTP2-Settings: AAMAAABkAARAAAAAAAIAAAAA
>
< HTTP/1.1 200 OK
< content-length: 2
< content-type: application/json
< date: Fri, 17 Jul 2020 11:01:02 GMT
<
* Connection #0 to host localhost left intact
[]* Closing connection 0
```

we can see with the line `> Connection: Upgrade, HTTP2-Settings` that the server tries to upgrade to HTTP2 but fails.
The answer `< HTTP/1.1 200 OK` indicates that we stayed in HTTP1.

## Test with HTTP2 with SSL

This time we start by generating the SSL certificates. This one generates two files `127.0.0.1.pem` and `127.0.0.1-key.pem`.

```
mkcert '127.0.0.1'
```

Then we use the certificate and the key to launch MeiliSearch with SSL.

```
./meilisearch --ssl-cert-path ./127.0.0.1.pem --ssl-key-path ./127.0.0.1-key.pem
```

Next, make the same request as above but change `http://` to `https://`.

```
curl -kvs --http2 --request GET 'https://localhost:7700/indexes'
```

The answer from the server is:

```
*   Trying ::1...
* TCP_NODELAY set
* Connection failed
* connect to ::1 port 7700 failed: Connection refused
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 7700 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/cert.pem
  CApath: none
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: O=mkcert development certificate; OU=quentindequelen@s-iMac (Quentin de Quelen)
*  start date: Jun  1 00:00:00 2019 GMT
*  expire date: Jul 17 10:38:53 2030 GMT
*  issuer: O=mkcert development CA; OU=quentindequelen@s-iMac (Quentin de Quelen); CN=mkcert quentindequelen@s-iMac (Quentin de Quelen)
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7ff601009200)
> GET /indexes HTTP/2
> Host: localhost:7700
> User-Agent: curl/7.64.1
> Accept: */*
>
* Connection state changed (MAX_CONCURRENT_STREAMS == 4294967295)!
< HTTP/2 200
< content-length: 2
< content-type: application/json
< date: Fri, 17 Jul 2020 11:06:27 GMT
<
* Connection #0 to host localhost left intact
[]* Closing connection 0
```

We can see here that the server has successfully switched from HTTP1 to HTTP2.

```
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
```

The server responds well in HTTP2.

```
< HTTP/2 200
```
