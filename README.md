# Brief introduction to IPv6

For people having a basic knowledge about how IPv4 works.

## The basics

- IPv6 recovers end-to-end communication.
  There is no NAT and CG-NAT.
  Each device can have a public address in the Internet if you wish.
- IPv6 was designed for automatic configuration.
  Even the dumbest device can connect to an IPv6 network without human
  intervention, but manual configuration is also allowed.
  Note that automatic configuration is implemented in the network layer.
  The data link layer may need manual configuration
  (WiFi networks, for instance).
- The IPv6 protocol was designed to work over fibre optics as a data link
  protocol as well as a network protocol, using the IPv6 stack alone.
  A network in this shape was called "Internet 2".
- Routing is hierarchical in IPv6.
- IPv6 addresses are 128 bit long (8 bytes).
  There is a new standard notation in hexadecimal composed by
  4 groups of 4 bytes separated by ":".
  However, a single sequence of continuous zeros can be summarized to "::".
  For example "2001:db8::1" equals to "2001:db80:0000:0001".
- Network mask are replaced by CIDR prefixes.
  For example, a /24 prefix means the leftmost 24 bits of the address is
  a network address. The rightmost 64 bits of the address is the node address.
  The other 40 bits can spread over several uses:
  - as a subnet address for routing (by increasing the prefix size)
  - added to the node address
  - set to zero.
- Each network interface can have two or more IPv6 addresses and
  usually does.
- IPv4 broadcast, multicast and anycast addresses are replaced by
  IPv6 multicast addresses, which work in a different way.

## Motivation

- IPv4 never was designed as a world-wide network.
  There are not enough IPv4 addresses for all nodes in the Internet.
- There is a "black market" selling IPv4 addresses 10-15 $ per unit.
- Several workarounds allowed to cope with the lack of IPv4 addresses:
  - DHCP: we hope not all workstations try to connect at the same time.
  - NAT (Network Address Translation) and CG-NAT (courier-grade NAT).
  - CIDR (variable-length address prefixes)
- On the other side, IPv6:
  - Uses hierarchical routing by extending CIDR.
  - IPSec is mandatory.
  - ICMP has been revisited to allow automatic configuration,
    network discovery and multicast.

## ICMPv6

Designed for protection against DDOS and packet fragmentation attacks.

- Packet fragmentation:

  - Takes place in the sender's node. No router is involved.
  - The original packet is rebuild in the receiver's node.
  - At first, the sender uses it's link MTU
  - If some node in the path does not support that MTU,
    it sends back an ICMP packet to the sender,
    telling the supported MTU.
    Then, the sender creates fragments accordingly and retry.

- Network control: as ICMPv4
- Network discovery:

  - Each node is able to discover other nodes in the neighborhood.
  - Each node is able to discover routers in the neighborhood.
  - MTU is properly assigned for the link.
  - A redirection packet can ask a node to use another router.

## IPv6 addressing

- Address types:
  - Unspecified.
  - Loopback.
  - Link-local:
    - Mandatory.
    - Unique in a local network.
    - Not routed.
    - Auto-configured (random or MAC-based).
  - Unique-local
    - A way to avoid address collision between two different organizations.
      Makes organization merges easier.
    - Not routed in the Internet.
    - Routed in the corporate network, only.
  - Multicast.
  - Global:
    - Routed in the Internet.
    - Uses a 48 bit prefix provided by an ISP or RIR.
    - Another 16 bits are available as a subnet address (optional).
    - A minimum of 64 bits are available for the node address suffix.
    - Some global addresses are reserved.

Note: "site-local" and "IPv4-compatible" addresses are no longer available.

| Address type | Prefix    | Notes                        |
| ------------ | --------- | ---------------------------- |
| Unspecified  | ::/128    | Can not be assigned to nodes |
| Loopback     | ::1/128   | Implicit                     |
| Multicast    | FF00::/8  | Can not be assigned to nodes |
| Link-local   | FE80::/10 | Not routed                   |
| Unique local | FC00::/7  | Not routed in the Internet   |
| Global       | the rest  | Up to /64                    |

Communication in a local network **always** takes place
using link-local addresses *exclusively*,
thus no router is involved in this scenario.
As a result:

- A router is not required for IPv6 communication in a local network.
- Every network interface (in the Internet) has two IPv6 addresses at least:
  link-local and global.

### Reserved global addresses

