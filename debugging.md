---
layout: home
title: Debugging
nav_order: 3
nav_titles: true
titles_max_depth: 2
---

To verify that everything is working correctly, or to debug issues, multiple
tools can be used.


## ss

The [`ss`](https://www.man7.org/linux/man-pages/man8/ss.8.html) command line
tool on Linux systems has an option to list MPTCP sockets: `-M`. The recommended
options are: `ss -Mani`.

- `M`, MPTCP sockets
- `a`, display all socket not just "established" ones
- `n`, prevent the port number to protocol conversion
- `i`, connection info matching that from Netlink Diag interface: `TCP_INFO`,
  `MPTCP_INFO`, etc.

{: .note}
Note that technically, each subflow (path) is a TCP connection. To see subflows,
`ss -tani` can be executed as root: subflows will be marked with `tcp-ulp-mptcp`.

`ss` can be used with extra filters, e.g. to restrict by connection states, IP
addresses, ports, devices, marks, CGroup, etc.


## ip mptcp

[`ip mptcp`](https://man7.org/linux/man-pages/man8/ip-mptcp.8.html) can be used
to configure the MPTCP path-manager, but also to monitor path-manager events:

```bash
ip mptcp monitor
```


## nstat

[`nstat`](https://www.man7.org/linux/man-pages/man8/nstat.8.html) can list
kernel MIB counters. MPTCP has a bunch of them, e.g.

```bash
nstat -asz | grep MPTcpExt
```

For more details about these counters, see:
[MPTCP Doc](https://mptcp-apps.github.io/mptcp-doc/mptcp-linux.html#analyzing-the-output-of-nstat).

When encountering an issue with MPTCP, it is often interesting to run `nstat -n`
before, and `nstat | grep Tcp` after the issue.


## TCPDump and WireShark

[TCPDump](https://www.tcpdump.org) and [WireShark](https://www.wireshark.org)
support MPTCP: they can be use to take packet traces, and recent enough versions
will show MPTCP options in TCP packets if present.
