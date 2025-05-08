Ansible role: Kea
=================

Install and configure Kea DHCP server.

Role Variables
--------------

```
ENTRY POINT: *main* - Install and configure Kea DHCP server

Options (= indicates it is required):

- kea_apt_repo_component  Component to use for the apt repository
          default: main
          type: str

- kea_apt_repo_gpg_key  Either a URL to a GPG key, absolute path to a keyring file, one or
                         more fingerprints of keys either in the
                         trusted.gpg keyring or in the keyrings in the
                         trusted.gpg.d/ directory, or an ASCII armored
                         GPG public key block
          default: https://dl.cloudsmith.io/public/isc/kea-{{ kea_apt_repo_version }}/gpg.0D9D9A1439E23DB9.key
          type: str

- kea_apt_repo_suite  Suite to use for the apt repository
          default: '{{ ansible_distribution_release }}'
          type: str

- kea_apt_repo_url  Base URL for the apt repository
          default: https://dl.cloudsmith.io/public/isc/kea-{{ kea_apt_repo_version }}/deb/{{
            ansible_distribution | lower }}
          type: str

- kea_apt_repo_version  Repository version to track, for available versions see:
                         https://cloudsmith.io/~isc/repos/
          default: 2-4
          type: str

- kea_control_agent_config  Control agent configuration, see
                             https://kea.readthedocs.io/en/latest/arm/agent.html.
                             You do not need to include the top-level
                             "Control-agent" key. If configuration is
                             left unset, the control agent will not be
                             installed.
          default: {}
          type: dict

- kea_control_agent_package  Name of the control agent package
          default: isc-kea-ctrl-agent
          type: str

- kea_ddns_config  DHCP-DDNS server configuration, see
                    https://kea.readthedocs.io/en/latest/arm/ddns.html.
                    You do not need to include the top-level
                    "DhcpDdns" key. If configuration is left unset,
                    the DHCP-DDNS server will not be installed.
          default: {}
          type: dict

- kea_ddns_package  Name of the DHCP-DDNS server package, defaults to
                     "isc-kea-dhcp-ddns-server" if the kea version is
                     less than 2.3
          default: isc-kea-dhcp-ddns
          type: str

- kea_dhcp4_config  DHCPv4 server configuration, see
                     https://kea.readthedocs.io/en/latest/arm/dhcp4-srv.html.
                     You do not need to include the top-level "Dhcp4"
                     key. If configuration is left unset, the DHCPv4
                     server will not be installed.
          default: {}
          type: dict

- kea_dhcp4_package  Name of the DHCPv4 server package, defaults to
                      "isc-kea-dhcp4-server" if the kea version is
                      less than 2.3
          default: isc-kea-dhcp4-server
          type: str

- kea_dhcp6_config  DHCPv6 server configuration, see
                     https://kea.readthedocs.io/en/latest/arm/dhcp6-srv.html.
                     You do not need to include the top-level "Dhcp6"
                     key. If configuration is left unset, the DHCPv6
                     server will not be installed.
          default: {}
          type: dict

- kea_dhcp6_package  Name of the DHCPv6 server package, defaults to
                      "isc-kea-dhcp6-server" if the kea version is
                      less than 2.3
          default: isc-kea-dhcp6-server
          type: str

- kea_packages  List of extra packages to install
          default: [isc-kea-admin, isc-kea-hooks]
          elements: str
          type: list
```

Installation
------------

This role can either be installed manually with the ansible-galaxy CLI tool:

    ansible-galaxy install git+https://github.com/wandansible/kea,main,wandansible.kea
     
Or, by adding the following to `requirements.yml`:

    - name: wandansible.kea
      src: https://github.com/wandansible/kea

Roles listed in `requirements.yml` can be installed with the following ansible-galaxy command:

    ansible-galaxy install -r requirements.yml

Example Playbook
----------------

    - hosts: dhcp_servers
      roles:
         - role: wandansible.kea
           become: true
           vars:
             kea_dhcp4_config:
               valid-lifetime: 3600
               max-valid-lifetime: 7200
               interfaces-config:
                 interfaces:
                   - eno1
               lease-database:
                 type: memfile
                 persist: true
                 name: /var/lib/kea/kea-leases4.csv
               control-socket:
                 socket-name: /run/kea/kea-dhcp4.sock
                 socket-type: unix
               loggers:
                 - name: kea-dhcp4
                   output_options:
                     - output: /var/log/kea/kea-dhcp4.log
                   severity: WARN
               client-classes:
                 - name: pxe_efi_x64
                   boot-file-name: bootx64.efi
                   next-server: 192.0.2.1
                 - name: pxe_bios
                   boot-file-name: pxelinux.0
                   next-server: 192.0.2.1
               subnet4:
                 - subnet: 192.0.2.0/24
                   option-data:
                     - name: domain-name
                       data: example.org
                     - name: domain-search
                       data: dhcp.example.org, example.org
                     - name: domain-name-servers
                       data: 8.8.8.8, 8.8.4.4
                     - name: routers
                       data: 192.0.2.254
                   match-client-id: false
                   reservations:
                     - hostname: static-host.example.org
                       hw-address: aa:bb:cc:dd:ee:ff
                       ip-address: 192.0.2.2
                     - hostname: pxe-host.example.org
                       hw-address: uu:vv:ww:xx:yy:zz
                       ip-address: 192.0.2.3
                       client-classes:
                         - pxe_efi_x64
               hooks-libraries:
                 - library: /usr/lib/x86_64-linux-gnu/kea/hooks/libdhcp_stat_cmds.so
