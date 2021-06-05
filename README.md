# nginx-quiche
Running
-------

The following configuration file can be used as a starting point to enable
HTTP/3 support:

```
events {
    worker_connections  1024;
}

http {
    server {
        # Enable QUIC and HTTP/3.
        listen 443 quic reuseport;

        # Enable HTTP/2 (optional).
        listen 443 ssl http2;

        ssl_certificate      cert.crt;
        ssl_certificate_key  cert.key;

        # Enable all TLS versions (TLSv1.3 is required for QUIC).
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

        # Add Alt-Svc header to negotiate HTTP/3.
        add_header alt-svc 'h3=":443"; ma=86400';
    }
}
```

List of configuration directives
--------------------------------

### http3_max_concurrent_streams

**syntax:** **http3_max_concurrent_streams** *number*;

**default:** *http3_max_concurrent_streams 128;*

**context:** *http*, *server*

Limits the maximum number of concurrent HTTP/3 streams in a connection.

### http3_max_requests

**syntax:** **http3_max_requests** *number*;

**default:** *http3_max_requests 1000;*

**context:** *http*, *server*

Limits the maximum number of requests that can be served on a single HTTP/3
connection, after which the next client request will lead to connection closing
and the need of establishing a new connection.

### http3_max_header_size

**syntax:** **http3_max_header_size** *size*;

**default:** *http3_max_header_size 16k;*

**context:** *http*, *server*

Limits the maximum size of the entire request header list after QPACK decompression.

### http3_initial_max_data

**syntax:** **http3_initial_max_data** *size*;

**default:** *http3_initial_max_data 10m;*

**context:** *http*, *server*

Sets the per-connection incoming flow control limit.

### http3_initial_max_stream_data

**syntax:** **http3_initial_max_stream_data** *size*;

**default:** *http3_initial_max_stream_data 1m;*

**context:** *http*, *server*

Sets the per-stream incoming flow control limit.

### http3_idle_timeout

**syntax:** **http3_idle_timeout** *time*;

**default:** *http3_idle_timeout 3m;*

**context:** *http*, *server*

Sets the timeout of inactivity after which the connection is closed.

List of variables
-----------------

### $http3

"h3" if HTTP/3 was negotiated, or an empty string otherwise.

0-RTT
-----

To support [0-RTT QUIC connection resumption](https://blog.cloudflare.com/even-faster-connection-establishment-with-quic-0-rtt-resumption/)
from the client, you will need the following configuration:

```
http {
    server {
        ...
        ssl_early_data on;
        ssl_session_ticket_key <file>;
        ...
    }
}
```

Please see
[ssl_session_ticket_key](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_ticket_key)
on how to generate the secret file used for TLS session tickets. This is
required when using multiple worker processes.
