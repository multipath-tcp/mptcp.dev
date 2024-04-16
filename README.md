---
layout: home
title: MPTCP
nav_order: 1
---

# Multipath TCP for Linux

Multipath TCP (MPTCP) builds on top of TCP to improve connection
redundancy and performance by sharing a network data stream across
multiple underlying TCP sessions. The MPTCP v1 protocol is defined
in [RFC 8684](https://www.rfc-editor.org/rfc/rfc8684.html).

The Linux MPTCP community develops and maintains the MPTCP v1 stack in
the Linux kernel (v5.6 or later) and associated userspace tools and
libraries.

This site is new and still evolving, so please refer to the [Linux MPTCP Upstream Project wiki](https://github.com/multipath-tcp/mptcp_net-next/wiki) for additional information.

_For out-of-tree kernels before v5.6 and an implementation of the experimental [MPTCP v0](https://www.rfc-editor.org/rfc/rfc6824.html) protocol, see https://multipath-tcp.org/_

## Features

As of Linux v5.19, major features of MPTCP include:

* Support of the `IPPROTO_MPTCP` protocol in `socket()` system calls.
* Fallback from MPTCP to TCP if the peer or a middlebox do not support MPTCP.
* Path management using either an in-kernel or userspace path manager.
* Socket options that are commonly used with TCP sockets.
* Debug features including MIB counters, diag support (used by the `ss` command), and tracepoints.

See the
[ChangeLog](https://github.com/multipath-tcp/mptcp_net-next/wiki/#changelog)
for more details.

## Communication

* Mailing List: mptcp@lists.linux.dev (and [archives](https://lore.kernel.org/mptcp))
* IRC: [#mptcp](https://web.libera.chat/?nick=mptcp-dev-guest?#mptcp) on libera.chat
* Online [Meetings](https://github.com/multipath-tcp/mptcp_net-next/wiki/Meetings)

## Projects

* Maintained by MPTCP community members
  * Kernel development on GitHub: https://github.com/multipath-tcp/mptcp_net-next/
  * Multipath TCP Daemon: https://github.com/intel/mptcpd
    * The `mptcpd` daemon can do full userspace path management or control the in-kernel path manager.
    * Includes the `mptcpize` utility to allow legacy TCP binaries to use MPTCP.
  * Packetdrill with MPTCP enhancements: https://github.com/multipath-tcp/packetdrill
* Projects with MPTCP-related enhancements
  * [iproute2](https://wiki.linuxfoundation.org/networking/iproute2) (for the `ip mptcp` command)
  * [Network Manager](https://networkmanager.dev): MPTCP features are included starting with v1.40.
  * [Multipath TCP applications](https://github.com/mptcp-apps/): A project to coordinate MPTCP updates for popular TCP applications.

## Kernel Development

* [Git Repository](https://github.com/multipath-tcp/mptcp_net-next.git) ([branch descriptions](https://github.com/multipath-tcp/mptcp_net-next/wiki/Git-Branches))
* [Issue tracker](https://github.com/multipath-tcp/mptcp_net-next/issues)
* [Patchwork](https://patchwork.kernel.org/project/mptcp/)
* [Continuous Integration](https://github.com/multipath-tcp/mptcp_net-next/wiki/CI)
* [Testing](https://github.com/multipath-tcp/mptcp_net-next/wiki/Testing)
