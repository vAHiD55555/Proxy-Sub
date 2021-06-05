# Introduction

Clash uses [YAML](https://yaml.org), *YAML Ain't Markup Language*, for configuration files. YAML is designed to be easy to be read, be written, and be interpreted by computers, and is commonly used for exact configuration files. In this chapter, we'll cover the common features of Clash and how they should be used and configured.

Clash works by opening HTTP, SOCKS5, or the transparent proxy server on the local end. When a request, or say packet, comes in, Clash *routes* the packet to different remote servers ("nodes") with either VMess, Shadowsocks, Snell, Trojan, SOCKS5 or HTTP protocol.

# All Configuration Options

```yaml
# Port of HTTP(S) proxy server on the local end
port: 7890

# Port of SOCKS5 proxy server on the local end
socks-port: 7891

# Transparent proxy server port for Linux and macOS (Redirect TCP and TProxy UDP)
# redir-port: 7892

# Transparent proxy server port for Linux (TProxy TCP and TProxy UDP)
# tproxy-port: 7893

# HTTP(S) and SOCKS5 server on the same port
# mixed-port: 7890

# authentication of local SOCKS5/HTTP(S) server
# authentication:
#  - "user1:pass1"
#  - "user2:pass2"

# Set to true to allow connections to the local-end server from
# other LAN IP addresses
allow-lan: false

# This is only applicable when `allow-lan` is `true`
# '*': bind all IP addresses
# 192.168.122.11: bind a single IPv4 address
# "[aaaa::a8aa:ff:fe09:57d8]": bind a single IPv6 address
bind-address: '*'

# Clash router working mode
# rule: rule-based packet routing
# global: all packets will be forwarded to a single endpoint
# direct: directly forward the packets to the Internet
mode: rule

# Clash by default prints logs to STDOUT
# info / warning / error / debug / silent
log-level: info

# When set to false, resolver won't translate hostnames to IPv6 addresses
ipv6: false

# RESTful web API listening address
external-controller: 127.0.0.1:9090

# A relative path to the configuration directory or an absolute path to a
# directory in which you put some static web resource. Clash core will then
# serve it at `http://{{external-controller}}/ui`.
external-ui: folder

# Secret for the RESTful API (optional)
# Authenticate by spedifying HTTP header `Authorization: Bearer ${secret}`
# ALWAYS set a secret if RESTful API is listening on 0.0.0.0
# secret: ""

# Outbound interface name
interface-name: en0

# Static hosts for DNS server and connection establishment (like /etc/hosts)
#
# Wildcard hostnames are supported (e.g. *.clash.dev, *.foo.*.example.com)
# Non-wildcard domain names have a higher priority than wildcard domain names
# e.g. foo.example.com > *.example.com > .example.com
# P.S. +.foo.com equals to .foo.com and foo.com
hosts:
  # '*.clash.dev': 127.0.0.1
  # '.dev': 127.0.0.1
  # 'alpha.clash.dev': '::1'

profile:
  # Store the `select` results in $HOME/.config/clash/.cache
  # set false If you don't want this behavior
  # when two different configurations have groups with the same name, the selected values are shared
  store-selected: false

# DNS server settings
# This section is optional. When not present, the DNS server will be disabled.
dns:
  enable: false
  listen: 0.0.0.0:53
  # ipv6: false # when the false, response to AAAA questions will be empty

  # These nameservers are used to resolve the DNS nameserver hostnames below.
  # Specify IP addresses only
  default-nameserver:
    - 114.114.114.114
    - 8.8.8.8
  enhanced-mode: redir-host # or fake-ip
  fake-ip-range: 198.18.0.1/16 # Fake IP addresses pool CIDR
  # use-hosts: true # lookup hosts and return IP record
  
  # Hostnames in this list will not be resolved with fake IPs
  # i.e. questions to these domain names will always be answered with their
  # real IP addresses
  # fake-ip-filter:
  #   - '*.lan'
  #   - localhost.ptlogin2.qq.com
  
  # Supports UDP, TCP, DoT, DoH. You can specify the port to connect to.
  # All DNS questions are sent directly to the nameserver, without proxies
  # involved. Clash answers the DNS question with the first result gathered.
  nameserver:
    - 114.114.114.114 # default value
    - 8.8.8.8 # default value
    - tls://dns.rubyfish.cn:853 # DNS over TLS
    - https://1.1.1.1/dns-query # DNS over HTTPS

  # When `fallback` is present, the DNS server will send concurrent requests
  # to the servers in this section along with servers in `nameservers`.
  # The answers from fallback servers are used when the GEOIP country
  # is not `CN`.
  # fallback:
  #   - tcp://1.1.1.1

  # If IP addresses resolved with servers in `nameservers` are in the specified
  # subnets below, they are considered invalid and results from `fallback`
  # servers are used instead.
  #
  # IP address resolved with servers in `nameserver` is used when
  # `fallback-filter.geoip` is true and when GEOIP of the IP address is `CN`.
  #
  # If `fallback-filter.geoip` is false, results from `nameserver` nameservers
  # are always used if not match `fallback-filter.ipcidr`.
  #
  # This is a countermeasure against DNS pollution attacks.
  fallback-filter:
    geoip: true
    ipcidr:
      # - 240.0.0.0/4
    # domain:
    #   - '+.google.com'
    #   - '+.facebook.com'
    #   - '+.youtube.com'

proxies:
  # Shadowsocks
  # The supported ciphers (encryption methods):
  #   aes-128-gcm aes-192-gcm aes-256-gcm
  #   aes-128-cfb aes-192-cfb aes-256-cfb
  #   aes-128-ctr aes-192-ctr aes-256-ctr
  #   rc4-md5 chacha20-ietf xchacha20
  #   chacha20-ietf-poly1305 xchacha20-ietf-poly1305
  - name: "ss1"
    type: ss
    server: server
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "password"
    # udp: true

  - name: "ss2"
    type: ss
    server: server
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "password"
    plugin: obfs
    plugin-opts:
      mode: tls # or http
      # host: bing.com

  - name: "ss3"
    type: ss
    server: server
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "password"
    plugin: v2ray-plugin
    plugin-opts:
      mode: websocket # no QUIC now
      # tls: true # wss
      # skip-cert-verify: true
      # host: bing.com
      # path: "/"
      # mux: true
      # headers:
      #   custom: value

  # vmess
  # cipher support auto/aes-128-gcm/chacha20-poly1305/none
  - name: "vmess"
    type: vmess
    server: server
    port: 443
    uuid: uuid
    alterId: 32
    cipher: auto
    # udp: true
    # tls: true
    # skip-cert-verify: true
    # servername: example.com # priority over wss host
    # network: ws
    # ws-path: /path
    # ws-headers:
    #   Host: v2ray.com

  - name: "vmess-h2"
    type: vmess
    server: server
    port: 443
    uuid: uuid
    alterId: 32
    cipher: auto
    network: h2
    tls: true
    h2-opts:
      host:
        - http.example.com
        - http-alt.example.com
      path: /
  
  - name: "vmess-http"
    type: vmess
    server: server
    port: 443
    uuid: uuid
    alterId: 32
    cipher: auto
    # udp: true
    # network: http
    # http-opts:
    #   # method: "GET"
    #   # path:
    #   #   - '/'
    #   #   - '/video'
    #   # headers:
    #   #   Connection:
    #   #     - keep-alive

  - name: vmess-grpc
    server: server
    port: 443
    type: vmess
    uuid: uuid
    alterId: 32
    cipher: auto
    network: grpc
    tls: true
    servername: example.com
    # skip-cert-verify: true
    grpc-opts:
      grpc-service-name: "example"

  # socks5
  - name: "socks"
    type: socks5
    server: server
    port: 443
    # username: username
    # password: password
    # tls: true
    # skip-cert-verify: true
    # udp: true

  # http
  - name: "http"
    type: http
    server: server
    port: 443
    # username: username
    # password: password
    # tls: true # https
    # skip-cert-verify: true
    # sni: custom.com

  # Snell
  # Beware that there's currently no UDP support yet
  - name: "snell"
    type: snell
    server: server
    port: 44046
    psk: yourpsk
    # version: 2
    # obfs-opts:
      # mode: http # or tls
      # host: bing.com

  # Trojan
  - name: "trojan"
    type: trojan
    server: server
    port: 443
    password: yourpsk
    # udp: true
    # sni: example.com # aka server name
    # alpn:
    #   - h2
    #   - http/1.1
    # skip-cert-verify: true

  - name: trojan-grpc
    server: server
    port: 443
    type: trojan
    password: "example"
    network: grpc
    sni: example.com
    # skip-cert-verify: true
    udp: true
    grpc-opts:
      grpc-service-name: "example"

  # ShadowsocksR
  # The supported ciphers (encryption methods): all stream ciphers in ss
  # The supported obfses:
  #   plain http_simple http_post
  #   random_head tls1.2_ticket_auth tls1.2_ticket_fastauth
  # The supported supported protocols:
  #   origin auth_sha1_v4 auth_aes128_md5
  #   auth_aes128_sha1 auth_chain_a auth_chain_b  
  - name: "ssr"
    type: ssr
    server: server
    port: 443
    cipher: chacha20-ietf
    password: "password"
    obfs: tls1.2_ticket_auth
    protocol: auth_sha1_v4
    # obfs-param: domain.tld
    # protocol-param: "#"
    # udp: true

proxy-groups:
  # relay chains the proxies. proxies shall not contain a relay. No UDP support.
  # Traffic: clash <-> http <-> vmess <-> ss1 <-> ss2 <-> Internet
  - name: "relay"
    type: relay
    proxies:
      - http
      - vmess
      - ss1
      - ss2

  # url-test select which proxy will be used by benchmarking speed to a URL.
  - name: "auto"
    type: url-test
    proxies:
      - ss1
      - ss2
      - vmess1
    # tolerance: 150
    # lazy: true
    url: 'http://www.gstatic.com/generate_204'
    interval: 300

  # fallback selects an available policy by priority. The availability is tested by accessing an URL, just like an auto url-test group.
  - name: "fallback-auto"
    type: fallback
    proxies:
      - ss1
      - ss2
      - vmess1
    url: 'http://www.gstatic.com/generate_204'
    interval: 300

  # load-balance: The request of the same eTLD+1 will be dial to the same proxy.
  - name: "load-balance"
    type: load-balance
    proxies:
      - ss1
      - ss2
      - vmess1
    url: 'http://www.gstatic.com/generate_204'
    interval: 300
    # strategy: consistent-hashing # or round-robin

  # select is used for selecting proxy or proxy group
  # you can use RESTful API to switch proxy is recommended for use in GUI.
  - name: Proxy
    type: select
    # disable-udp: true
    proxies:
      - ss1
      - ss2
      - vmess1
      - auto
  
  - name: UseProvider
    type: select
    use:
      - provider1
    proxies:
      - Proxy
      - DIRECT

proxy-providers:
  provider1:
    type: http
    url: "url"
    interval: 3600
    path: ./provider1.yaml
    health-check:
      enable: true
      interval: 600
      # lazy: true
      url: http://www.gstatic.com/generate_204
  test:
    type: file
    path: /test.yaml
    health-check:
      enable: true
      interval: 36000
      url: http://www.gstatic.com/generate_204

rules:
  - DOMAIN-SUFFIX,google.com,auto
  - DOMAIN-KEYWORD,google,auto
  - DOMAIN,google.com,auto
  - DOMAIN-SUFFIX,ad.com,REJECT
  - SRC-IP-CIDR,192.168.1.201/32,DIRECT
  # optional param "no-resolve" for IP rules (GEOIP, IP-CIDR, IP-CIDR6)
  - IP-CIDR,127.0.0.0/8,DIRECT
  - GEOIP,CN,DIRECT
  - DST-PORT,80,DIRECT
  - SRC-PORT,7777,DIRECT
  - RULE-SET,apple,REJECT # Premium only
  - MATCH,auto
```

# Specifying Configuration Directory
If not otherwise specified, Clash by default reads the configuration file at `$HOME/.config/clash/config.yaml`.
If it doesn't exist, Clash will generate the default settings.  

You can use command-line option `-d` to specify a configuration directory:

```
$ clash -d . # current directory
$ clash -d /etc/clash
```

You can use command-line option `-f` to specify a configuration:

```
$ clash -f ./config.yaml # current directory
$ clash -f /etc/clash/config.yaml
```

# Syntax
* IPv6 addresses should be wrapped with `[` and `]`. For example: `[aaaa::a8aa:ff:fe09:57d8]`.
* Wildcard characters. Beware any domain with these characters should be wrapped with single-quotes `'`.
  * `*`: single-level wildcard character. `*.google.com` matches `www.google.com` but not `foo.bar.google.com`. It is possible to use `*.*.*.google.com`.
  * `+`: multi-level wildcard character. `+.google.com` matches `google.com`, `www.google.com` and `foo.bar.google.com`. This works exactly like `DOMAIN-SUFFIX`.

# DNS
The DNS server shipped with Clash aims to minimize DNS pollution attack impact and improve network performance. There are two modes for it to work: `redir-host` and `fake-ip`. The biggest difference between the two is how IP addresses are resolved and how the connections are established.

## redir-host
This is more of a traditional way of how proxies work. In this mode, depending on the settings in `dns.nameserver`, `dns.fallback` and `dns.fallback-filter`, the destination FQDN are resolved in several different ways. The first result received by Clash DNS module will be sent back to the client. The client can then establish a connection to the said IP address through Clash.

## fake-ip
The concept of "fake IP" addresses is originated from [RFC 3089](https://tools.ietf.org/rfc/rfc3089):

> A "fake IP" address is used as a key to look up the corresponding "FQDN" information.

When a DNS request is sent to the DNS server, Clash allocates a free *fake IP address* in the fake IP address pool, a mapping table that manages mappings between the FQDN and "fake IP" address. Note that the IP addresses in the fake IP address pool should never be used in real communications. The default CIDR for the pool is a reserved IPv4 address space `198.18.0.1/16`, which can be changed in `dns.fake-ip-range`.

Clash will then lookup the FQDN and check the GEOIP for the IP address, this is merely for the rules (like GEOIP). When a request to the said, "fake IP" address is sent to Clash, Clash establishes a connection to the FQDN linked with the "fake IP" through a SOCKS5, Shadowsocks (or other protocols) server.

# Proxy Groups
Proxy Groups are groups of proxies that you can utilize some special features of Clash to manage and make use of.

* `relay`: The request sent to this proxy group will be relayed through the specified proxy servers sequently. There's currently no UDP support on this. The specified proxy servers should not contain another relay.
* `url-test`: Clash benchmarks each proxy servers in the list, by sending HTTP HEAD requests to a specified URL through these servers periodically. It's possible to set a maximum tolerance value, benchmarking interval, and the target URL.
* `fallback`: Clash periodically tests the availability of servers in the list with the same mechanism of `url-test`. The first available server will be used.
* `load-balance`: The request to the same eTLD+1 will be dialed with the same proxy.
* `select`: The first server is by default used when Clash starts up. Users can choose the server to use with the RESTful API. In this mode, you can hardcode servers in the config or use [Proxy Providers](https://github.com/Dreamacro/clash/wiki/Configuration#proxy-providers).

# Proxy Providers
Proxy Providers give users the power to load proxy server lists dynamically, instead of hardcoding them in the configuration file. There are currently two sources for a proxy provider to load server list from:

* `http`: Clash loads the server list from a specified URL on startup. Clash periodically pulls the server list from remote if the `interval` option is set.
* `file`: Clash loads the server list from a specified location on the filesystem on startup.

Health check is available for both modes, and works exactly like `fallback` in Proxy Groups. The configuration format for the server list files is also exactly the same in the main configuration file:

```yaml
proxies:
  - name: "ss1"
    type: ss
    server: server
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "password"

  - name: "ss2"
    type: ss
    server: server
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "password"
    plugin: obfs
    plugin-opts:
      mode: tls
  
  # ……
```

# Rules
Available keywords:

* `DOMAIN`: `DOMAIN,www.google.com,policy` routes only `www.google.com` to `policy`.
* `DOMAIN-SUFFIX`: `DOMAIN-SUFFIX,youtube.com,policy` routes any FQDN that ends with `youtube.com`, for example, `www.youtube.com` or `foo.bar.youtube.com`, to `policy`. This works like the wildcard character `+`.
* `DOMAIN-KEYWORD`: `DOMAIN-KEYWORD,google,policy` routes any FQDN that contains `google`, for example, `www.google.com` or `googleapis.com`, to `policy`.
* `GEOIP`: `GEOIP,CN,policy` routes any requests to a China IP address to `policy`.
* `IP-CIDR`: `IP-CIDR,127.0.0.0/8,DIRECT` routes any packets to `127.0.0.0/8` to the `DIRECT` policy.
* `IP-CIDR6`: `IP-CIDR6,2620:0:2d0:200::7/32,policy` routes any packets to `2620:0:2d0:200::7/32` to `policy`.
* `SRC-IP-CIDR`: `SRC-IP-CIDR,192.168.1.201/32,DIRECT` routes any packets **from** `192.168.1.201/32` to the `DIRECT` policy.
* `SRC-PORT`: `SRC-PORT,80,policy` routes any packets **from** the port 80 to `policy`.
* `DST-PORT`: `DST-PORT,80,policy` routes any packets **to** the port 80 to `policy`.
* `PROCESS-NAME`: `PROCESS-NAME,nc,DIRECT` routes the process `nc` to `DIRECT`. (support macOS、Linux、FreeBSD and Windows)
* `MATCH`: `MATCH,policy` routes the rest of the packets to `policy`. This rule is **required**.

There are two additional special policies:

* `DIRECT`: directly connects to the target without any proxies involved
* `REJECT`: a black hole for packets. Clash will not process any I/O to this policy.

A policy can be either `DIRECT`, `REJECT`, a proxy group or a proxy server.

## no-resolve
`no-resolve` is an additional option for `GEOIP`, `IP-CIDR`, or `IP-CIDR6` rules. Append `,no-resolve` to these rules to enable. Clash by default translates the domain names to IP addresses when encountering IP rules. Clash skips the IP rules with this option enabled when encountering packets that have an FQDN target.
