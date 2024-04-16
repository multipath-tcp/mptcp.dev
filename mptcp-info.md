---
layout: home
title: MPTCP info (Devs)
nav_order: 5
nav_titles: true
titles_max_depth: 2
---

This section is for **application developers** trying to access MPTCP
specific info from their app on Linux. End-users can retrieve info via other
interfaces, see the [Debugging](debugging.html) section.

Like TCP, it is possible to retrieve MPTCP info from the socket, via
[`getsockopt()`](https://www.man7.org/linux/man-pages/man2/getsockopt.2.html),
using `MPTCP_INFO` (`1`). It is also possible to retrieve info from the subflows
via `MPTCP_TCPINFO` (`2`), `MPTCP_SUBFLOW_ADDRS` (`3`), and `MPTCP_FULL_INFO`
(`4`).

## MPTCP socket level

`MPTCP_INFO` is the easiest way to retrieve info at the MPTCP level. Note that
these stats are also exposed via the Netlink Diag interface, and can be seen
with `ss -Mi` for example.

To retrieve info, `getsockopt(MPTCP_INFO)` can be used: the kernel will fill a
given buffer of a given size with items following the `mptcp_info` structure,
and update the size with the number of bytes that were written:

<div class="language-c highlighter-rouge">
  <div class="highlight">
    <pre class="highlight"><code>int err = <span class="color-main">getsockopt</span>(<span class="color-blue">mptcp_fd</span>, <span class="color-yellow">SOL_MPTCP</span>, <span class="color-green">MPTCP_INFO</span>, &<span class="color-red">buffer</span>, &<span class="color-orange">optlen</span>)</code></pre>
  </div>
</div>

{: .warning}
It is important to note that this `mptcp_info` structure adds fields in newer
kernel versions, and might continue to grow. It means that the userspace
application and the kernel need to work with different versions of this
structure. The kernel will share the number of bytes written using the
`optlen`{: .color-orange} variable, so the userspace application can detect what
the current kernel has exposed.

The kernel might write less data than what the userspace application expects.
If the userspace application asks for less data than what the kernel can
provide, the kernel will stop at the given size. To check the presence of an
option, there are two simple ways:

- Only use non-zero values ; the buffer had to be initialized to 0 first.
- Check the relative position of an entry in the structure, and compare it with
  the `optlen`{: .color-orange} value using the `offsetof` function, look at the
  example beflow for more details.

<details markdown="block">
<summary><code>struct mptcp_info</code> </summary>

```c
// Items are grouped by waves of addition
struct mptcp_info {
    // v5.9
    __u8   mptcpi_subflows;
    __u8   mptcpi_add_addr_signal;
    __u8   mptcpi_add_addr_accepted;
    __u8   mptcpi_subflows_max;
    __u8   mptcpi_add_addr_signal_max;
    __u8   mptcpi_add_addr_accepted_max;
    __u32  mptcpi_flags;
    __u32  mptcpi_token;
    __u64  mptcpi_write_seq;
    __u64  mptcpi_snd_una;
    __u64  mptcpi_rcv_nxt;

    // v5.12
    __u8   mptcpi_local_addr_used;
    __u8   mptcpi_local_addr_max;

    // v5.14
    __u8   mptcpi_csum_enabled;

    // v6.5
    __u32  mptcpi_retransmits;
    __u64  mptcpi_bytes_retrans;
    __u64  mptcpi_bytes_sent;
    __u64  mptcpi_bytes_received;
    __u64  mptcpi_bytes_acked;

    // v6.8
    __u8   mptcpi_subflows_total;

    // v6.10
    __u8   reserved[3];
    __u32  mptcpi_last_data_sent;
    __u32  mptcpi_last_data_recv;
    __u32  mptcpi_last_ack_recv;
};
```

Check [`include/uapi/linux/mptcp.h`](https://github.com/multipath-tcp/mptcp_net-next/blob/export/include/uapi/linux/mptcp.h)
to get the latest version.
</details> {: .ctsm}

<details markdown="block">
<summary><code>MPTCP_INFO</code> example in C </summary>

```c
int mptcp_fd; // 'mptcp_fd' has been created before with socket() or accept()
struct mptcp_info info = { 0 };
socklen_t optlen = sizeof(struct mptcp_info);

if (getsockopt(mptcp_fd, SOL_MPTCP, MPTCP_INFO, &info, &optlen) < 0)
    return; // handle errors here

// 'optlen' bytes have been written in the 'info' buffer: can be <= sizeof(struct mptcp_info).
// Check if a field is != 0, or if the kernel support the required info, e.g.:
if ((socklen_t)__builtin_offsetof(struct mptcp_info, mptcpi_subflows_total) < optlen)
    printf("Subflows: %u\n", info.mptcpi_subflows_total);
else
    printf("%s", "mptcpi_subflows_total is not available\n");
```
</details> {: .ctsm}

For more detailed examples, feel free to look at the
[MPTCP selftests](https://github.com/multipath-tcp/mptcp_net-next/blob/export/tools/testing/selftests/net/mptcp/mptcp_sockopt.c).

## Per-subflow Information

Since v5.16, it is possible to get the `TCP_INFO` structure and used IP
addresses for each subflow, thanks to `MPTCP_TCPINFO` and `MPTCP_SUBFLOW_ADDRS`.
However, retrieving everything means doing two `getsockopt()` calls: subflows
can be removed or added in-between system calls, making tracking difficult. To cope with
that `MPTCP_FULL_INFO` has been added in v6.5.

Here are the supported options:
- `MPTCP_TCPINFO`: Uses `struct mptcp_subflow_data`, followed by an array of
  `struct tcp_info`.
- `MPTCP_SUBFLOW_ADDRS`: Uses `struct mptcp_subflow_data`, followed by an
  array of `mptcp_subflow_addrs`.
- `MPTCP_FULL_INFO`: Uses `struct mptcp_full_info`, with one pointer to an
  array of `struct mptcp_subflow_info` (including the `struct mptcp_subflow_addrs`),
  and one pointer to an array of `struct tcp_info`, followed by the content of
  `struct mptcp_info`.

<details markdown="block">
<summary>Structures </summary>
<details markdown="block">
<summary><code>struct mptcp_subflow_data</code> </summary>

```c
struct mptcp_subflow_data {
    __u32  size_subflow_data;  /* size of this structure in userspace */
    __u32  num_subflows;       /* must be 0, set by kernel */
    __u32  size_kernel;        /* must be 0, set by kernel */
    __u32  size_user;          /* size of one element in data[] */
};
```
</details> {: .ctsm}

<details markdown="block">
<summary><code>struct mptcp_subflow_addrs</code> </summary>

```c
struct mptcp_subflow_addrs {
    union {
        __kernel_sa_family_t              sa_family;
        struct sockaddr                   sa_local;
        struct sockaddr_in                sin_local;
        struct sockaddr_in6               sin6_local;
        struct __kernel_sockaddr_storage  ss_local;
    };
    union {
        struct sockaddr                   sa_remote;
        struct sockaddr_in                sin_remote;
        struct sockaddr_in6               sin6_remote;
        struct __kernel_sockaddr_storage  ss_remote;
    };
};
```
</details> {: .ctsm}

<details markdown="block">
<summary><code>struct mptcp_full_info</code> </summary>

```c
struct mptcp_full_info {
    __u32              size_tcpinfo_kernel;  /* must be 0, set by kernel */
    __u32              size_tcpinfo_user;
    __u32              size_sfinfo_kernel;   /* must be 0, set by kernel */
    __u32              size_sfinfo_user;
    __u32              num_subflows;         /* must be 0, set by kernel (real subflow count) */
    __u32              size_arrays_user;     /* max subflows that userspace is interested in;
                                              * the buffers at subflow_info/tcp_info
                                              * are respectively at least:
                                              *  size_arrays * size_sfinfo_user
                                              *  size_arrays * size_tcpinfo_user
                                              * bytes wide
                                              */
    __aligned_u64      subflow_info;
    __aligned_u64      tcp_info;
    struct mptcp_info  mptcp_info;
};
```
</details> {: .ctsm}

<details markdown="block">
<summary><code>struct mptcp_subflow_info</code> </summary>

```c
struct mptcp_subflow_info {
    __u32                       id;
    struct mptcp_subflow_addrs  addrs;
};
```
</details> {: .ctsm}

Check [`include/uapi/linux/mptcp.h`](https://github.com/multipath-tcp/mptcp_net-next/blob/export/include/uapi/linux/mptcp.h)
file to get the latest version.
</details> {: .ctsm}

<details markdown="block">
<summary><code>MPTCP_FULL_INFO</code> example in C </summary>

```c
int mptcp_fd; // 'mptcp_fd' has been created before with socket() or accept()
struct mptcp_full_info full_info = { 0 };
socklen_t optlen = sizeof(struct mptcp_full_info);

// Restricted to two subflows in this example. Adapt the sizes if needed.
full_info.size_arrays_user = 2;
struct mptcp_subflow_info subflow_info[2] = { 0 };
struct tcp_info tcp_info[2] = { 0 };

// Set the size and addresses
full_info.size_sfinfo_user = sizeof(struct struct mptcp_subflow_info);
full_info.size_tcpinfo_user = sizeof(struct tcp_info);
full_info.subflow_info = (unsigned long)&subflow_info[0];
full_info.tcp_info = (unsigned long)&tcp_info[0];

if (getsockopt(fd, SOL_MPTCP, MPTCP_FULL_INFO, &full_info, &optlen) < 0)
    return; // handle errors here

for (int i = 0; i < MIN(full_info.size_arrays_user, full_info.num_subflows); i++) {
    printf("subflow %d:\n", i);
    printf("\tid: %u\n", subflow_info[i].id);
    printf("\trtt: %u\n", tcp_info[i].tcpi_rtt);
}

printf("token: %u\n", full_info.mptcp_info.mptcpi_token);
```
</details> {: .ctsm}

For more detailed examples, feel free to look at the
[MPTCP selftests](https://github.com/multipath-tcp/mptcp_net-next/blob/export/tools/testing/selftests/net/mptcp/mptcp_sockopt.c).


## Number of subflows

The number of subflows is available from multiple fields, but with small
differences:

| Structure | Field name | Description |
| --- | --- | --- |
| `mptcp_info` | `mptcpi_subflows` | The number of **additional** subflows |
| `mptcp_info` | `mptcpi_subflows_total` | The **total** number of subflows |
| `mptcp_subflow_data` | `num_subflows` | The **total** number of subflows |
| `mptcp_full_info` | `num_subflows` | The **total** number of subflows |

`mptcpi_subflows` shows the number of **additional** subflows, not taking
into account the initial one. `mptcpi_subflows_total` and `num_subflows` do
include the initial one if it is still attached to the MPTCP connection. It is
therefore recommended to use these last two fields.

## Check for TCP fallback

Since kernel v5.16, `getsockopt(MPTCP_INFO)` can be used to check if an
MPTCP connection fell back to TCP. If this `getsockopt()` call returns `-1`,
and `errno` is set to `EOPNOTSUPP` (v4) or `ENOPROTOOPT` (v6), it means the
MPTCP connection has fallen back to TCP at some point. In case of a client
(`connect()`), or for a server to check if an established connection has later
fallen back to TCP (should be rare), it is also required to check if the
`mptcpi_flags` field from the `mptcp_info` structure has the
`MPTCP_INFO_FLAG_FALLBACK` bit (`0x1`) set.

{: .warning}
On kernels < v5.16, `getsockopt(MPTCP_INFO)` will always fail, and `errno` will
also be set to `EOPNOTSUPP` (v4) or `ENOPROTOOPT` (v6). Do not use this method
on older kernels.

On the server side, it is possible to look at the protocol of the `accept`ed
sockets with `getsockopt(SO_PROTOCOL)`: if the client requested to use MPTCP,
the protocol will be set to `IPPROTO_MPTCP`. This can be used on kernels < 5.16
too.

<details markdown="block">
<summary>Example in C (<b>client only</b>) </summary>

{: .warning}
**Client only**: It requires **kernel >= 5.16**

```c
#define MPTCP_INFO 1
#define MPTCP_INFO_FLAG_FALLBACK 1
bool socket_is_mptcp(int client_fd)
{
    socklen_t len = sizeof(struct mptcp_info);
    struct mptcp_info info = { 0 };

    if (getsockopt(client_fd, SOL_MPTCP, MPTCP_INFO, &info, &len) < 0) {
        perror("kernel < v5.16: we cannot tell (workaround: use NetLink or ss -Mi)");
        return true;
    }
    return (info.mptcpi_flags & MPTCP_INFO_FLAG_FALLBACK) == 0;
}
```
</details> {: .ctsm}

<details markdown="block">
<summary>Example in C (<b>server only</b>: MPTCP requested?) </summary>

{: .warning}
**Server only**: it **only** checks if the client requested to use MPTCP

```c
bool client_requested_mptcp(int accept_fd)
{
    socklen_t len = 0;
    int protocol;

    if (getsockopt(accept_fd, SOL_SOCKET, SO_PROTOCOL, &protocol, &len) < 0) {
        perror("getsockopt(SO_PROTOCOL)");
        return true; /* cannot tell */
    }
    return protocol == IPPROTO_MPTCP; /* Always true on 'connect' and 'listen' sockets */
}
```
</details> {: .ctsm}

<details markdown="block">
<summary>Example in C (<b>server only</b>: full solution) </summary>

{: .warning}
**Server only**: Requires **kernel >= 5.16**, it also checks for fallback
that would have happened after the establishment of the connection (should be
rare)

```c
#define MPTCP_INFO 1
#define MPTCP_INFO_FLAG_FALLBACK 1
bool socket_is_mptcp(int accept_fd)
{
    socklen_t len = sizeof(struct mptcp_info);
    struct mptcp_info info = { 0 };

    /* kernel < 5.16 will always fail with errno set to EOPNOTSUPP (v4) or ENOPROTOOPT (v6) */
    if (kernel_version_lower(5, 16))
        return true; /* This method cannot be used: check the next example */

    if (getsockopt(accept_fd, SOL_MPTCP, MPTCP_INFO, &info, &len) < 0) {
        if (errno != EOPNOTSUPP && errno != ENOPROTOOPT)
            perror("getsockopt(MPTCP_INFO)"); /* Should not happen */
        return false; /* The client didn't ask to use MPTCP */
    }
    /* The connection has been established in MPTCP, check for fallback later (rare) */
    return (info.mptcpi_flags & MPTCP_INFO_FLAG_FALLBACK) == 0;
}
```
</details> {: .ctsm}

<details markdown="block">
<summary>Example in Go </summary>
Call [`MultipathTCP()`](https://pkg.go.dev/net#TCPConn.MultipathTCP) on
the `TCPConn`.
```go
d := &Dialer{}
d.SetMultipathTCP(true)
c, err := d.Dial("tcp", addr) // check for error + defer c.Close()
tcp, ok := c.(*TCPConn) // should not fail
mptcp, err := tcp.MultipathTCP() // 'mptcp' is a boolean
```
</details> {: .ctsm}