| Purpose       | Prefix        | Notes                         |
| ------------- | ------------- | ----------------------------- |
| 6to4          | 2002::/16     | For IPv4 coexistence          |
| Teredo        | 2001::/32     | Reserved by IANA              |
| BMWG          | 2001:2::/48   | For benchmarking              |
| ORCHID        | 2001:10::/28  | Not routed (hash identifiers) |
| Documentation | 2001:db8::/32 | Can not be assigned to nodes  |

When writing examples a "documentation address" should be used.
This way, mere examples without context can not be confused with
real IPv6 addresses in use.

### Multicast addresses

This is just a summary.

| Target                                       | Address   |
| -------------------------------------------- | --------- |
| Every node in the local network segment      | FF02::1   |
| Every router in the local network segment    | FF02::1:2 |
| All DHCP servers and relays in a site        | FF05::1:3 |
| Simple Service Discovery Protocol (see note) | FF0x::C   |
| Multicast DNS (see note)                     | FF0x::FB  |

> Note:
>
> x (an hexadecimal digit) equals to 2,5,8 or E depending on scope:
> 2=link local, E=Internet.

### Special address identifiers

- **URL**: place the IPv6 address between brackets.
- **UNC paths**: Substitute the ":" character with "-",
  then add the ".ipv6-literal.net" suffix.
  This is a fake DNS entry automatically resolved by Microsoft Windows locally.

## Link-local address auto-configuration

A link-local address is computed via one of two algorithms:
EUI-64 or "privacy extensions" (RFC 4941),
both based on the MAC address.
Note that link-local addresses are guaranteed a 64 bit suffix
coinciding with MAC address lengths.
The link-local address can always be computed
without human or network intervention.

Any possible address collision is solved thanks to simple ICMPv6 messages.

## Global address auto-configuration

Routers announce themselves in the local network periodically,
thanks to the "router announce" ICMPv6 multicast packet,
but any node can ask for a router announcement via a "router request"
ICMPv6 multicast packet.

- The "router request" contains:
  - The link-local source address.
  - The "FF02::2" multicast destination address (all routers).

- The "router announce" contains:
  - The router link-local source address.
  - The "FF02::1" multicast destination address (all nodes).
  - Subnet prefix
  - Expiration information (if applies)
  - "M" and "O" flags (see below).
  - Router MAC address for caching purposes.

The global address auto-configuration method
depends on the "router announce" flags:

- O = 0, M = 0 (SLAAC method):
  - IPv6 address assigned by the router.
  - DNS servers assigned by hand (not automatic).
- O = 1, M = 0 (Stateless DHCPv6 method):
  - IPv6 address assigned by the router.
  - DNS servers assigned via DHCPv6.
- O = 0, M = 1 (Stateful DHCPv6 method):
  - IPv6 address assigned via DHCPv6.
  - DNS servers assigned via DHCPv6.
- O = 1, M = 1 (**stupid method never used**)
  - IPv6 address assigned via DHCPv6.
  - DNS servers assigned by hand (not automatic).

When M = 0, the node computes its own global address
using the subnet prefix and its own MAC address in a similar way
to the link-local address.
However, in this case, auto-configuration
works in /64 prefixed networks **only**.

### DHCPv6

DHCPv6 servers are discovered using ICMPv6 multicast messages,
in a similar way to routers.
DHCPv6 clients listen to the 546 UDP port.
DHCPv6 servers and relays listen to the 547 UDP port.

When using the stateless method, the DHCP server does not keep track of
previously assigned global addresses.

DHCPv6 does not uniquely identify nodes via MAC addresses as DHCPv4 does.
Instead, it uses a pair of computed identifiers:

- DUID: a device identifier
- IAID: a network interface identifier
  (a device can handle several network interfaces at the same time).

There are four distinct algorithms to compute a DUID, but all of them
generate an unique identifier.

## IPv6 adoption

There are several sources:

