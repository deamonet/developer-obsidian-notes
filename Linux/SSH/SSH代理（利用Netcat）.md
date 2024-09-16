[Jonathan Perkin](https://www.perkin.org.uk/)

[about me](https://www.perkin.org.uk/about.html) · [rss](https://www.perkin.org.uk/rss.xml) · [twitter](https://twitter.com/jperkin) · [github](https://github.com/jperkin)

# SSH via HTTP proxy in OSX

Apr 20, 2011

tags: [netcat](https://www.perkin.org.uk/tags/netcat.html), [osx](https://www.perkin.org.uk/tags/osx.html), [ssh](https://www.perkin.org.uk/tags/ssh.html)

If you happen to be stuck behind a corporate firewall with only HTTP proxies for external access, you might still be able to SSH out through them using the built-in `nc` on OSX.

First, hope that the proxies haven’t disabled the `CONNECT` method, then simply add a section to your `.ssh/config` like this:

```text
Host foobar.example.com
    ProxyCommand          nc -X connect -x proxyhost:proxyport %h %p
    ServerAliveInterval   10
```

This will tunnel the connection through the HTTP proxy to the remote server. The `ServerAliveInterval` setting is required as most proxies will drop the connection after a period of inactivity.

To avoid issues with trying to connect to the host when not behind the corporate firewall, replace the above with a fake entry for the proxy method like this:

```text
Host foobar-proxy.example.com
    HostName              foobar.example.com
    ProxyCommand          nc -X connect -x proxyhost:proxyport %h %p
    ServerAliveInterval   10
```

Then use

```console
$ ssh foobar-proxy.example.com
```

when inside the firewall, and

```console
$ ssh foobar.example.com
```

when outside.

Simples, no external software required.

Share this post on [Twitter](https://twitter.com/share?url=//www.perkin.org.uk/posts/ssh-via-http-proxy-in-osx.html&text=%22SSH+via+HTTP+proxy+in+OSX%22+by+%40jperkin), [HackerNews](https://news.ycombinator.com/submitlink?u=//www.perkin.org.uk/posts/ssh-via-http-proxy-in-osx.html&t=SSH+via+HTTP+proxy+in+OSX), [Facebook](https://www.facebook.com/sharer/sharer.php?u=//www.perkin.org.uk/posts/ssh-via-http-proxy-in-osx.html&t=SSH+via+HTTP+proxy+in+OSX) or [Google+](https://plus.google.com/share?url=//www.perkin.org.uk/posts/ssh-via-http-proxy-in-osx.html)

---

# All Posts

- 16 Jul 2015 » [Reducing RAM usage in pkgin](https://www.perkin.org.uk/posts/reducing-ram-usage-in-pkgin.html)
- 03 Mar 2015 » [pkgsrc-2014Q4: LTS, signed packages, and more](https://www.perkin.org.uk/posts/pkgsrc-2014Q4-lts-signed-packages-and-more.html)
- 06 Oct 2014 » [Building packages at scale](https://www.perkin.org.uk/posts/building-packages-at-scale.html)
- 04 Dec 2013 » [A node.js-powered 8-bit CPU - part four](https://www.perkin.org.uk/posts/a-nodejs-powered-8-bit-cpu-part-four.html)
- 03 Dec 2013 » [A node.js-powered 8-bit CPU - part three](https://www.perkin.org.uk/posts/a-nodejs-powered-8-bit-cpu-part-three.html)
- 02 Dec 2013 » [A node.js-powered 8-bit CPU - part two](https://www.perkin.org.uk/posts/a-nodejs-powered-8-bit-cpu-part-two.html)
- 01 Dec 2013 » [A node.js-powered 8-bit CPU - part one](https://www.perkin.org.uk/posts/a-nodejs-powered-8-bit-cpu-part-one.html)
- 21 Nov 2013 » [MDB support for Go](https://www.perkin.org.uk/posts/mdb-support-for-go.html)
- 30 Jul 2013 » [What's new in pkgsrc-2013Q2](https://www.perkin.org.uk/posts/whats-new-in-pkgsrc-2013Q2.html)
- 24 Jul 2013 » [Distributed chrooted pkgsrc bulk builds](https://www.perkin.org.uk/posts/distributed-chrooted-pkgsrc-bulk-builds.html)
- 07 Jun 2013 » [pkgsrc on SmartOS - creating new packages](https://www.perkin.org.uk/posts/pkgsrc-on-smartos-creating-new-packages.html)
- 15 Apr 2013 » [What's new in pkgsrc-2013Q1](https://www.perkin.org.uk/posts/whats-new-in-pkgsrc-2013Q1.html)
- 19 Mar 2013 » [Installing SVR4 packages on SmartOS](https://www.perkin.org.uk/posts/installing-svr4-packages-on-smartos.html)
- 27 Feb 2013 » [SmartOS is Not GNU/Linux](https://www.perkin.org.uk/posts/smartos-is-not-gnu-linux.html)
- 18 Feb 2013 » [SmartOS development preview dataset](https://www.perkin.org.uk/posts/smartos-development-preview-dataset.html)
- 17 Jan 2013 » [pkgsrc on SmartOS - fixing broken builds](https://www.perkin.org.uk/posts/pkgsrc-on-smartos-fixing-broken-builds.html)
- 15 Jan 2013 » [pkgsrc on SmartOS - zone creation and basic builds](https://www.perkin.org.uk/posts/pkgsrc-on-smartos-zone-creation-and-basic-builds.html)
- 10 Jan 2013 » [Multi-architecture package support in SmartOS](https://www.perkin.org.uk/posts/multiarch-package-support-in-smartos.html)
- 09 Jan 2013 » [Solaris portability - cfmakeraw()](https://www.perkin.org.uk/posts/solaris-portability-cfmakeraw.html)
- 08 Jan 2013 » [Solaris portability - flock()](https://www.perkin.org.uk/posts/solaris-portability-flock.html)
- 06 Jan 2013 » [pkgsrc-2012Q4 illumos packages now available](https://www.perkin.org.uk/posts/pkgsrc-2012Q4-packages-for-illumos.html)
- 23 Nov 2012 » [SmartOS and the global zone](https://www.perkin.org.uk/posts/smartos-and-the-global-zone.html)
- 24 Oct 2012 » [Setting up Samba on SmartOS](https://www.perkin.org.uk/posts/setting-up-samba-on-smartos.html)
- 10 Oct 2012 » [pkgsrc-2012Q3 packages for illumos](https://www.perkin.org.uk/posts/pkgsrc-2012Q3-packages-for-illumos.html)
- 23 Aug 2012 » [Creating local SmartOS packages](https://www.perkin.org.uk/posts/creating-local-smartos-packages.html)
- 10 Jul 2012 » [7,000 binary packages for OSX Lion](https://www.perkin.org.uk/posts/7000-packages-for-osx-lion.html)
- 09 Jul 2012 » [9,000 packages for SmartOS and illumos](https://www.perkin.org.uk/posts/9000-packages-for-smartos-and-illumos.html)
- 07 May 2012 » [Goodbye Oracle, Hello Joyent!](https://www.perkin.org.uk/posts/goodbye-oracle-hello-joyent.html)
- 13 Apr 2012 » [SmartOS global zone tweaks](https://www.perkin.org.uk/posts/smartos-global-zone-tweaks.html)
- 12 Apr 2012 » [Automated VirtualBox SmartOS installs](https://www.perkin.org.uk/posts/automated-virtualbox-smartos-installs.html)
- 30 Mar 2012 » [iptables script for Debian / Ubuntu](https://www.perkin.org.uk/posts/iptables-script-for-debian-ubuntu.html)
- 20 Feb 2012 » [New site design](https://www.perkin.org.uk/posts/new-site-design.html)
- 11 Jan 2012 » [Set up anonymous FTP upload on Oracle Linux](https://www.perkin.org.uk/posts/set-up-anonymous-ftp-upload-on-oracle-linux.html)
- 09 Jan 2012 » [Kickstart Oracle Linux in VirtualBox](https://www.perkin.org.uk/posts/kickstart-oracle-linux-in-virtualbox.html)
- 09 Jan 2012 » [Kickstart Oracle Linux from Ubuntu](https://www.perkin.org.uk/posts/kickstart-oracle-linux-from-ubuntu.html)
- 22 Dec 2011 » [Last day at MySQL](https://www.perkin.org.uk/posts/last-day-at-mysql.html)
- 15 Dec 2011 » [Installing OpenBSD with softraid](https://www.perkin.org.uk/posts/installing-openbsd-with-softraid.html)
- 21 Sep 2011 » [Create VirtualBox VM from the command line](https://www.perkin.org.uk/posts/create-virtualbox-vm-from-the-command-line.html)
- 14 Sep 2011 » [Creating chroots for fun and MySQL testing](https://www.perkin.org.uk/posts/creating-chroots-for-fun-and-mysql-testing.html)
- 30 Jun 2011 » [Graphing memory usage during an MTR run](https://www.perkin.org.uk/posts/graphing-memory-usage-during-an-mtr-run.html)
- 29 Jun 2011 » [Fix input box keybindings in Firefox](https://www.perkin.org.uk/posts/fix-input-box-keybindings-in-firefox.html)
- 24 Jun 2011 » [How to lose weight](https://www.perkin.org.uk/posts/how-to-lose-weight.html)
- 23 Jun 2011 » [How to fix stdio buffering](https://www.perkin.org.uk/posts/how-to-fix-stdio-buffering.html)
- 13 Jun 2011 » [Serving multiple DNS search domains in IOS DHCP](https://www.perkin.org.uk/posts/serving-multiple-dns-search-domains-in-ios-dhcp.html)
- 13 Jun 2011 » [Fix Firefox URL double click behaviour](https://www.perkin.org.uk/posts/fix-firefox-url-double-click-behaviour.html)
- 20 Apr 2011 » [SSH via HTTP proxy in OSX](https://www.perkin.org.uk/posts/ssh-via-http-proxy-in-osx.html)
- 09 Nov 2010 » [How to build MySQL releases](https://www.perkin.org.uk/posts/how-to-build-mysql-releases.html)
- 29 Apr 2010 » ['apt-get' and 5,000 packages for Solaris10/x86](https://www.perkin.org.uk/posts/apt-get-and-5000-packages-for-solaris10x86.html)
- 16 Sep 2009 » [ZFS and NFS vs OSX](https://www.perkin.org.uk/posts/zfs-and-nfs-vs-osx.html)
- 12 Sep 2009 » [pkgsrc on Solaris](https://www.perkin.org.uk/posts/pkgsrc-on-solaris.html)
- 09 Dec 2008 » [Jumpstart from OSX](https://www.perkin.org.uk/posts/jumpstart-from-osx.html)
- 31 Dec 2007 » [Set up local caching DNS server on OSX 10.4](https://www.perkin.org.uk/posts/set-up-local-caching-dns-server-on-osx-104.html)