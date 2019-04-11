---
title: "Improving DNS privacy"
excerpt: "Clear text DNS queries can be observed and those queries can reveal not only what websites an individual visits but also meta data about other services"
permalink: /secure-dns/
header:
  overlay_image: /assets/images/spying.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: The most significant leaks of data on the Internet
categories:
  - Sysadmin
tags:
  - raspberrypi
  - wifi
  - sysadmin
  - dns
  - privacy
toc: true
---

# Problem statement

What is a _DNS_?

> The Domain Name System (DNS) is a hierarchical and decentralized naming system for computers, services, or other resources connected to the Internet or a private network. It associates various information with domain names assigned to each of the participating entities.

The problem with DNS is primarily **privacy**.

From the [dnsprivacy.org](dnsprivacy.org) site I've extracted some from the many issues:

- Whilst the data in the DNS is public, **individual transactions** made by an end user should not be public.
- However DNS queries are sent in **clear text** (using UDP or TCP) which means passive eavesdroppers can **observe all the DNS lookups** performed.
- **Some ISPs log DNS queries** at the resolver and share this information with third-parties in ways not known or obvious to end users.

{% include video id="JnxE5RPnyiE" provider="youtube" %}

## Is there any solution

There are multiple solutions:

- DNS-over-TLS (DoT)
- DNS-over-HTTP (DoH)
- DNS-over-DTLS
- DNSCrypt
- DNS-over-HTTPS (proxied)
- DNS-over-QUIC
- DNSCurve

In this post I'm going to describe how to implement DNS-over-TLS (DoT) that can be shared across all your private LAN, for further information on how the various solutions compare [see here](https://dnsprivacy.org/wiki/display/DP/DNS+Privacy+-+The+Solutions).

## DNS-over-TLS (DoT)

The use of Transport Layer Security (TLS) provides privacy for DNS.

Encryption provided by TLS eliminates opportunities for eavesdropping and on-path tampering with DNS queries in the network.

## DoT implementation: Stubby

[Stubby](https://dnsprivacy.org/wiki/display/DP/About+Stubby) is an application that acts as a local DNS Privacy stub resolver (using DNS-over-TLS). Stubby encrypts DNS queries sent from a client machine (desktop or laptop) to a DNS Privacy resolver increasing end user privacy.

### Raspbian installation

Install build dependencies:

```bash
sudo apt install -y git automake build-essential libssl-dev libtool m4 autoconf libyaml-dev
```

Clone and configure getdns project:

```bash
git clone https://github.com/getdnsapi/getdns.git
cd getdns
git submodule update --init
libtoolize -ci
autoreconf -fi
mkdir -v build && cd build
../configure --prefix=/usr/local --without-libidn --without-libidn2 --enable-stub-only --with-stubby
```

Build and install:

```bash
make
sudo make install
```

Install runtime dependencies:

```bash
sudo apt install -y libev4 libevent-core-2.0.5 libuv1 libidn11 dns-root-data libunbound2
cd ~
wget http://pyyaml.org/download/libyaml/yaml-0.2.2.tar.gz
tar xvf yaml-0.2.2.tar.gz
cd yaml-0.2.2
./configure
make
sudo make install
```

Edit stubby configuration:

```bash
vi /usr/local/etc/stubby/stubby.yml
````

My configuration uses cloudflare dns as first choice and quad9 as fallback:

```yaml
resolution_type: GETDNS_RESOLUTION_STUB
# Enable DoT
dns_transport_list:
  - GETDNS_TRANSPORT_TLS
# Strict mode TLS only
tls_authentication: GETDNS_AUTHENTICATION_REQUIRED
tls_query_padding_blocksize: 128
edns_client_subnet_private : 1
# Set to 0 to treat the upstreams below as an ordered list and use a single
# upstream until it becomes unavailable, then use the next one.
round_robin_upstreams: 0
idle_timeout: 10000
# Set the listen addresses for the stubby DAEMON. IPv4 only.
listen_addresses:
  - 127.0.0.1@5300
# Require DNSSEC validation.
dnssec: GETDNS_EXTENSION_TRUE
appdata_dir: "/var/cache/stubby"
upstream_recursive_servers:
## Cloudflare 1.1.1.1 and 1.0.0.1
  - address_data: 1.1.1.1
    tls_auth_name: "cloudflare-dns.com"
  - address_data: 1.0.0.1
    tls_auth_name: "cloudflare-dns.com"
## Quad 9 'secure' service - Filters, does DNSSEC, doesn't send ECS
  - address_data: 9.9.9.9
    tls_auth_name: "dns.quad9.net"
```

Install it:

```bash
sudo /usr/bin/install -Dm644 /usr/local/etc/stubby/stubby.yml /etc/stubby.yml
```

Create a _systemd_ service:

```bash
sudo vi /lib/systemd/system/stubby.service
```

With content:

```text
[Unit]
Description=Stubby DNS resolver
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/usr/local/bin/stubby -C /etc/stubby.yml
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

Enable and start it

```bash
sudo systemctl daemon-reload
sudo systemctl enable stubby
sudo systemctl start stubby
```

### Pi-hole

Combining Stubby with [Pi-hole](https://pi-hole.net/) we can obtain a perfect match of privacy and performance.

Installing it is quick and easy, just follow the [documentation site](https://docs.pi-hole.net/main/basic-install/).

<figure>
	<img src="/assets/images/pihole-stubby.png">
	<figcaption>Configure Stubby as the only upstream DNS server.</figcaption>
</figure>

## Benchmark

Using [dnsperftest](https://github.com/cleanbrowsing/dnsperftest/) script results are pretty good:

```bash
bash ./dnstest.sh | sort -k 22 -n
                  test1   test2   ... test10  Average
cloudflare        24 ms   26 ms   ... 27 ms     26.00
pi-hole-stubby    73 ms   6 ms    ... 16 ms     32.90
google            39 ms   23 ms   ... 27 ms     35.70
cleanbrowsing     34 ms   31 ms   ... 37 ms     36.90
neustar           37 ms   33 ms   ... 37 ms     38.00
level3            46 ms   30 ms   ... 41 ms     39.00
norton            41 ms   33 ms   ... 42 ms     39.40
quad9             50 ms   45 ms   ... 35 ms     43.90
adguard           48 ms   49 ms   ... 46 ms     50.60
opendns           20 ms   26 ms   ... 22 ms     56.50
yandex            72 ms   120 ms  ... 62 ms     99.20
freenom           80 ms   168 ms  ... 148 ms    111.80
comodo            1000 ms 1000 ms ... 1000 ms   815.00
```