- [https://6lab.cisco.com/](https://6lab.cisco.com/)
- [https://labs.apnic.net/](https://labs.apnic.net/)
- [https://ipv6observatory.org/](https://ipv6observatory.org/) (Europe-only)
- Count of AAAA entries in DNS servers
- Count of "IPv6 ready" certificates
- Akamai reports
- RIPE stars

## How to obtain an IPv6 address range in the Internet

- From your ISP.
  If you change to another ISP, your address range changes as well.

- Make your organization a LIRC. Costly.

- Ask RIPE for a /48 range, then ask your ISP to route it.

## Is IPv6 more secure than IPv4 ?

Not by design:

- You have the same level of security when using IPv6 addresses.
  The scan range is wider, but more predictable.
- IPSEC is not the ultimate security feature:
  - Stays in the way of IDS tools and policy enforcers.
  - It was designed only to traverse unsecure networks.
  - Not configured by default.
- An installed, but not configured, IPV6 stack poses a security risky.
- Network security is provided via firewalls, as in IPv4.
- Protecting final nodes (workstations) is more relevant in IPv6 than IPv4.
- DHCPv6 and DNS server protection is critical.

## DNS

There is not such a thing as "DNSv6" because the old DNS protocol works
on any transport layer (IPv6 stack, IPv4 stack or both).

As a distributed database, DNS holds information about IPv6 (AAAA records)
and IPv4 (A records) independently of the network stack.
As a result, you may ask for IPv4 addresses via IPv6 transport or
ask for IPv6 addresses via IPv4 transport.

## IP v6/v4 coexistence

- Double stack:
  - Easy.
  - Widely adopted.
    All popular operating systems have double stack support.
  - Double the management effort.
  - Double the security risk.

- Tunnels:
  - Good as a temporary remedy, but not for long-term.
  - There are 3 kinds:
    - Manual.
    - Semi-automatic (brokers).
    - Automatic.

- NAT:
  - Last resort.
  - May work at OSI level 3 or level 4.

Check your IPv6 connectivity at [test-ipv6.com](https://test-ipv6.com)

### IPv6 over IPv4 via Teredo

The Teredo protocol allows IPv6 communication via an IPv4 tunnel
against a single server.

Microsoft provides free Teredo servers for all Windows and Xbox machines,
but may turn them off at any time because Teredo is a transition technology.

There are some requirements:

- UDP port 3544 traffic must be allowed to pass firewalls and routers.
- UDP port 1900 traffic must be allowed in the workstation local firewall unless
  UPNP is configured in the router.
- The workstation must be able to solve the domain name
  "teredo.ipv6.microsoft.com".

The Teredo protocol automatically detects NAT and tries to open
router ports via UPNP. If UPNP is not available, the user must configure the
router manually to allow Teredo traffic.

#### Teredo in Windows machines

Teredo can stablish a TCP connection to an specific IPv6 address.
It does not allow WWW navigation.

Teredo is not activated by default in machines logged to an
Active Directory domain, even when properly configured.
To activate it, there are two ways:

1. Choose an "enterprise client" role:

   > netsh interface ipv6 set teredo enterpriseclient
   > *«server»* *«clientport»*

2. Edit the proper configuration directive via `gspedit.msc`.

If the workstation uses an IPv4 public address,
the Teredo tunnel is automatically created. If you don't like this,
use the Windows registry to disable it.
Check
[https://appuals.com/microsoft-teredo-tunneling-adapter/](https://appuals.com/microsoft-teredo-tunneling-adapter/)
for further instructions.

Some Teredo issues may be solved using the "game configuration" in Windows 10:
"Configuration > games > Xbox network" then "troubleshoot".

There is a default Teredo server configured in Windows,
but it is not active anymore.
There are other free Teredo servers you can use. Search the Internet.
For example: teredo.trex.fi

Run the command:

> netsh interface ipv6 set teredo enterpriseclient
> *«type»*  *«server»* *«clientport»*

- *type*: see below.
- *server*: DNS name of the Teredo server.
- *clientport*: an open port number in your NAT.

A system reset may be required.

To check connectivity, run:

> ping -6 google.com

To disable the tunnel, run:

> netsh interface ipv6 set teredo disable

##### The Teredo network adapter

This is a virtual network adapter already installed in Windows 10 and later.
Previous versions require manual installation of the
"Microsoft Teredo Tunneling Adapter" from Microsoft,
in the "Legacy hardware" group.

##### Tunnel status

Execute:

> netsh interface ipv6 show teredo
>

And check the "type" information:

- disabled: no service
- client: enabled as client
- enterpriseclient: enabled as client
  by omitting the managed network detection.
  Allows tunnel establishment even when the workstation is not joined
  to a domain.
- natawareclient: enabled as client by identifying NAT type.
- server: enabled as server
- default: same as "client".

Then check the client port:

- unspecified: a port is automatically chosen by the operating system.

Then check the connection status:

- offline: no server response. Check the "error" field.
- dormant: server available but inactive.
- qualified: ser available and connected.
