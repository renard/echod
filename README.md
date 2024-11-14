# echod

Simple multiprotocol server that can be used to test proxies. It can
respond over UDP, TCP and HTTP.

## Usage

By default `echod` listens to:

* `udp@0.0.0.0:5555`
* `tcp@0.0.0.0:5555`
* `http@0.0.0.0:5556`

Those binds can be changed with the `-L` option. The listen format is:

```
PROTO@IP[%IFACE]:PORT
```

where:

* `PROTO` is the protocol: `udp`, `tcp` or `http`
* `IP` is the IP to listen to (either IPv4 or IPv6)
* `IFACE` is the optional interface to bind to. Default is to bind on
  all interfaces.
* `PORT` is the port to listen to.


On the client side use these command to connect to the server:

```
curl 198.19.0.154:5556
echo 'C' | nc -w1 -t 198.19.0.154 5555
echo 'C' | nc -w1 -u 198.19.0.154 5555
```

it will respond:


```
HTTP: echod.example.com 198.18.0.1:23712->0.0.0.0:5556
TCP: echod.example.com 198.18.0.1:27246->0.0.0.0:5555
UDP: echod.example.com 198.18.0.1:26249->0.0.0.0:5555
```

If a specific source IP address should be specified use:

```
curl --interface 198.18.0.1 198.19.0.154:5556
echo 'C' | nc -w1 -s 198.18.0.1 -t 198.19.0.154 5555
echo 'C' | nc -w1 -s 198.18.0.1 -u 198.19.0.154 5555
```

In HTTP `?f=1` van be added to the request to get a full debug response:

```
curl  198.19.0.154:5556/?f=1
GET /?f=1 HTTP/1.1
host: 198.19.0.154:5556
user-agent: curl/7.64.0
accept: */*

HTTP/1.1 200 OK
Content-type: text/plain
Connection: Close

HTTP: echod.example.com 198.18.0.1:57248->0.0.0.0:5556
```

## Usage with HAProxy

Basic HAProxy configuration:

```
frontend http-fe
    bind 198.19.0.154:5556
    mode http
    log-format "${HAPROXY_HTTP_LOG_FMT} %fi:%fp %bi:%bp %si:%sp"
    log global
    use_backend http-be

backend http-be
    mode http
    balance source
    default-server check
    server srv1 198.18.0.102:5556
    server srv2 198.18.0.103:5556

frontend tcp-fe
    bind 198.19.0.154:5555
    mode tcp
    log-format "${HAPROXY_TCP_LOG_FMT} %fi:%fp %bi:%bp %si:%sp"
    log global
    use_backend tcp-be

backend tcp-be
    mode tcp
    balance source
    default-server check
    server srv1 198.18.0.102:5555
    server srv2 198.18.0.103:5555
```

With the Enterprise Edition the UDP module can be used:


```
global
    module-path /opt/hapee-2.9/modules
    module-load hapee-lb-udp.so

udp-lb udp
    dgram-bind 198.19.0.154:5555
    balance source
    #proxy-requests 1
    #proxy-responses 1
    server srv1 198.18.0.102:5555 check
    server srv2 198.18.0.103:5555 check
```

## Additional topics

This has been written in Python without any external requirements. It
should be able to run on most systems without being to intrusive as a
stand alone script.

