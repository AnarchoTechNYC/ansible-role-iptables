# AnarchoTechNYC: iptables [![Build Status](https://travis-ci.org/AnarchoTechNYC/ansible-role-iptables.svg?branch=master)](https://travis-ci.org/AnarchoTechNYC/ansible-role-iptables)

An [Ansible role](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) for configuring the [`iptables` Linux kernel packet filter](https://netfilter.org/) tables, chains, and rules. Notably, this role has been tested with [Raspbian](https://www.raspbian.org/) on [Raspberry Pi](https://www.raspberrypi.org/) hardware. This role's purpose is to make it possible to configure and apply arbitrary Internet Protocol packet filtering ("IP firewalling") rules.

# Configuring open ports

The default configuration of the kernel's packet filtering tables for the Internet Protocol version 4 and version 6 network stacks are defined by the `ipv4_tables` and `ipv6_tables` dictionaries, respectively. However, [those variables mimic the structure of the IP filtering tables themselves](#advanced-firewall-configuration), which is harder to remember and more cumbersome to work with.

Most of the time, users simply want a firewall that blocks every incoming packet, except for ones destined for a few chosen ports. For this reason, this role provides a convenience variable, `iptables_open_ports`, which is simply a list of rules that are appended to the `filter` table's `INPUT` chain. For example, to open port `5353` and `80` (for HTTP, aka. Web traffic):

```yml
iptables_open_ports:
  - destination_port: 5353 # Can use a port number,
  - destination_port: http # or a service name in `/etc/services`.
```

By default, this role blocks all incoming connections except for port `22`. If you want to open a second port, you must define it in the `iptables_open_ports` list. You can restrict a given 

You can restrict port you open to a given protocol:

```yml
iptables_open_ports:
  - protocol: udp
    destination_port: 5353
  - protocol: tcp
    destination_port: http
```

# Configuring closed ports

By default, this role blocks all incoming connections not matched by an explicitly-set open port (i.e., an `ACCEPT` rule). In effect, this functions as an IP filtering whitelist. However, you can [change the default policy of the firewall](#advanced-firewall-configuration) so that ports are open by default unless explicitly closed, effectively creating a blacklist.

If you choose to do that, you may still want to close certain ports, or you simply may want to proactively close ports explicitly. You can accomplish this through the `iptables_closed_ports` convenience variable. For example:

```yml
iptables_closed_ports:
  - destination_port: 11211
```

You can still choose the jump target (action) to take when receiving a packet on a closed port:

```yml
iptables_closed_ports:
  - destination_port: 11211
    target: REJECT           # Return a `RST` packet.
  - destination_port: telnet # Service names work here, too.
    target: DROP             # Silently drop the packet.
```

# Advanced firewall configuration

This role provides two dictionary variables that model the structure of the Linux kernel IP network packet filter for IPv4 and IPv6. These are `ipv4_tables` and `ipv6_tables`, respectively. The structure of the dictionaries is identical, except that one applies filtering rules to the IPv4 network stack while the other to the IPv6 network stack. The structure of the dictionary is as follows:

* `<table_name>`: Name of the kernel IP packet filtering table, e.g., `filter` or `nat`.
    * `<chain_name>`: Name of the rule chain in the table, e.g., `INPUT` or `PREROUTING`.
        * `policy`: Policy to set for this chain. See the *TARGETS* section in `iptables(8)` for valid policy targets.
        * `rules`: List ("chain") of IP packet filtering rules to apply.
            * `in_interface`: Filter the packet if it arrived on this interface. Set this to an interface device name such as `lo` or `eth0`.
            * `out_interface`: Filter the packet if it will be emitted on the given interface.
            * `protocol`: Filter the packet if it matches The named protocol. This can be one of `tcp`, `udp`, `udplite`, `icmp`, `icmpv6`, `esp`, `ah`, `sctp`, `mh`, or any other value recognized by the `--protocol` option to `iptables(8)`.
            * `source_addresses`: Filter the packet if its source address is in this list. The items in this list are dictionaries with the following keys:
                * `ip`: Source IP address to match.
                * `mask`: Network mask in classful or CIDR notation to match.
                * `not`: Boolean that inverts the source address match. For example:
                    ```yml
                    source_addresses:
                      - not: true
                        ip: 192.168.1.0
                        mask: 24
                    ```
                The above, in English: "match packets not originating from the 192.168.1.0/24 network."
            * `source_port`: Source port or port range in `[first]:[last]` to filter.
            * `destination_addresses`: List of destination IP addresses to filter. Same structure as for the `source_addresses` key.
            * `destination_port`: Destination port or port range in `[first]:[last]` to filter.
            * `target`: Action to take when the packet filtering rules match the given packet. Set this to the name of the chain to jump to when a packet matches the filtering rules. This corresponds to the `--jump` option of the `iptables(8)` command.
            * `module`: Name of an iptables extension matching module to load (see `iptables-extensions(8)` for details).
            * `module_options`: List of command-line flags to set for the module.

Some examples may prove illustrative:

1. Allow multicast DNS (mDNS) traffic so that `.local` names can be resolved by local Avahi daemons on IPv4:
    ```yml
    ipv4_tables:
      filter:
        INPUT:
          policy: DROP
          rules:
            - protocol: udp
              destination_port: 5353
              target: ACCEPT
    ```
1. :construction: TODO: More examples.

See the comments in the [`defaults/main.yml`](defaults/main.yml) file for further details.
