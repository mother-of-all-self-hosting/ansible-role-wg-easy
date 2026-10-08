<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up WireGuard Easy

This is an [Ansible](https://www.ansible.com/) role which installs [WireGuard Easy](https://wg-easybudget.org) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

WireGuard Easy is a local-first personal finance tool.

See the project's [documentation](https://wg-easybudget.org/docs/) to learn what WireGuard Easy does and why it might be useful to you.

## Adjusting the playbook configuration

>[!NOTE]
> There are a few variables that you may wish to adjust before doing the initial [unattended setup](https://github.com/wg-easy/wg-easy/blob/v15.2.0/docs/content/advanced/config/unattended-setup.md). The reason it's important to do this early on is because certain variables (`wg_easy_environment_variables_additional_variable_init_*`) **only take effect during the initial setup phase**.

To enable WireGuard Easy with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# wg-easy                                                              #
#                                                                      #
########################################################################

wg_easy_enabled: true

########################################################################
#                                                                      #
# /wg-easy                                                             #
#                                                                      #
########################################################################
```

### Set the hostname

To enable WireGuard Easy you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
wg_easy_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting WireGuard Easy under a subpath (by configuring the `wg_easy_path_prefix` variable) does not seem to be possible due to WireGuard Easy's technical limitations.

### Set details for the initial setup user

You can create an instance's admin user for accessing the web UI by adding the following configuration to your `vars.yml` file. Make sure to replace values with your own ones.

```yaml
wg_easy_environment_variables_additional_variable_init_username: ADMIN_USERNAME_HERE
wg_easy_environment_variables_additional_variable_init_password: ADMIN_PASSWORD_HERE
```

`wg_easy_environment_variables_additional_variable_init_username` needs to be at least 8 characters long. Generating a strong password (e.g. `pwgen -s 64 1`) is recommended for `wg_easy_environment_variables_additional_variable_init_password`.

>[!NOTE]
> Subsequent changes to them will not affect the existing user.

### Adjusting the web UI URL

In the example configuration above, we configure the service to be hosted at `https://wg-easy.example.com/`.

You can adjust the hostname of the web UI with the `wg_easy_hostname` variable.

Previously (prior to wg-easy v15), a `wg_easy_path_prefix` variable could allow you to host wg-easy at a subpath (e.g. `wg_easy_path_prefix: /wg-easy`), but this is [no longer possible](https://github.com/wg-easy/wg-easy/issues/1704#issuecomment-2705873936) and such a feature [may re-appear later](https://github.com/wg-easy/wg-easy/issues/1704#issuecomment-2706575504).

💡 WireGuard clients may optionally be pointed to a different hostname than the one used for the web UI. Refer to [Adjusting the WireGuard endpoint](#adjusting-the-wireguard-endpoint) for details.

### Adjusting the Wireguard endpoint

By default, the WireGuard endpoint that clients are configured to connect to uses:

- the same hostname as the web UI (controlled by the `wg_easy_hostname` variable)
- the port `51820`, configured via the `wg_easy_environment_variables_additional_variable_init_port` and `wg_easy_container_wireguard_bind_port` variables

Sometimes, you may wish to use a different hostname or port for the WireGuard endpoint. You can do so with the following variables:

```yml
# Controls the public hostname of the WireGuard endpoint that will be configured during initial unattended setup.
wg_easy_environment_variables_additional_variable_init_host: wg-easy.example.com

# Controls the public port of the WireGuard endpoint that will be configured during initial unattended setup.
# Also points to the port actually used in the container (in case you've reconfigured it via the web UI).
wg_easy_environment_variables_additional_variable_init_port: 51820

# Controls the exposed (published) port of the WireGuard container.
# By default, this matches the `wg_easy_environment_variables_additional_variable_init_port` value.
wg_easy_container_wireguard_bind_port: 51820
```

> [!WARNING]
> If you need to change the hostname or port after the initial setup, you need to do so from the Admin Panel -> Config section (`/admin/config` URL path) of the web UI.

### Adjusting the default DNS servers

If you'd like to provide custom DNS servers (instead of the default ones seen below), you can do so with the following **initial unattended setup** variables:

```yml
wg_easy_environment_variables_additional_variable_init_dns: "1.1.1.1,2606:4700:4700::1111"
```

> [!WARNING]
> If you need to change the DNS servers after the initial setup, you need to do so from the Admin Panel -> Config section (`/admin/config` URL path) of the web UI.

💡 DNS configuration can also be adjusted later on, on a per-client basis, but these changes need to be made before one downloads the WireGuard configuration profile files, because they do hardcode the DNS configuration (and all lots of other configuration) inside them.

### Adjusting the IPv4/IPv6 CIDR

If you'd like to provide custom IPv4 and IPv6 CIDRs (instead of the default ones seen below), you can do so with the following **initial unattended setup** variables:

```yml
wg_easy_environment_variables_additional_variable_init_ipv4_cidr: "10.8.0.0/24"

# This looks like the documentation-reserved IPv6 CIDR value, because it is. Read why below.
wg_easy_environment_variables_additional_variable_init_ipv6_cidr: "2001:db8::/32"
```

💡 The `wg_easy_environment_variables_additional_variable_init_ipv6_cidr` value you see above is what we use by default. It represents the documentation-reserved IPv6 CIDR value, but we're not only using it for documentation purposes, but because it's a GUA-like CIDR value. Refer to [Note about the IPv6 CIDR and IPv6 connectivity](#note-about-the-ipv6-cidr-and-ipv6-connectivity) for more details and for a recommended alternative if you can use your own GUA address.

> [!WARNING]
> If you need to change the IPv4/IPv6 CIDRs after the initial setup, you need to do so from the Admin Panel -> Interface page of the web UI, via the Change CIDR button. After changing the CIDR in wg-easy's settings, you must restart the wg-easy service for the changes to take effect.

### Adjusting the default Allowed IPs

If you'd like to set global [Allowed IPs](https://techoverflow.net/2021/07/09/what-does-wireguard-allowedips-actually-do/) for all WireGuard clients during the initial setup, you can do so with the following **initial unattended setup** variable:

```yml
wg_easy_environment_variables_additional_variable_init_allowed_ips: "10.8.0.0/24,2001:0DB8::/32"
```

> [!WARNING]
> Changing this variable after the initial setup will not have any effect. If you need to adjust Allowed IPs after the initial setup, you need to do so on a per-client basis from the web UI.

### Adjusting your firewall

**In addition** to ports `80` and `443` exposed by the [Traefik](traefik.md) reverse-proxy, the following ports will be exposed by the WireGuard containers on **all network interfaces**:

- `51820` over **UDP**, controlled by `wg_easy_container_wireguard_bind_port` — used for [Wireguard](https://www.wireguard.com/) connections

Docker automatically opens these ports in the server's firewall, so you **likely don't need to do anything**. If you use another firewall in front of the server, you may need to adjust it.

### Adjusting the host's iptables configuration

If you're running `iptables`/`ip6tables` on the host with a custom config (which whitelists some traffic and denies everything else), you may find that WireGuard clients cannot reach certain ports on server where wg-easy runs via its LAN IP address (e.g. `192.168.1.50`).

You may wish to adjust your iptables configuration (typically `/etc/iptables/iptables.rules`) like this:

```iptables
# ... Additional configuration ...

# Allow all private IPv4 ranges (RFC1918 private addresses) to access us via SSH.
#
# This allows wg-easy WireGuard clients which try to speak to us via our LAN IP to be able to reach us.
# They "exit" through mash-wg-easy's container subnet (e.g. 172.18.0.1).
-A INPUT -m tcp -p tcp --dport 22 -s 10.0.0.0/8 -j ACCEPT
-A INPUT -m tcp -p tcp --dport 22 -s 172.16.0.0/12 -j ACCEPT
-A INPUT -m tcp -p tcp --dport 22 -s 192.168.0.0/16 -j ACCEPT

# Allow private IPv4 ranges to access us via HTTP.
# This is like the above private IPv4 range rules for SSH.
-A INPUT -m tcp -p tcp --dport 80 -s 10.0.0.0/8 -j ACCEPT
-A INPUT -m tcp -p tcp --dport 80 -s 172.16.0.0/12 -j ACCEPT
-A INPUT -m tcp -p tcp --dport 80 -s 192.168.0.0/16 -j ACCEPT

# ... Additional configuration ...
```

or for ip6tables (typically `/etc/iptables/ip6tables.rules`) like this:

```iptables
# ... Additional configuration ...

# Allow all private IPv6 ranges to access us via SSH.
#
# This allows wg-easy WireGuard clients which try to speak to us via our LAN IP to be able to reach us.
# They "exit" through mash-wg-easy's container subnet (e.g. 172.18.0.1).
# Unique Local Addresses (ULA)
-A INPUT -p tcp --dport 22 -s fc00::/7 -j ACCEPT
# Link-local addresses
-A INPUT -p tcp --dport 22 -s fe80::/10 -j ACCEPT

# Allow private IPv6 ranges to access us via HTTP.
# This is like the above private IPv6 range rules for SSH.
-A INPUT -m tcp -p tcp --dport 80 -s fc00::/7 -j ACCEPT
-A INPUT -m tcp -p tcp --dport 80 -s fe80::/10 -j ACCEPT

# ... Additional configuration ...
```

After doing so, you'll wish to restart `iptables`/`ip6tables`. Restarting iptables typically necessitates restarting Docker (`docker.service`) as well.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `wg_easy_environment_variables_additional_variables` variable

>[!NOTE]
> The new wg-easy version (after the v15 release) does not support most of the environment variables that were supported in previous versions. Most of the configuration happens via the web UI after installation. Refer to [Adjusting the post-installation configuration](#adjusting-the-post-installation-configuration) for more details.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, WireGuard Easy becomes available at the specified hostname like `https://example.com`. To use it, open the URL on the browser and create an account.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu wg-easy` (or how you/your playbook named the service, e.g. `mash-wg-easy`).
