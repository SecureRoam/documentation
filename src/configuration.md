# Configuration

By default, the security settings of the Secure Roam Companion are set to a restrictive level to ensure the highest level of security. However, you can customize the security settings to suit your needs.

## DNS settings

Secure Roam Companion uses Pi-hole to handle DNS requests and is configured to use the Google DNS servers by default. You can change the DNS settings to use a different DNS server.

Follow the [Pi-hole documentation](./services/Pi-Hole.md) to change the DNS settings.

## Firewall settings

Secure Roam Companion uses iptables to manage the firewall settings. The default configuration blocks all incoming connections except for the HTTP and HTTPS ports.

Your needs may vary, and you may want to open additional ports or block specific ports. You can customize the firewall settings by editing the iptables configuration.

Secure Roam Companion provides a simple web interface to manage the firewall settings trough Cockpit for simple configuration.

Follow the [iptables documentation](./services/iptables.md) to customize the firewall settings.

## Snort settings

Secure Roam Companion uses Snort to monitor network traffic and detect suspicious activity. The default configuration is set to alert mode, which logs suspicious activity without blocking it.

You can change the Snort settings to block suspicious activity by changing the configuration file.

Follow the [Snort documentation](./services/snort.md) to change the Snort settings.