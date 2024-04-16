---
layout: home
title: Implementation guide (Devs)
nav_order: 4
nav_titles: true
titles_max_depth: 2
---

This guide is for **application developers** working to add native MPTCP support
when running on Linux. End-users can also [force some apps to use
MPTCP](setup.html#force-applications-to-use-mptcp), but that's more of a
workaround, because the app will not notice a different socket type is being
used.

## MPTCP socket
On Linux, MPTCP can be used by selecting MPTCP instead of TCP when
creating the `socket`:

<div class="language-c highlighter-rouge">
  <div class="highlight">
    <pre class="highlight"><code><span class="color-main">socket</span>(<span class="color-blue">AF_INET</span>(6), <span class="color-yellow">SOCK_STREAM</span>, <span class="color-green">IPPROTO_MPTCP</span>)</code></pre>
  </div>
</div>

That's it!

{: .note}
Note that `IPPROTO_MPTCP`{: .color-green} is defined as `262`{: .text-yellow-300}.
It can be manually defined if it is not already handled in your system headers.

If MPTCP is not supported, `errno`{: .text-red-200} will be set to:
- `EINVAL`{: .text-red-200}: (*Invalid argument*): MPTCP is not available, on
  kernels < 5.6.
- `EPROTONOSUPPORT`{: .text-red-200} (*Protocol not supported*): MPTCP has not
  been compiled, on kernels >= v5.6.
- `ENOPROTOOPT`{: .text-red-200} (*Protocol not available*): MPTCP has been
  disabled using `net.mptcp.enabled` sysctl knob.
- Even if it is unlikely, MPTCP can also be blocked using different techniques
  (SELinux, eBPF, etc.). In this case, check with your sysadmin.


## Examples in different languages

For more detailed examples, please see this [Git
repository](https://github.com/mptcp-apps/mptcp-hello/). Do not hesitate to
contribute here and there!

<details markdown="block">
<summary>Example in C </summary>

`IPPROTO_MPTCP` has been added in GNU C library (`glibc`) in version
[2.32](https://sourceware.org/git/?p=glibc.git;h=f9ac84f92f151)

```c
#include <sys/socket.h>
int s = socket(AF_INET, SOCK_STREAM, IPPROTO_MPTCP)
if (s < 0) /* fallback to TCP */
    s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
```
</details> {: .ctsm}

<details markdown="block">
<summary>Example in Python </summary>

`socket.IPPROTO_MPTCP` has been added in [CPython 3.10](https://bugs.python.org/issue43571).

```python
import socket
try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM, socket.IPPROTO_MPTCP)
except:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM, socket.IPPROTO_TCP)
```
</details> {: .ctsm}

<details markdown="block">
<summary>Example in Go </summary>

`SetMultipathTCP` support has been [added](https://github.com/golang/go/issues/56539)
in [GoLang 1.21](https://tip.golang.org/doc/go1.21#netpkgnet).

```go
import ("net" "context")

// Client
d := &net.Dialer{}
d.SetMultipathTCP(true)
c, err := d.Dial("tcp", *addr)

// Server
lc := &ListenConfig{}
lc.SetMultipathTCP(true)
ln, err := lc.Listen(context.Background(), "tcp", *addr)
defer ln.Close()
```
</details> {: .ctsm}
