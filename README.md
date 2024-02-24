# mslice - A simple application level routing tool

## Overview

This tool is intended for application level IP traffic routing for
multi-home environment such as 5G UE with multiple slices.

For example, in case you are using a 5GS stack such as Free5GC + UERANSIM
and configured multiple slices environment. You would see multiple
GTP-U tunnel interfaces along with the slices on the UE side like
the following example.

* ens3
    * 192.168.1.1/24 (default route. Used for Radio Link Emulation traffic)
* uesimtun0
    * 10.1.1.1/32  (from tac#1/slice#2 pool - 10.1.1.0/24)
* uesimtun1
    * 10.1.2.1/32  (from tac#1/slice#2 pool - 10.1.2.0/24)

In this case, an example usecase for the multiple slices is to use
one slice (e.g., slice#1 through uesimtun0) for latency sensitive applications
and another slice (e.g., slice#2 through uesimtun1) for bandwidth sensitive
applications.

One typical resolution is to use routing function such as statie route.
Indeed the static route resolution works fine, but it's inconvenient
in several situations. For example, when:

1. You don't have root privilege to configure static routes
2. destination is given FQDN and the IP addresses change dynamically
3. etc.

Alternatively, you can modify your application, but it's also not always
possible. Or maybe your application have a feature specifying
local IP address (or corresponding interfaces) like 'ping -I' or 'ssh -B'.

This tool can be used for those situations in generic way.


## How it Works

mslice uses a classical technique to hook function calls at binary level
using LD_PRELOAD environment variable.

mslice inserts a hoook for connect(2) system call and checks, and
if the destination IP address belongs to the specified target network address.
If it is, mslice calls bind(2) using the specified local binding address, and
finally calles the original connect(2).

## Usage

Suppose you want to use multiple tunnels like below:
* destination1: 192.168.100.0/24, tun0 w/local address: 10.1.1.1
* destination2: 192.168.200.0/24, tun1 w/local address: 10.1.2.1

You can specify the mappings of desitination CIDR and local bind addresses
using MSLICE_TARGETS environment variable like below.

Use ',' (comma) for destination CIDR local bind address separator, and
';' (semi colon) for multiple pairs separator of the above format.

```
 $ env MSLICE_TARGETS="192.168.100.0/24,10.1.1.1;192.168.200.0/24,10.1.2.1" \
       LD_PRELOAD=./libmslice.so SOME_PROGRAM (e.g., curl)
```

For debug usage, set any value to MSLICE_DEBUG environment variable.

## Restrictions/TODO
* Support IPv6
* Enable to use interface name (e.g., uesimtun0)
* Enable to use FQDN, not IP address

## License

Apache License 2.0

## Notes

* LD_PRELOAD
  * https://man7.org/linux/man-pages/man8/ld.so.8.html
* Similar works
  * There are lots of similar works. For example, UERANSIM bundles a similar
tool called 'binder'.
    * https://github.com/aligungr/UERANSIM
    * The main difference with this tool is that UERANSIM binder allows
    only 1 local (tunnel device) bind address.
