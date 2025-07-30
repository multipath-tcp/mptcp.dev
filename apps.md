---
layout: home
title: Support in Apps and Operating Systems
nav_order: 9
nav_titles: true
titles_max_depth: 2
---

## Linux apps

Applications listed below support MPTCP natively since the version that is
mentioned. Please note that other apps can be forced to use it by following
[these instructions](setup.html#force-applications-to-use-mptcp).

| Name | Version | How to use |
| --- | --- | --- |
| [Apache HTTP Server](https://httpd.apache.org) | [2.5.1](https://svn.apache.org/viewvc?view=revision&revision=1920586) | [`Listen 80 options=multipathtcp`](https://github.com/apache/httpd/pull/476/commits/0d56d533f4af) or [`ProxyPass multipathtcp=On`](https://github.com/apache/httpd/pull/476/commits/dfa6aec0dc74) |
| [Apache Traffic Server](https://trafficserver.apache.org/) | [9.2.4](https://github.com/apache/trafficserver/pull/10701) | [`server_ports: <port> mptcp`](https://docs.trafficserver.apache.org/en/latest/admin-guide/files/records.yaml.en.html) |
| [Caddy](https://caddyserver.com) | [v2.7.4](https://github.com/caddyserver/caddy/releases/tag/v2.7.4) | Enabled by default since [v2.10.0](https://github.com/caddyserver/caddy/releases/tag/v2.10.0) (Go 1.24), controllable with [`GODEBUG=multipath=`](https://go.dev/doc/godebug) since [v2.7.4](https://github.com/caddyserver/caddy/releases/tag/v2.7.4) (Go 1.21) |
| [cURL](https://curl.se/) | [8.9.0](https://github.com/curl/curl/pull/13278) | [`--mptcp`](https://curl.se/docs/manpage.html) |
| [dae](https://github.com/daeuniverse/dae) | [v0.8.0](https://github.com/daeuniverse/dae/pull/601) | [`global { mptcp: true }`](https://github.com/daeuniverse/dae/blob/main/example.dae) |
| [Envoy](https://www.envoyproxy.io/) | [1.21.0](https://github.com/envoyproxy/envoy/pull/18780) | [`"enable_mptcp": "true"`](https://www.envoyproxy.io/docs/envoy/v1.21.6/api-v3/config/listener/v3/listener.proto#envoy-v3-api-field-config-listener-v3-listener-enable-mptcp) (`listening` config) |
| [freenginx](https://freenginx.org) | [1.27.5](https://freenginx.org/hg/nginx/rev/cb20978439c8) | [`listen <address|port> multipath`](https://freenginx.org/en/docs/http/ngx_http_core_module.html#multipath) |
| [GoLang](https://go.dev) | [1.21](https://github.com/golang/go/issues/56539) | Apps written in Go can easily [enable MPTCP support](implementation.html). [Since 1.24](https://go-review.googlesource.com/c/go/+/607715), it is enabled on the server side by default |
| [HAProxy](https://www.haproxy.org) | [3.1.0](https://git.haproxy.org/?p=haproxy.git;a=commit;h=20efb856e) | [`mptcp{,4,6}@<address>`](https://github.com/haproxy/haproxy/blob/master/examples/mptcp.cfg) |
| [iperf3](https://software.es.net/iperf/) | [3.19](https://github.com/esnet/iperf/pull/1661) | `-m` or `--mptcp` |
| [Lighttpd 1.4](https://www.lighttpd.net/) | [1.4.76](https://github.com/lighttpd/lighttpd1.4/pull/132) | [`server.feature-flags = ( "server.network-mptcp" => "enable" )`](https://redmine.lighttpd.net/projects/lighttpd/wiki/Server_feature-flagsDetails) |
| [QEmu](https://www.qemu.org/) | [6.1](https://lore.kernel.org/qemu-devel/20210421112834.107651-1-dgilbert@redhat.com/) | [`<ip>:<port>,mptcp`](https://www.qemu.org/docs/master/interop/qemu-qmp-ref.html#qapidoc-48) |
| [Shadowsocks libev](https://github.com/shadowsocks/shadowsocks-libev) | [v3.3.6](https://github.com/shadowsocks/shadowsocks-libev/pull/2902) | [`--mptcp`](https://github.com/shadowsocks/shadowsocks-libev) |
| [Shadowsocks Rust](https://github.com/shadowsocks/shadowsocks-rust) | [v1.16.0](https://github.com/shadowsocks/shadowsocks-rust/pull/1157) | [`--mptcp`](https://github.com/shadowsocks/shadowsocks-rust) |
| [Shadowsocks Go](https://github.com/database64128/shadowsocks-go) | [v1.9.0](https://github.com/database64128/shadowsocks-go/commit/49a094a3722a) | [`servers.tcpListeners.multipath`](https://github.com/database64128/shadowsocks-go/blob/673cc9217ef5/docs/config.json#L33), [`clients.multipathTCP`](https://github.com/database64128/shadowsocks-go/blob/673cc9217ef5/docs/config.json#L297) |
| [sing-box](https://sing-box.sagernet.org) | [v1.4.0](https://github.com/SagerNet/sing-box/commit/1019ecfdcfb7) | [`"tcp_multi_path": true`](https://sing-box.sagernet.org/configuration/shared/listen/#tcp_multi_path) |
| [SystemD](https://systemd.io/) | [v257](https://github.com/systemd/systemd/pull/32958) | [`SocketProtocol=mptcp`](https://www.freedesktop.org/software/systemd/man/latest/systemd.socket.html) (`[Socket]` section) |
| [Traefik](https://traefik.io) | [v2.10.5](https://github.com/traefik/traefik/releases/tag/v2.10.5) | Enabled by default since [v2.11.26](https://github.com/traefik/traefik/releases/tag/v2.11.26) / [v3.4.2](https://github.com/traefik/traefik/releases/tag/v3.4.2) / [v3.5.0-rc1](https://github.com/traefik/traefik/releases/tag/v3.5.0-rc1) (Go 1.24), controllable with [`GODEBUG=multipath=`](https://go.dev/doc/godebug) since [v2.10.5](https://github.com/traefik/traefik/releases/tag/v2.10.5) (Go 1.21) |
| [v2ray-core](https://github.com/v2fly/v2ray-core) | [v5.17.0](https://github.com/v2fly/v2ray-core/pull/3109) | [`"mptcp": true`](https://www.v2fly.org/en_US/config/transport.html#sockoptobject) |
| [Valkey](https://valkey.io) | [v8.2.0](https://github.com/valkey-io/valkey/pull/1811) | [`mptcp yes`](https://github.com/valkey-io/valkey/commit/4a92db917858696120c9ef078b08f8e17c21125b#diff-dd9a6c95849b36490b9bae44e9f0d3dbcd7a93080cf2d5aee6d6033586ab8fa4R158), [`repl-mptcp yes`](https://github.com/valkey-io/valkey/commit/c9c49b446623a6b626f6b61663a8768b68431739#diff-dd9a6c95849b36490b9bae44e9f0d3dbcd7a93080cf2d5aee6d6033586ab8fa4) and [`--mptcp`](https://github.com/valkey-io/valkey/pull/2067) |

Do not hesitate to add native MPTCP support in more apps! For some ideas, feel
free to look at this [spreadsheet](https://docs.google.com/spreadsheets/d/1F2-v4Dhdn0rMyJZ3m5chyNiwg7oj0rpSR11GEykatJw/edit#gid=0).

## Linux distributions

Here is a list of Linux distributions where MPTCP is supported by the default
kernel since a certain version, with or without
[`mptcpd`](https://github.com/multipath-tcp/mptcpd) (including `mptcpize`)
package available in the official repositories:

| Name | Version | `mptcpd` | Comments |
| --- | --- | :---: | --- |
| [ArchLinux](https://archlinux.org) | ✅ | ✅ | Rolling release: MPTCP support is enabled since kernel v5.6. `mptcpd` is packaged in [AUR](https://aur.archlinux.org/packages/mptcpd). |
| [Alpine Linux](https://alpinelinux.org) | 3.20 | ✅ | [`mptcpd`](https://pkgs.alpinelinux.org/packages?name=mptcp*) is in [`edge/community`](https://gitlab.alpinelinux.org/alpine/aports/-/merge_requests/79478), expected to be in 3.22. |
| [Amazon Linux](https://aws.amazon.com/linux/amazon-linux-2023/) | 2023 | ❌ | `mptcpd` / `mptcpize` can be used from containers.  |
| [Azure Linux](https://github.com/microsoft/AzureLinux) | [3.0](https://github.com/microsoft/azurelinux/pull/10014) | ❌ | `mptcpd` / `mptcpize` can be used from containers. |
| [Debian](https://www.debian.org) | 12 | ✅ | |
| [Fedora](https://fedoraproject.org) | 36 | ✅ | |
| [Gentoo](https://www.gentoo.org) | ✅ | ✅ | Rolling release. |
| [Home Assistant OS](https://www.home-assistant.io) | 12.2 | ❌ | `mptcpd` / `mptcpize` can be used from containers. |
| [Kylin OS](https://www.kylinos.cn) | V11 | ✅ | |
| [NixOS](https://nixos.org) | ✅ | ✅ | `mptcpd` has been [accepted](https://github.com/NixOS/nixpkgs/pull/355928) in Jan. 2025 |
| [Open MPTCP Router](https://www.openmptcprouter.com) | v0.60 | ✅ | MPTCP-specific, where everything is automatically configured. |
| [OpenWrt](https://openwrt.org) | [24.10](https://github.com/openwrt/openwrt/pull/16786) | ✅ | `ip-full` package is required to use `ip mptcp` command. |
| [Raspberry Pi OS](https://www.raspberrypi.com/software/) | 20231004 | ✅ | |
| [RHEL](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux) | 9 | ✅ | MPTCP is available as a tech-preview since v8.3. |
| [Slackware](http://www.slackware.com/) | ✅ | ✅ | Version: to be completed. `mptcpd` is packaged in [SlackBuilds](https://slackbuilds.org/repository/15.0/network/mptcpd/) |
| [SUSE](https://www.suse.com) | ✅ | ❌ | Version: to be completed. |
| [Ubuntu](https://ubuntu.com) | 22.04 | ✅ | |
| [Void](https://voidlinux.org) | ✅ | ❌ | Rolling release: MPTCP support is enabled since kernel v5.10. |

Forks based on these distributions are not listed here. It is very likely that
if the upstream distribution supports it, the fork does it too.

Do not hesitate to contribute to the packaging, and to update this table!

## Tools

| Name | Version | Comment |
| --- | --- | --- |
| [BCC](https://github.com/iovisor/bcc) | [v0.35.0](https://github.com/iovisor/bcc/pull/5274) | `mptcpify` tool uses eBPF to force apps creating MPTCP sockets instead of TCP ones |
| [NFTables](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page) | [v1.0.2](https://lore.kernel.org/netdev/YhO5Pn+6+dgAgSd9@salvia/) | Can specify `mptcp` in TCP option, subtype IDs + symbols since [v1.1.2](https://lore.kernel.org/netdev/Z_1KxMUDT0D8e6wH@calendula/) |
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
| [Apple Siri](https://www.apple.com/siri/) | [>= iOS 7](https://www.tessares.net/apples-mptcp-story-so-far/) | Enabled by default |
| [Firefox](https://www.mozilla.org/en-US/firefox/browsers/mobile/ios/) | [v133](https://github.com/mozilla-mobile/firefox-ios/pull/21480) | Enabled by default |
| [Nextcloud](https://nextcloud.com) | [v6.1.4](https://github.com/nextcloud/NextcloudKit/pull/85) | Enabled by default |
| [VLC](https://www.videolan.org/vlc/) | [v4.0](https://code.videolan.org/videolan/vlc-ios/-/commit/210c88b3e4e0dac0e4f2d18b3e3dfbe664693658) | Enabled by default |

## Others?

Please open a new [pull request](https://github.com/multipath-tcp/mptcp.dev/pulls)
or an [issue](https://github.com/multipath-tcp/mptcp.dev/issues) if you know any
other apps with a community of users and supporting MPTCP natively.
