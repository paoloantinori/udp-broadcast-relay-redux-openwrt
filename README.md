# udp-broadcast-relay-redux (OpenWrt packaging)

OpenWrt package for [udp-broadcast-relay-redux](https://github.com/udp-redux/udp-broadcast-relay-redux)
(GPL-2.0, C): relays UDP **broadcast and multicast** packets between
interfaces, **preserving the original source address** via raw sockets, so
replies reach the real sender. The classic use case: discovery protocols
(SSDP-like, vendor LAN protocols) that use broadcast/multicast and cannot
cross a router or a VLAN boundary.

The upstream repo is archived ("works fine, feel free to fork"): this repo
vendors the source (`net/udp-broadcast-relay-redux/src/main.c`) unchanged
and adds the OpenWrt packaging layer (UCI config + procd init + feed Makefile
+ CI builds).

## Install

Grab the `.apk` (25.12+) or `.ipk` (24.10) for your target from
[Releases](../../releases), then on the router:

```sh
apk add ./udp-broadcast-relay-redux-*.apk   # 25.12.x
# or: opkg install ./udp-broadcast-relay-redux-*.ipk   # 24.10.x
```

## Configure

`/etc/config/udp-broadcast-relay-redux`, one section per instance:

```conf
config relay
	option id '1'            # unique per instance (echo detection)
	list port '4002'         # broadcast ports to relay
	list dev 'br-lan.1'      # first interface
	list dev 'br-lan.21'     # second interface (VLAN)
	list multicast '239.255.255.250'   # optional multicast groups
```

```sh
uci commit udp-broadcast-relay-redux
service udp-broadcast-relay-redux enable
service udp-broadcast-relay-redux start
```

## Example: Govee LAN discovery across a VLAN

Govee devices with LAN control discover via **multicast** `239.255.255.250:4001`
(replies fixed to the querier's source port) and push on UDP `4002`/`4003`
broadcasts. The daemon takes ONE port per instance, so the full working setup
(Home Assistant on the main VLAN, devices on an IoT VLAN) is three sections:

```conf
config relay 'push'
	option id '1'
	list port '4002'
	list dev 'br-lan.1'
	list dev 'br-lan.21'

config relay 'scan'
	option id '2'
	list port '4003'
	list dev 'br-lan.1'
	list dev 'br-lan.21'

config relay 'mc'
	option id '3'
	list port '4001'
	list dev 'br-lan.1'
	list dev 'br-lan.21'
	list multicast '239.255.255.250'
```

The relay is transparent: devices see the real querier IP and reply directly
(you still need your firewall to allow the reply unicast flow between zones,
and to accept the relayed traffic as INPUT if the zone policy is closed).

### Verify it works

From a host on the first VLAN, send a UDP packet to the multicast group and
capture on the second VLAN; the reinjected copy must carry the ORIGINAL
source address:

```sh
python3 -c "import socket;s=socket.socket(socket.AF_INET,socket.SOCK_DGRAM);s.setsockopt(socket.SOL_IP,socket.IP_MULTICAST_TTL,3);s.sendto(b'test',('239.255.255.250',4001))"
# on the router: tcpdump -i br-lan.21 -nn udp port 4001
```

This exact setup runs in production relaying Govee discovery between VLANs
on OpenWrt 25.12 (bcm2711).

## Targets built by CI

24.10 and 25.12 and snapshot, for `x86/64`, `bcm27xx/bcm2711` (RPi4),
`mediatek/filogic`, `mediatek/mt7622`, `ramips/mt7621`. More on request
(one line in the workflow matrix).

## Building yourself

Feed-style package: copy `package/udp-broadcast-relay-redux` into your
OpenWrt buildroot or SDK `package/` dir and
`make package/udp-broadcast-relay-redux/compile`.

## License

GPL-2.0 (upstream tool, see LICENSE and CONTRIBUTORS.md in this repo).
