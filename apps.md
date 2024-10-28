---
layout: home
title: Apps supporting MPTCP
nav_order: 9
nav_titles: true
titles_max_depth: 2
---

Applications listed below support MPTCP natively since the version that is
mentioned. Please note that other apps can be forced to use it by following
[these instructions](setup.html#force-applications-to-use-mptcp).

## Linux apps

| Name | Version | How to use |
| --- | --- | --- |
| [Apache HTTP Server](https://httpd.apache.org) | [2.5.1](https://svn.apache.org/viewvc?view=revision&revision=1920586) | [`Listen 80 options=multipathtcp`](https://github.com/apache/httpd/pull/476/commits/0d56d533f4af) or [`ProxyPass multipathtcp=On`](https://github.com/apache/httpd/pull/476/commits/dfa6aec0dc74) |
| [Apache Traffic Server](https://trafficserver.apache.org/) | [9.2.4](https://github.com/apache/trafficserver/pull/10701) | [`server_ports: <port> mptcp`](https://docs.trafficserver.apache.org/en/latest/admin-guide/files/records.yaml.en.html) |
| [cURL](https://curl.se/) | [8.9.0](https://github.com/curl/curl/pull/13278) | [`--mptcp`](https://curl.se/docs/manpage.html) |
| [dae](https://github.com/daeuniverse/dae) | [v0.8.0](https://github.com/daeuniverse/dae/pull/601) | [`global { mptcp: true }`](https://github.com/daeuniverse/dae/blob/main/example.dae) |
| [Envoy](https://www.envoyproxy.io/) | [1.21.0](https://github.com/envoyproxy/envoy/pull/18780) | [`"enable_mptcp": "true"`](https://www.envoyproxy.io/docs/envoy/v1.21.6/api-v3/config/listener/v3/listener.proto#envoy-v3-api-field-config-listener-v3-listener-enable-mptcp) (`listening` config) |
| [HAProxy](https://www.haproxy.org) | [3.1.0](https://git.haproxy.org/?p=haproxy.git;a=commit;h=20efb856e) | [`mptcp{,4,6}@<address>`](https://github.com/haproxy/haproxy/blob/master/examples/mptcp.cfg) |
| [Lighttpd 1.4](https://www.lighttpd.net/) | [1.4.76](https://github.com/lighttpd/lighttpd1.4/pull/132) | [`server.feature-flags = ( "server.network-mptcp" => "enable" )`](https://redmine.lighttpd.net/projects/lighttpd/wiki/Server_feature-flagsDetails) |
| [QEmu](https://www.qemu.org/) | [6.1](https://lore.kernel.org/qemu-devel/20210421112834.107651-1-dgilbert@redhat.com/) | [`<ip>:<port>,mptcp`](https://www.qemu.org/docs/master/interop/qemu-qmp-ref.html#qapidoc-48) |
| [Shadowsocks libev](https://github.com/shadowsocks/shadowsocks-libev) | [v3.3.6](https://github.com/shadowsocks/shadowsocks-libev/pull/2902) | [`--mptcp`](https://github.com/shadowsocks/shadowsocks-libev) |
| [Shadowsocks Rust](https://github.com/shadowsocks/shadowsocks-rust) | [v1.16.0](https://github.com/shadowsocks/shadowsocks-rust/pull/1157) | [`--mptcp`](https://github.com/shadowsocks/shadowsocks-rust) |
| [SystemD](https://systemd.io/) | [v257](https://github.com/systemd/systemd/pull/32958) | [`SocketProtocol=mptcp`](https://www.freedesktop.org/software/systemd/man/latest/systemd.socket.html) (`[Socket]` section) |
| [v2ray-core](https://github.com/v2fly/v2ray-core) | [v5.17.0](https://github.com/v2fly/v2ray-core/pull/3109) | [`"mptcp": true`](https://www.v2fly.org/en_US/config/transport.html#sockoptobject) |

## Tools

| Name | Version | Comment |
| --- | --- | --- |
| [pTCPDump](https://github.com/mozillazg/ptcpdump) | [v0.24.0](https://github.com/mozillazg/ptcpdump/pull/152) | Can decode MPTCP options in TCP packets |
| [Syzkaller](https://github.com/google/syzkaller) | [Feb 2020](https://github.com/google/syzkaller/pull/1579) | Can exercise MPTCP sockets and Netlink APIs |
| [TCPDump](https://www.tcpdump.org/) | [4.5.0](https://github.com/the-tcpdump-group/tcpdump/commit/578dd316f3) | Can decode MPTCP options in TCP packets |
| [Wireshark](https://www.wireshark.org/) | [4.2.4](https://github.com/wireshark/wireshark/commit/3bc42dbf8e) | Can decode and filter MPTCP options |

## macOS apps

MPTCP is also available on [macOS](macOS.html) (client side only).

| Name | Version | How to use |
| --- | --- | --- |
| [iperf3-darwin](https://software.es.net/iperf/) | [11.0](https://developer.apple.com/documentation/foundation/nsurlsessionmultipathservicetype?language=objc) | `--apple-multipathsvc` |
| SSH | [OpenSSH-281.40.2](https://github.com/apple-oss-distributions/OpenSSH) | `applemultipath` option |
| [Shellfish](https://secureshellfish.app) | ? | Settings / Advanced / 🌍 Multipath TCP |

## iOS apps

MPTCP is also available on [iOS](macOS.html) (client side only).

| Name | Version | How to use |
| --- | --- | --- |
| [Apple Maps](https://www.apple.com/maps/) | [>= iOS 13](https://www.tessares.net/apple-pushes-multipath-tcp-further/) | Enabled by default |
| [Apple Music](https://www.apple.com/music/) | [>= iOS 13](https://www.tessares.net/apple-pushes-multipath-tcp-further/) | Enabled by default |
| [Siri](https://www.apple.com/fr/siri/) | [>= iOS 7](https://www.tessares.net/apples-mptcp-story-so-far/) | Enabled by default |
| [VLC](https://www.videolan.org/vlc/) | [v4.0](https://code.videolan.org/videolan/vlc-ios/-/commit/210c88b3e4e0dac0e4f2d18b3e3dfbe664693658) | Enabled by default |

## Misc.

| Name | Version |
| --- | --- |
| [Open MPTCP Router](https://www.openmptcprouter.com) | v0.60 |
| [OpenWrt](https://openwrt.org) | [v24](https://github.com/openwrt/openwrt/pull/16786) |

## Add native MPTCP support

Do not hesitate to add native MPTCP support in more apps! For some ideas, feel
free to look at this [spreadsheet](https://docs.google.com/spreadsheets/d/1F2-v4Dhdn0rMyJZ3m5chyNiwg7oj0rpSR11GEykatJw/edit#gid=0).

## Others?

Please open a new [pull request](https://github.com/multipath-tcp/mptcp.dev/pulls)
or an [issue](https://github.com/multipath-tcp/mptcp.dev/issues) if you know any
other apps with a community of users and supporting MPTCP natively.
