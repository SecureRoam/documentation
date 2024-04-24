# Pi-hole

Pi-hole is a network-wide ad blocker that improves your browsing experience by blocking ads, trackers, and malware at the network level. It is a free and open-source software that runs on a Raspberry Pi or any other Linux-based system.

Pi-hole can be accessed at [dns-filter.secureroamcompanion.org](http://dns-filter.secureroamcompanion.org).

Pi-hole is also used to provide DHCP services to the network.

## Features

### DNS

Pi-hole acts as a Domain Name System (DNS) server for your network. It resolves domain names to IP addresses and blocks requests to known ad-serving domains. By default, Pi-hole uses the following upstream DNS servers:

- Google (default)
- OpenDNS
- Level3
- Comodo
- DNS.WATCH
- Quad9
- CloudFlare DNS
- Custom

During the pi-hole installation, you select 1 of the 7 preset providers or enter one of your own. Below you can find more information on each of the DNS providers, along with some additional providers which have different kinds of extra filtering options (spam, phishing, adult content, etc).

### DHCP

Pi-hole also provides Dynamic Host Configuration Protocol (DHCP) services to your network. It assigns IP addresses to devices on your network and provides them with the necessary network configuration information. By default, Pi-hole uses the following settings for DHCP:

- IP range: `192.168.25.2` to `192.168.25.254`
- Gateway: `192.168.25.1`
- DNS: `192.168.25.1`
- Lease time: `24 hours`
- Domain name: `secureroamcompanion.org`

## Configuration

To configure Pi-hole, follow these steps:

1. Open a web browser and navigate to [dns-filter.secureroamcompanion.org/admin](http://dns-filter.secureroamcompanion.org/admin).
2. Log in with the default credentials:
   - Username: `test`
   - Password: `test`
3. Follow the on-screen instructions to configure Pi-hole.
4. Once you have configured Pi-hole, you can start using it to block ads, trackers, and malware on your network.

For more information on how to configure Pi-hole, see the [Pi-hole documentation](https://docs.pi-hole.net/).
