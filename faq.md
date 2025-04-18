---
layout: home
title: FAQ
nav_order: 8
nav_titles: true
titles_max_depth: 2
---

## Are there any security & privacy concerns?
MPTCP aims to maintain the same level of security as traditional TCP, with
specific mechanisms to counter common network attacks. Find out more in
[RFC 8684](https://datatracker.ietf.org/doc/html/rfc8684#name-security-considerations).
To be more secure than TCP, some modifications of the protocol would be needed,
e.g. [MPTCPsec](https://inl.info.ucl.ac.be/system/files/infocom_mptpcsec.pdf).

## Why & when should MPTCP be enabled by default?
<details markdown="block">
<summary>Here, servers and clients must be considered separately: MPTCP can be
enabled by default on servers, to be used only when requested, with a minimal
impact. On the client side, it might be useful to notify users that MPTCP is
being used by default. </summary>

- Clients typically have the most to gain by using MPTCP, but the benefits of
  MPTCP are mainly realized when users have
  [configured](setup.html#using-multiple-ip-addresses) their system to make use
  of its multipath capability. Still, even when only one network interface is
  available, MPTCP can be helpful in mobility use-cases that involve frequent
  switching from one network to another without stopping the connections. When
  servers don't support MPTCP, the connection continues in "plain" TCP.

- Servers usually don't directly benefit from MPTCP, due to their stable, fast,
  and reliable network connections. The client/server system as a whole is where
  the typical MPTCP improvements are experienced. There are server-specific use
  cases as well:
    - Switching from one network to another without a disconnection that would
      restart a long operation
    - Faster throughput by aggregating multiple TCP flows
    - Lowering latency by sending data in parallel on multiple subflows, so the
      lowest-latency path "wins". (Note: The MPTCP protocol allows for this but
      the Linux packet scheduler does not yet implement such a feature)
    - etc.

  We recommend enabling MPTCP on servers by default to let users choose whether
  to use MPTCP. When clients don't request to use MPTCP, server applications
  will create "plain" TCP sockets within the kernel when connections are
  `accept`ed, making the performance impact minimal.
</details> {: .ctsm}

## Are there any performance impacts when using MPTCP?
MPTCP is engineered to improve network resilience and utilization without
adversely affecting the performance of TCP applications. It adds a few bytes in
each TCP packet, causing a manageable overhead (~1%) that becomes advantageous
when leveraging multiple network paths, potentially increasing throughput and
reliability.

It is also important to note that, when clients don't request to use MPTCP,
server applications will create "plain" TCP sockets within the kernel when
connections are `accept`ed, making the performance impact minimal.

## Are there unsupported socket options?
On Linux, MPTCP supports most socket options handled by TCP. It is possible some
less common ones are not supported. If it is the case, please document your
use-case in a new
[issue](https://github.com/multipath-tcp/mptcp_net-next/issues/), and/or share
patches.

For example, MPTCP in the Linux kernel is
[currently](https://github.com/multipath-tcp/mptcp_net-next/issues/480) not
compatible with KTLS.

Generally, the same value is propagated to all subflows, including the ones
created after the calls to `setsockopt()`. eBPF can be used to set different
values per subflow.

## What are the supported operating systems?
<details markdown="block">
<summary>MPTCP is supported in the official Linux kernel starting with version
5.6. Any applications can <a href="implementation.html">easily use it</a>. The
adoption of MPTCP extends beyond to various platforms including macOS, load
balancers, and eventually FreeBSD and Windows. But... </summary>

The use of MPTCP on macOS differs from Linux:
- Its usage is limited to the client side, to establish new MPTCP connections
  only.
- It is most straightforward when applications use the system's
  [frameworks](https://developer.apple.com/documentation/foundation/nsurlsessionconfiguration/improving_network_reliability_using_multipath_tcp),
  as explained on the [macOS](/macOS.html) page.
- There is some documentation for the
  [connectx](https://opensource.apple.com/source/xnu/xnu-7195.81.3/bsd/man/man2/connectx.2.auto.html)
  system call that mentions multipath support if the socket has been created with
  the `AF_MULTIPATH` (`39`) domain. However, it is not clear what is required to
  use these interfaces. It appears that MPTCP connections can be created, but
  additional subflows cannot be created. There is no public
  example code from Apple, so this method doesn't seem to be recommended.

On FreeBSD, there was an ongoing implementation, but that was years ago, and it
is not working today according to
[this](http://www-cs-students.stanford.edu/~sjac/freebsd_mptcp_info.html).

There are other implementations, but on specific systems (Citrix load balancer,
userspace, etc.): more details
[here](http://blog.multipath-tcp.org/blog/html/2018/12/15/apple_and_multipath_tcp.html).

It is possible to use MPTCP on Windows with
[WSL2](https://perso.uclouvain.be/tom.barbette/mptcp-on-windows-with-wsl2/).
</details> {: .ctsm}

## MPTCP vs. QUIC
MPTCP enhances TCP's functionality at the transport layer by enabling multipath
capabilities, whereas QUIC, built atop UDP, focuses on reducing latency and
improving connection migration. While both propose multipath functionality,
their development and standardization stages differ.

Multipath capability in QUIC is not yet standardized. Here is the
[draft](https://quicwg.org/multipath/draft-ietf-quic-multipath.html).
Applications might have more configuration options to be able to select
different paths, and control data flow over those paths.

## MPTCPv0 vs. MPTCPv1
There are two different versions of the protocol:
[RFC 6824](https://datatracker.ietf.org/doc/html/rfc6824) (MPTCPv0, obsolete) and
[RFC 8684](https://datatracker.ietf.org/doc/html/rfc8684) (MPTCPv1, current).
The upstream Linux kernel only supports v1. A previous
[out-of-tree kernel](https://github.com/multipath-tcp/mptcp) supported both
v0 and v1, but it is now recommended to use the
[upstream](https://github.com/multipath-tcp/mptcp_net-next/) releases instead.

## What about middleboxes?
MPTCP is meticulously designed to ensure fallback to standard TCP when necessary.
This ensures uninterrupted connectivity amidst the presence of NATs and
firewalls (middleboxes) that might remove "unknown" TCP options like MPTCP.

If you notice that MPTCP is not allowed in your network, and "plain" TCP is used
instead, please report this issue to the people in charge of your network: it is
very likely a mistake that MPTCP is not allowed.

## How should applications handle missing MPTCP support?
Applications supporting MPTCP natively should first try to create an MPTCP
socket, then fallback to "plain" TCP in case of errors. If the Linux kernel does
not support MPTCP, a proper error will be returned when creating the socket. The
kernel version and its config can be different at build-time and at run-time
(e.g. some builders are using chroot/containers with up-to-date software
programs, but an old stable kernel). So it is recommended to do such checks at
run-time, letting the kernel returning an error if it is not supported, rather
than trying to guess at build-time what the end-user will have at run-time.

## What build time checks are needed/recommended?
Since a common practice is to compile source code on a different machine,
potentially with an older kernel version running on the build machine, it is
recommended not to restrict MPTCP utilization at build-time other than checking
than the target is Linux.

Note that it might be necessary to manually define `IPPROTO_MPTCP`, because old
libC versions might not have it. See the [Implementation Guide](implementation.html)
for more details about that.

## Why is <code>IPPROTO_MPTCP</code> not defined?
Since a program is not always compiled on the system it runs on, it is
recommended to manually define `IPPROTO_MPTCP` if the symbol is not already
defined:
```js
#ifndef IPPROTO_MPTCP
#define IPPROTO_MPTCP 262
#endif
```

## How to check if MPTCP is working?

The easiest way is to check with a special test server that will confirm MPTCP
is being used and return that info to the client, e.g.
```bash
$ mptcpize run curl https://check.mptcp.dev
You are using MPTCP.
```

It is also possible to use tools like `ss -Mia`, `tcpdump` and `wireshark`, or
check counters with `nstat` or directly in `/proc/net/netstat`.

## How to bootstrap a kernel development environment?

If you are interested in contributing to MPTCP in the Linux kernel, a
[Docker image](https://docs.docker.com) can be used to create a basic kernel
development environment. Download the
[kernel source code](https://github.com/multipath-tcp/mptcp_net-next/) and then
start the container:
```bash
$ cd [kernel source code]
$ docker run -v "${PWD}:${PWD}:rw" -w "${PWD}" --privileged --rm -it \
    --pull always mptcp/mptcp-upstream-virtme-docker:latest \
    manual-normal
```

More details on the
[MPTCP Upstream Virtme Docker](https://github.com/multipath-tcp/mptcp-upstream-virtme-docker#readme)
page.

## High number of retransmissions / dropped packets at the NIX RX queue level?

Even with MPTCP, subflow processing is done by the TCP stack. The main
difference with plain TCP is that this processing does not use the socket
backlog and always happens in second-level interrupt handlers called
[*bottom half*](https://en.wikipedia.org/wiki/Interrupt_handler) (BH) in Linux.
When the host is under heavy load, BH processing happens in `ksoftirqd` context,
and there is some latency between the `ksoftirqd` scheduling and the moment
`ksoftirqd` actually runs the handler. This depends on process scheduler
decisions (and settings).

A way to reduce these retransmissions and avoid dropped packets at the NIC level
is to increase the NIC RX queue. See issue
[#253](https://github.com/multipath-tcp/mptcp_net-next/issues/253) for more
details.

## Is GRO supported with MPTCP?

Software Generic Receive Offload (GRO) will merge packets when the MAC headers
are identical, but only specific TCP and IP headers can differ. In other words,
if the TCP options of one packet following another one are identical, they can
be merged.

MPTCP adds TCP options in each packet, but when TCP Segmentation Offload (TSO)
is used, the same MPTCP options will be duplicated in each packet. When such
packets are received, and if all the other conditions are met, the receiver-side
software GRO can re-create the sender's original pre-TSO packet because the TCP
options of the segmented packets are all identical.

With hardware GRO, in theory, NICs can use the same technique. But some of them,
like many Intel cards, are more "cautious", and they will not merge TCP packets
if they contain non-supported TCP options, i.e. anything but TCP timestamps.
With such NICs, hardware GRO cannot work with MPTCP and a fallback to software
GRO will be done.

When comparing MPTCP performance with another protocol, it is then interesting
to check if hardware GRO is used in both cases.

## How to enable MPTCP support with OpenSSH?

<details markdown="block">
<summary>So far, OpenSSH maintainers do not want to add a "native" MPTCP
support, because this feature is not available on all platforms, but "only" on
Linux distributions and macOS:
<a href="https://github.com/openssh/openssh-portable/pull/335" target=_blank>[1]</a>
<a href="https://github.com/openssh/openssh-portable/pull/547" target=_blank>[2]</a>.
Maybe this native support will come later on. In the meantime, there are
workarounds: </summary>
- On the server side:
  - If systemd 257 or newer is in charge of creating the SSH socket, edit its
    config with `systemctl edit ssh.socket` to add the following lines, then
    restart the socket with `systemctl restart ssh.socket`:
    ```
    [Socket]
    SocketProtocol=mptcp
    ```
  - If an older systemd version is used -- or if it is not in charge of the SSH
    socket -- first, disable the SSH socket if used:
    ```
    systemctl disable --now ssh.socket
    rm -f /etc/systemd/system/ssh.service.d/00-socket.conf
    rm -f /etc/systemd/system/ssh.socket.d/addresses.conf
    systemctl daemon-reload
    systemctl enable --now ssh.service
    ```
    Then force the SSH service to create MPTCP sockets instead of TCP ones:
    ```
    mptcpize enable ssh.service
    ```
  - If another system manager is used, prefix the execution of `sshd` with
    `mptcpize run`, or set `LD_PRELOAD` to the full path of
    `libmptcpwrap.so.0.0.1`.
- On the client side:
  - Prefix the SSH command line (or `scp`, `rsync`, `git`, etc.) with
    `mptcpize run`, e.g.
    ```
    mptcpize run ssh example.org
    ```
  - An alternative is to set the `ProxyCommand` option to use `mptcpize run`,
    e.g. by using this line in the `~/.ssh/config` file:
    ```
    Host (...)
        ProxyCommand mptcpize run ssh -W %h:%p -l %r -p %p %h
    ```
    This proxy command is less efficient, because it will force `ssh` to be
    launched a second time as a proxy. But it is useful not to require a prefix
    for all `ssh` commands, or if SSH is used by other tools, e.g. `git`, a file
    manager like Nautilus, Filezilla, etc.
</details> {: .ctsm}
