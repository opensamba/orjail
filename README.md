# torjail
bind a program inside a network namespace that can only exit through tor

### requirements
you'll need a running tor instance with following configuration:
```
VirtualAddrNetwork 10.200.1.1/10
AutomapHostsSuffixes .onion,.exit
AutomapHostsOnResolve 1
TransPort 9040
TransListenAddress 10.200.1.1
DNSPort 5353
DNSListenAddress 10.200.1.1
```

### why?
we've tried to deanonimize a program executed in torsocks environemnt and that was not so difficult, as torsocks use LD_PRELOAD so you only need to statically compile your stuff.
as whonix is too much, the idea is to experiment with linux namespaces and learn by doing something usefull (at least for us).


### how it works
it creates a separated network namespace (using `ip netns`) with its own network
interface and a bridge to the host interface with some iptables rules (on host)
that force traffic generated from inside torjail to only exit via tor.
inside torjail you'll be in another pid namespace (this way you cannot switch
namespace), and another mount namespace (we use this to show a different /etc/resolv.conf).
if you find a way to deanonimize torjail, write us!

### usage examples:

- the help menu:
`torjail -h`

- get an homepage content via tor, executing curl as user $USER
`sudo torjail -u $USER curl autistici.org > autistici.org `

- resolve a onion address:
`sudo torjail dig wi7qkxyrdpu5cmvr.onion`

- get an onion webserver content via tor:
`sudo torjail curl wi7qkxyrdpu5cmvr.onion`

- open a firefox with user $USER that could only reach internet via tor:
`./torjail -u $USER firefox -P /tmp/tmpprofile`

> warning: firefox has a flag that block .onion resolution by default -> change it using about:config network.dns.blockDotOnion

- get a shell that could not reach internet (only via tor)
`./torjail -s -u $USER`

- custom network configuration:
`./torjail -r 10.10.1.1 10.10.1.2 24 ls`

- create a namespace called $NS in verbose mode:
`./torjail -n $NS -v ls`

- keep the existing /etc/resolv.conf after the execution:
`./torjail -k ls`

> in order to support the correct routing, /etc/resolv.conf is mounted to a temporary file inside the namespace, generated by torjail.
