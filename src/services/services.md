# Services

To make the user experience more enjoyable, Secure Roam Companion provides a set of services that improve the network's security, privacy, and performance. These services are free and open-source software that can be installed on a Raspberry Pi or any other Linux-based system.

Most of these services are accessible via a Cockpit web interface that makes it easy to administer your GNU/Linux servers via a web browser.

NGINX is used as a reverse proxy to route traffic to the appropriate service based on the domain name. This allows you to access the services using user-friendly URLs such as `http://secureroamcompanion.org` for [Cockpit](./cockpit.md) or `http://dns-filter.secureroamcompanion.org` for [Pi-hole](./Pi-Hole.md).

Secure Roam Companion provides the following services:

| Services                   | Description                  |
| -------------------------- | ---------------------------- |
| [Cockpit](./cockpit.md)    | Browser-based administration |
| [Pi-hole](./Pi-Hole.md)    | Network-wide Ad Blocking     |
| [Snort](./snort.md)        | Network Intrusion Detection  |
| [iptables](./iptables.md)  | Firewall                     |

## Cockpit

Cockpit is a web-based server manager that makes it easy to administer your GNU/Linux servers via a web browser. It is a free and open-source software that provides a friendly interface for system administrators.

See the [Cockpit](./cockpit.md) page for more information.

## Pi-hole

Pi-hole is a network-wide ad blocker that improves your browsing experience by blocking ads, trackers, and malware at the network level. It is a free and open-source software that runs on a Raspberry Pi or any other Linux-based system.

See the [Pi-hole](./Pi-Hole.md) page for more information.
