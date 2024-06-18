---
layout: home
title: Contributing
nav_order: 10
nav_titles: true
titles_max_depth: 2
---

## Many ways to contribute

There are many ways to contribute to MPTCP in general:
- [Reporting bugs](https://github.com/multipath-tcp/mptcp_net-next/issues)
- [Testing the development version](https://github.com/multipath-tcp/mptcp-upstream-virtme-docker)
- Talking about it on social media, mailing lists, conferences, etc.
- Sending supportive messages to other contributors
- Helping new users on different platforms: [mailing lists, IRC, external
  services](/#communication)
- Asking to enable it by default on services (servers) and devices
- Adding [native MPTCP support in apps](apps.html) (or asking app developers to
  add native MPTCP support)
- Improving this [website](https://github.com/multipath-tcp/mptcp.dev)
- Improving [`mptcpd`](https://github.com/multipath-tcp/mptcpd), and tools used
  to configure MPTCP, e.g.
  [IPRoute2](https://wiki.linuxfoundation.org/networking/iproute2),
  [NetworkManager](https://networkmanager.dev/), etc.
- Sponsoring the maintainers (e.g. on [GitHub for Matt](https://github.com/sponsors/matttbe))
- etc.

And of course, contributing to the Linux kernel itself, see below.

## Kernel development

This part is for kernel developers: modifying MPTCP in the Upstream Linux
kernel. Patches with new features and bug fixes are of course welcome.

### Testing the modifications

It is easy to build the kernel, the dependences, and start a VM thanks to
[MPTCP Upstream Virtme Docker](https://github.com/multipath-tcp/mptcp-upstream-virtme-docker).
With just one command, you can get a prompt from a VM using the modified kernel
or run the whole test suite. For more details, please see this
[page](https://github.com/multipath-tcp/mptcp_net-next/wiki/CI) from our wiki.


{: .note}
Any push to a fork of the [`mptcp_net-next`](https://github.com/multipath-tcp/mptcp_net-next)
repository on GitHub will trigger the CI: look at the `Actions` tab in the fork
repository to follow the progress, and show the results.

### Patches are shared on the mailing list

#### Why?

Like many kernel subsystems, patches can be sent to the assigned mailing list
([mptcp@lists.linux.dev](mailto:mptcp@lists.linux.dev)), and **not** via Pull
Requests on GitHub. The idea is not to restrict access to people who have a
GitHub account, and to have a decentralized system.

#### It is easy with `b4`

Sending patches by email is **not difficult**, no more than setting up a
GitHub account, with the verification of the email address, a new SSH key, etc.

To help with this task, we recommend using
[b4](https://b4.docs.kernel.org/en/latest/contributor/overview.html). After
having [installed](https://b4.docs.kernel.org/en/latest/installing.html) it, you
need to either:
- configure the
[web-endpoint](https://b4.docs.kernel.org/en/latest/contributor/send.html)
(launching 3 commands)
- or set info about your [SMTP server](https://git-send-email.io/#step-2) in the
  git config file

#### Workflow

The workflow is then easy: use
[`b4 prep`](https://b4.docs.kernel.org/en/latest/contributor/prep.html) to
create a new branch, and
[`b4 send`](https://b4.docs.kernel.org/en/latest/contributor/send.html) to send
your patches by email.

In short, once `b4` has been installed and configured, the workflow looks like:

```bash
git switch export # or export-net for fixes
b4 prep -n <new branch name>
# edit your code
git add -p && git commit -s
./scripts/checkpatch.pl -g HEAD ## check the output, fix issues with 'git commit --amend'
b4 prep --edit-cover # if you have more than one patch
b4 send --reflect # check if the mails you sent to yourself are OK
b4 send
```

Once sent, the patches will be tested by our CI. A report will then be posted on
[Patchwork](https://patchwork.kernel.org/project/mptcp/list/?state=*). See the
[CI](https://github.com/multipath-tcp/mptcp_net-next/wiki/CI) page on our wiki
for more details.

### Commit messages

Please **always** include the reason why a patch is needed: we can easily list
the modifications by looking at the patch itself (`diff`), but understanding why
a modification has been done, why it has been done this way and not another one,
etc. is important context for reviewers and future maintenance. This information
should appear in the commit message.

For more details, please check the [kernel
doc](https://www.kernel.org/doc/html/latest/process/submitting-patches.html)
page.

Do not hesitate to look at the Git history to get some examples.

### Code style

It is the same as the guidance for Netdev, see their
[FAQ](https://www.kernel.org/doc/html/latest/process/maintainer-netdev.html) for
more details.

#### tl;dr

* Designate your patch for a specific tree - `[PATCH mptcp-net]` or `[PATCH mptcp-next]`,
  see the [Patch prefixes](https://github.com/multipath-tcp/mptcp_net-next/wiki/Patch-prefixes)
  page for more details.
* For fixes, the `Fixes:` tag is required, regardless of the tree.
* Don't post large series (> 15 patches), break them up.
* Don't repost your patches within one 24h period.
* [Reverse Xmas tree](https://www.kernel.org/doc/html/latest/process/maintainer-netdev.html#local-variable-ordering-reverse-xmas-tree-rcs)
  for the variable declaration (from longest to shortest lines).
* Use [`checkpatch.pl`](https://www.kernel.org/doc/html/latest/dev-tools/checkpatch.html),
  e.g. `./scripts/checkpatch.pl -g HEAD`
* Use [`shellcheck`](https://www.shellcheck.net) for the Shell scripts.
* Keep the same style as the surrounding code.
