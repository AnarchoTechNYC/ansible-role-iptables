# AnarchoTechNYC: iptables [![Build Status](https://travis-ci.org/AnarchoTechNYC/ansible-role-iptables.svg?branch=master)](https://travis-ci.org/AnarchoTechNYC/ansible-role-iptables)

An [Ansible role](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) for configuring the [`iptables` Linux kernel packet filter](https://netfilter.org/) tables, chains, and rules. Notably, this role has been tested with [Raspbian](https://www.raspbian.org/) on [Raspberry Pi](https://www.raspberrypi.org/) hardware. This role's purpose is to make it possible to configure and apply arbitrary Internet Protocol packet firewall rules.

# Configuring the firewall

This role provides two dictionary variables that model the structure of the Linux kernel IP network packet filter for IPv4 and IPv6. These are `ipv4_tables` and `ipv6_tables`, respectively. The structure of the dictionaries is identical, except that one applies filtering rules to the IPv4 network stack while the other to the IPv6 network stack. The structure of the dictionary is as follows:

* `<table_name>`: The name of the table.
    * `<chain_name>`: The name of the chain.
        * `policy`: The policy to set for the chain.
        * `rules`: List of IP packet filtering rules to apply.
            * `in_interface`: The inbound interface to filter.
            * `out_interface`: The outbound interface to filter.
            * `protocol`: The protocol carried by the IP packet to filter.
            * `source_addresses`: List of source IP addresses to filter.
                * `ip`: IP address in the source field to filter.
                * `mask`: Network mask in classful or CIDR notation.
                * `not`: Boolean that inverts the source address match.
            * `source_port`: Source port or port range in `[first]:[last]` to filter.
            * `destination_addresses`: List of destination IP addresses to filter. Same structure as for the `source_addresses` key.
            * `destination_port`: Destination port or port range in `[first]:[last]` to filter.
            * `target`: The name of the chain to jump to when a packet matches the filtering rules.
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

See the comments in the [`defaults/main.yml`](defaults/main.yml) file for details.
