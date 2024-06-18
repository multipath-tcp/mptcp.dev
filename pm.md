---
layout: home
title: Path-Manager
nav_order: 3
nav_titles: true
titles_max_depth: 2
---

The Path Manager is in charge of *subflows*, from creation to deletion, and also
address announcements. Typically, it is the client side that initiates subflows,
and the server side that announces additional addresses via the `ADD_ADDR` and
`REMOVE_ADDR` options.

```mermaid
graph LR;
    C_1(<div style="display: inline-block; min-width: 35px"><font size="7">fa:fa-mobile</font></div>)
    S_1((<div style="display: inline-block; min-width: 60px"><font size="7">fa:fa-cloud</font></div>))

    C_1 -. "Potential subflow" -.- S_1
    C_1 <== "Initial subflow" ==> S_1
    C_1 ~~~|"Subflows creation"| C_1
    S_1 ~~~|"Addresses announcement"| S_1

    linkStyle 0 stroke:orange;
    linkStyle 1 stroke:green;
```

As of Linux v5.19, there are two path managers controlled by the netns-aware
`net.mptcp.pm_type` sysctl knob: the [in-kernel](#in-kernel-path-manager) one
(type `0`), and the [userspace](#userspace-path-manager) one (type `1`).

## In-kernel Path-Manager

With the In-kernel Path-Manager, the same rules are applied to all connections.
Address endpoints and limits can be set to control its behavior.

### Configuration

This configuration can be automated with tools like
[Network Manager](https://networkmanager.dev) -- in command lines, look for
`mptcp-flags` in the [settings](https://networkmanager.dev/docs/api/latest/nm-settings-nmcli.html) --
and [mptcpd](https://mptcpd.mptcp.dev). Here, the focus is on the manual
configuration, using the [`ip mptcp`](https://man7.org/linux/man-pages/man8/ip-mptcp.8.html)
command.

#### Endpoints

MPTCP endpoints can be configured with this command:

```sh
ip mptcp endpoint add <IP address> dev <interface> [ signal | subflow ] [ backup ] [ fullmesh ]
```

{: .warning}
It is important to specify the network interface linked to the address by adding
`dev <interface>`. If not, the routing will probably not be done properly, and
will require manual configuration, see below:
[Manual Routing Configuration](#manual-routing-configuration).

One of the following flags needs to be set:
- `signal`: The endpoint will be announced to each peer via an MPTCP `ADD_ADDR`
  sub-option. Typically, a server would be responsible for this.
- `subflow`: The endpoint will be used to create an additional subflow using
  the given source IP address. A client would typically do this.

Optionally, the following flags can be set:
- `backup`: Subflows created from this endpoint instruct the peers to only send
  data on it when all non-backup subflows are unavailable.
- `fullmesh`: The MPTCP path manager will try to create an additional subflow
  for each known peer address, using this endpoint as the source IP address.

The IP address is an IPv4 or IPv6 address. The endpoints are netns-aware.

#### Example

- Servers can announce extra IP addresses:
```sh
ip mptcp endpoint add 10.2.2.2 dev eth0 signal
```

- Clients can create additional subflows from a cellular interface, and flag
  this subflow as "backup", to be used to carry data only if the main path is
  unavailable:
```sh
ip mptcp endpoint add 100.64.1.134 dev usb0 subflow backup
```

#### Limits

It is also important to make sure the limits are high enough:

```sh
ip mptcp limits set [ subflows NR ] [ add_addr_accepted NR ]
```

- `subflows` is the limit of additional created and accepted subflows (paths),
  for both the client and server sides (default is 2).
- `add_addr_accepted` is the limit of accepted `ADD_ADDR` -- IP address
  notification from the other peer -- that will result in the creation of
  subflows, typically only for the client side (default is 0).

The limits are per MPTCP connection, and netns-aware.

{: .note}
It is possible to reach the limits with fewer established subflows than
expected, e.g. when new subflow requests cannot reach the other peer. In case of
problem, please increase the limits, use `ss -Mai` to check the counters, and
modify the routing or firewall rules to avoid using certain paths between
specific IP addresses. For example, in a lab setup with dedicated links, use
specific routes rather than letting the kernel select the default route.


## Userspace Path-Manager

With the userspace MPTCP path-manager -- `sysctl net.mptcp.pm_type=1` --
different rules can be applied for each connection. The path-manager will then
need to be controlled by a userspace daemon, i.e.
[`mptcpd`](https://mptcpd.mptcp.dev). In this case, the configuration has to be
done on the userspace daemon side.

{: .note}
`mptcpd` can help to create custom userspace Path-Managers: please check this
[Plugins](https://github.com/multipath-tcp/mptcpd/wiki/Plugins) page for more
details about that.
