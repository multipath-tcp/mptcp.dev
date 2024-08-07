---
layout: home
title: Implementation details (Kernel)
nav_order: 10
nav_titles: false
titles_max_depth: 2
---

The MPTCP protocol is described in [RFC 8684](https://datatracker.ietf.org/doc/html/rfc8684),
and also in the [MPTCP Doc](https://mptcp-apps.github.io/mptcp-doc/mptcp.html),
but neither of those describe the internals of the MPTCP implementation in the
Linux kernel.

A new socket type has been added for MPTCP for the userspace-facing socket.
The kernel is in charge of creating subflow sockets: they are TCP sockets
where the behavior is modified using TCP-ULP.

MPTCP listen sockets will create "plain" *accepted* TCP sockets if the
connection request from the client didn't ask for MPTCP.

If you are interested by **contributing to the kernel**, please check the
[Contributing](contributing.html) page and this
[MPTCP Upstream Virtme Docker](https://github.com/multipath-tcp/mptcp-upstream-virtme-docker)
repo.

This page needs to be improved, feel free to
[contribute](https://github.com/multipath-tcp/mptcp.dev/edit/main/details.md).
