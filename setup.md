---
layout: home
title: Setup
nav_order: 2
nav_titles: true
titles_max_depth: 2
---

This setup page is specific to the Multipath TCP support in the Linux kernel.

## Kernel version

MPTCP support debuted with version 5.6 of the Linux kernel. It has continued to
evolve, and is still evolving today. See the
[ChangeLog](https://github.com/multipath-tcp/mptcp_net-next/wiki/#changelog) for
more details.

For this reason, we do recommend you to use a
[recent kernel](https://www.kernel.org/), ideally the last stable version, or
the last "long term support" (LTS) one.

Note that the RedHat/CentOS kernels have a good support for MPTCP, where new
features and bug fixes are regularly backported.


## Enable MPTCP

{: .info }
Most recent GNU/Linux distributions support MPTCP by default. It is very
likely MPTCP is already enabled, and you can skip this section.

### Linux kernel build configuration

The Linux kernel being used has to be compiled with `CONFIG_MPTCP=y` and
`CONFIG_MPTCP_IPV6=y` options, and ideally `CONFIG_INET_MPTCP_DIAG=y/m`. If not,
please report this to your GNU/Linux distribution: MPTCP in the kernel doesn't
add much overhead, and is enabled in most main Linux distributions (Debian,
Ubuntu, RedHat, Fedora, etc.), and other specific ones like Raspbian.

{: .note}
Note that `CONFIG_MPTCP_IPV6=y` requires `IPV6` to be inlined (`=y`), and not
built as a module (`=m`). Having `IPV6` inlined is recommended by NetDev
maintainers anyway: today, it is very likely that IPv6 will be used, e.g. for
the loopback address.

### Enable the creation of MPTCP sockets

If available, MPTCP should be enabled by default. If not, change this `sysctl`
knob:

```bash
sysctl net.mptcp.enabled=1
```

{: .note}
Note that MPTCP can also be blocked by SELinux, eBPF, etc. Please check with
your system administrators if it is the case.


## Force applications to use MPTCP

By default, applications will only use MPTCP if it has been explicitly
requested when creating a [stream network socket](https://en.wikipedia.org/wiki/Network_socket)
to communicate with the outside work. In other words, you likely have to enable
an option in the app you want to use with MPTCP, or to force MPTCP by changing
the behaviour of an app. The best is to have MPTCP supported [natively](implementation.html)
by applications, so they know where MPTCP is being used, and they can act
accordingly, e.g. force a fallback to TCP if MPTCP is not supported by the
kernel, etc.

Apps can be forced to use MPTCP with one of the following methods:

- [mptcpize](https://www.mankier.com/8/mptcpize): internally, it uses the
  [`LD_PRELOAD`](https://en.wikipedia.org/wiki/LD_PRELOAD) technique to force
  creating `MPTCP` sockets, instead of a `TCP` ones. Only `MPTCP` sockets will
  then be created, instead of `TCP`.

  ```bash
mptcpize run <command>
mptcpize enable <systemd unit>
  ```

- [GODEBUG](https://go-review.googlesource.com/c/go/+/507375): For applications
  written in GO, the libC is not used, so `mptcpize` does not work. Since GoLang
  1.21, it is possible to force MPTCP by setting the environment variable
  `GODEBUG=multipathtcp=1`:

  ```bash
GODEBUG=multipathtcp=1 <command>
  ```

- [eBPF](https://ebpf.io/what-is-ebpf/): since kernel v6.6, it is possible to
  change the socket being created per cGroup. A small eBPF program -- e.g.
  [mptcpify](https://elixir.bootlin.com/linux/latest/source/tools/testing/selftests/bpf/progs/mptcpify.c) --
  can be used, see [this example](https://git.kernel.org/pub/scm/linux/kernel/git/bpf/bpf-next.git/commit/?id=ddba122428a7).

- [SystemTap](https://sourceware.org/systemtap/) can also be used to modify the
  `socket` system call. See this [documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/getting-started-with-multipath-tcp_configuring-and-managing-networking#preparing-rhel-to-enable-mptcp-support_getting-started-with-multipath-tcp)
  for more details about that.


## Using multiple IP addresses

To be able to use multiple IP addresses on a host to create multiple *subflows*
(paths), the MPTCP path-manager needs to know which IP addresses can be used.

{: .info}
> A server having only one network interface does not need to configure anything
> else: the client will create additional subflows as needed.
>
> It might be interesting to announce additional IPv4/6 addresses. Some clients
> might be connected to networks having only an IPv4 or an IPv6 address. Also
> consider that IPv4 and IPv6 packets are often routed differently through some
> networks, resulting in different latencies.

### Path-Manager configuration

With the default in-kernel MPTCP path-manager, additional IP addresses need to
be specified.

This configuration can be automated with tools like
[Network Manager](https://networkmanager.dev) -- in command lines, look for
`mptcp-flags` in the [settings](https://networkmanager.dev/docs/api/latest/nm-settings-nmcli.html) --
and [mptcpd](https://mptcpd.mptcp.dev). Here, the focus is on manual
configuration, using the [`ip mptcp`](https://man7.org/linux/man-pages/man8/ip-mptcp.8.html)
command.

{: .note}
With the userspace MPTCP path-manager -- `sysctl net.mptcp.pm_type=0` -- the
configuration has to be done on the userspace daemon side.

#### Endpoints

MPTCP endpoints can be configured with

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

The IP address is an IPv4 or IPv6 address.

#### Limits

It is also important to make sure the limits are high enough:

```sh
ip mptcp limits set [ subflows NR ] [ add_addr_accepted NR ]
```

`subflows` is the limit of created and accepted subflows (paths), and
`add_addr_accepted` is the limit of accepted `ADD_ADDR` -- IP address
notification from the other peer -- that will result in the creation of
subflows.

#### Example

- Servers can announce extra IP addresses:
```sh
ip mptcp endpoint add 10.2.2.2 signal
```

- Clients can create additional subflows from a cellular interface, and flag
  this subflow as "backup", to be used to carry data only if the main path is
  unavailable:
```sh
ip mptcp endpoint add 100.64.1.134 subflow backup
```

### Manual routing configuration

<details markdown="block">
<summary>Only if MPTCP endpoints have <b>not</b> been configured with a network interface </summary>

The system needs to know how to route packets from a specific IP address to the
correct network interface.

{: .warning}
This manual routing configuration should not be required if the MPTCP endpoints
have been configured with a network interface `dev <interface>`, and if the
GNU/Linux distribution has automatically configured default route attached to
each network interface. To verify the latter, please check if the following
command `ip route show default` lists all the IPs you want to use with a `dev`
and a `src`.

To be able to use multiple paths from different local IP addresses at the same
time, it is then required to configure the system to route the traffic from a
specific local IP address (e.g. the one of the Wi-Fi) through the correct
interface, and not the default one. Such configuration can be automated with
tools like [Network Manager](https://networkmanager.dev), but here, we will
focus on the manual configuration, using
[`ip route`](https://man7.org/linux/man-pages/man8/ip-route.8.html) and
[`ip rule`](https://man7.org/linux/man-pages/man8/ip-rule.8.html) commands.

For each (additional) interface that will be used with MPTCP, run the following
commands with the correct IP address and a different table number:

```sh
ip rule add from <local interface IP address> table <table number>
ip route add default via <default gateway IP> dev <interface name> table <table number>
```

This configuration might need to be done with both IPv4 and IPv6 addresses.

#### Example

On a system with 3 network interfaces:

- Ethernet:
  - IP Address: 10.1.1.2
  - Gateway (next hop): 10.1.1.1

- Wi-Fi:
  - IP Address: 192.168.1.2
  - Gateway (next hop): 192.168.1.1

- Cellular:
  - IP Address: 100.64.1.134
  - Gateway (next hop): 100.64.1.133

Where the Ethernet interface is the default one:

```sh
$ ip route show default
default via 10.1.1.1 dev eth0 (...) metric 100
default via 192.168.1.1 dev wlan0 (...) metric 600
default via 100.64.1.133 dev usb0 (...) metric 1000
```

It is then required to configure the routing for the Wi-Fi and the cellular
interface, not to have the traffic routed only through the Ethernet interface:

```sh
ip rule add from 192.168.1.2 table 42
ip route add default via 192.168.1.1 dev wlan0 table 42

ip rule add from 100.64.1.134 table 43
ip route add default via 100.64.1.133 dev wlan0 table 43
```
</details> {: .ctsm}
