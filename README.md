# jvpn.pl - Juniper VPN Client

This fork is a workaround for running [jvpn](https://github.com/samm-git/jvpn) in
[crouton](https://github.com/dnschneid/crouton) on a chromebook ASUS C200.

> :warning: This is now obsolete as Pulse Secure has released a
> [VPN client for Chrome OS](https://chrome.google.com/webstore/detail/pulse-secure/eiddfaedmgnpnnojolcknhpjbmmpplgd)
> meanwhile.

## Explanation

### Original issue 1

On my chromebook, the original `jvpn` hits the following error:

```
Status=6e
Authentication failed, exiting
```

This can always be reproduced on this device but not on any of my other laptops. None of the other
workarounds I've found for this error helped (e.g. checking `ifconfig` and `route`'s location, or
`/dev/net/tun`'s type and permissions). Thus I've decided to investigate the issue by:

* writing `debug=1` in `jvpn.ini`, and
* letting `jvpn.pl` start `ncsvc` with `strace`.

It appears to be a [heisenbug](https://en.wikipedia.org/wiki/Heisenbug) because doing so suddenly
makes `jvpn` work. I bet we are dealing with a race condition here, although I haven't had time to
investigate this any further yet. Removing one of both modifications makes me hit this hypothetical
race condition again.

### Original issue 2

A more recent issue is that
[shill](https://www.chromium.org/chromium-os/chromiumos-design-docs/network-portal-detection)
prevents the `tun0` interface to be created.
[This solution](https://github.com/dnschneid/crouton/wiki/Using-Cisco-AnyConnect-VPN-with-openconnect#solution-to-make-the-tun0-interface-stable-and-the-openvpn-command-reliable)
had to be applied to `jvpn`.

## Installation

### Create a crouton chroot

> **NOTE:** if you're trying this workaround directly on GNU/Linux (not on Chrome OS), skip this
> step.

If you don't have one yet, create a [crouton](https://github.com/dnschneid/crouton) chroot

```
$ cd /tmp/
$ wget -c https://goo.gl/fd3zc
$ mv fd3zc crouton
$ sudo sh crouton -t core -n myVpnChroot
```

and enter it

```
$ sudo enter-chroot -n myVpnChroot
```

### Install jvpn

Make sure everything `jvpn` needs is there:

```
$ sudo apt-get install libterm-readkey-perl libio-socket-ssl-perl libhttp-message-perl libwww-perl unzip libdbus-glib-1-2:i386 psmisc
$ mkdir -p ~/.juniper_networks/network_connect
```

Now make sure what my workaround needs is there:

```
$ sudo apt-get install strace
```

Download `jvpn` anywhere:

```
$ git clone https://github.com/AurelienLourot/jvpn/
$ cd jvpn/
```

### Create jvpn.ini

Create a default `jvpn`'s configuration file `jvpn.ini`:

```
$ cp jvpn.sample.ini jvpn.ini
```

Edit it to fit your needs. In particular set the variables `host`, `url`, `username`, `realm` and
`password`.

**Do not** change the `debug` variable.

### Start jvpn

Here you go:

```
$ sudo perl jvpn.pl
```

> **NOTE:** as this is a workaround against a race condition, this is not guaranteed to work.
> Try again a few times if it doesn't.

----------------------------------------------------------------------------------------------------

[Original documentation](README)
