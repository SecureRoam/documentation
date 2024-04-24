# Network configuration

The Raspberry Pi 4 can use the USB-C port to connect to a computer and share the internet connection using the USB Ethernet gadget.

RNDIS is a Microsoft proprietary protocol that allows a computer to use the USB port to emulate an Ethernet connection. The Raspberry Pi 4 supports RNDIS out of the box.

`libComposite` is a Linux kernel module that allows the Raspberry Pi 4 to emulate a USB Ethernet gadget. The module is not enabled by default, and it needs to be enabled manually.

## How it works

The Raspberry Pi 4 is connected to a computer using the USB-C port. The Raspberry Pi 4 is configured to emulate a USB Ethernet gadget using the `libComposite` module. The computer detects the Raspberry Pi 4 as a USB Ethernet device and assigns an IP address to the Raspberry Pi 4. The Raspberry Pi 4 can then share the internet connection with the computer.

ECM/CDC is a protocol that allows the Raspberry Pi 4 to emulate a USB Ethernet gadget. The Raspberry Pi 4 supports ECM/CDC out of the box but Microsoft use only RNDIS. RNDIS is a fork of an old DOS protocol called NDIS made to work with USB. This Microsoft software is the only way to get internet over USB on Windows, it's compatible with the open ECM/CDC protocol.

## Interface

Secure Roam Companion needs three interfaces to work. The first two interfaces are `usb0` and `usb1`, the USB Ethernet gadget that is used to connect to the computer. The third interface is `wlan0`, the Wi-Fi interface that is used to connect to the internet.

The `usb0` is used for UNIX-like systems, and the `usb1` is used for Windows systems (RNDIS).

## Configuration

Follow theses step-by-step instruction to replicate the original configuration.

The following packages need to be installed : `iptables`, `firewalld` and `networkmanager`.

```bash
sudo apt-get install iptables firewalld networkmanager
```

### BIOS configuration file

Raspberry Pi uses a configuration file instead of the BIOS you would expect to find on a conventional PC. The system configuration parameters, which would traditionally be edited and stored using a BIOS, are stored instead in an optional text file named `config.txt`.

To enable USB On-The-Go, the specification used for Ethernet over USB, the firmware `dwc2` need to be added using the `dtoverlay` instruction.

Inside `/boot/firmware/config.txt`, add :

```bash
dtoverlay=dwc2
```

`cmdline` is the alternative filename on the boot partition from which to read the kernel command line string. Once the firmware activated, load `dwc2` and `g_ether` modules using the `modules-load` instruction.

<div class="warning">
"g_ether" was used for legacy reasons (only supported by Microsoft RNDIS 5). If "g_ether" is used, Windows will recognize this device as "USB Serial Device (COMx)" and will not install the RNDIS driver automatically.
</div>

Inside `/boot/firmware/cmdline.txt`, add after the `rootwait` line :

```bash
dwc_otg.lpm_enable=0 console=tty1 root=PARTUUID=b9cecb52-02 rootfstype=ext4 fsck.repair=yes rootwait cfg80211.ieee80211_regdom=FR modules-load=dwc2
```

Add at the end of `/etc/modules` :

```bash
libcomposite
```

In the `/root` folder, create a script called `usb.sh` with the following content. A walkthrough of the script is available in the comments.

```bash
#!/bin/bash

# Set up the USB gadget
configfs="/sys/kernel/config/usb_gadget"  # The path to the configfs
this="${configfs}/secureRoamCompanion"           # The path to the libComposite device

# Configure values
serial="$(grep 'Serial' /proc/cpuinfo | head -n 1 | sed -E -e 's/^Serial\s+:\s+0000(.+)/\1/')"  # The serial number of the Pi
model="$(grep 'Model' /proc/cpuinfo | head -n 1 | sed -E -e 's/^Model\s+:\s+(.+)/\1/')" # The model of the Pi
manufacturer="Secure Roam Companion"  # The manufacturer of the Pi

# The serial number ends in a mac-like address. Let's use this to build a MAC address.
#   The first binary xxxxxx10 octet "locally assigned, unicast" which means we can avoid
#   conflicts with other vendors.
mac_base="$(echo "${serial}" | sed 's/\(\w\w\)/:\1/g' | cut -b 4-)"
ecm_mac_address_dev="02${mac_base}"  # ECM/CDC address for the Pi end
ecm_mac_address_host="12${mac_base}" # ECM/CDC address for the "host" end that the Pi is plugged into
rndis_mac_address_dev="22${mac_base}"  # RNDIS address for the Pi end
rndis_mac_address_host="32${mac_base}" # RNDIS address for the "host" end that the Pi is plugged into

# Make sure that libComposite is loaded
libcomposite_loaded="$(lsmod | grep -e '^libcomposite' 2>/dev/null)"
[ -z "${libcomposite_loaded}" ] && modprobe libcomposite
while [ ! -d "${configfs}" ]
do
  sleep 0.1
done

# Make the path to the libComposite device
mkdir -p "${this}"

echo "0x0200" > "${this}/bcdUSB"       # USB Version (2)
echo "0x1d6b" > "${this}/idVendor"     # Device Vendor: Linux Foundation
echo "0x0104" > "${this}/idProduct"    # Device Type: MultiFunction Composite Device
echo "0x02" > "${this}/bDeviceClass" # This means it is a communications device

# Device Version (this seems a bit high, but OK)
# This should be incremented each time there's a "breaking change" so that it's re-detected
# rather than cached (apparently)
echo "0x4000"                      > "${this}/bcdDevice"

# "The OS_Desc config must specify a valid OS Descriptor for correct driver selection"
#   See: https://www.kernel.org/doc/Documentation/ABI/testing/configfs-usb-gadget
mkdir -p "${this}/os_desc"
echo "1" > "${this}/os_desc/use" # Enable OS Descriptors
echo "0xcd" > "${this}/os_desc/b_vendor_code" # Extended feature descriptor: MS
echo "MSFT100" > "${this}/os_desc/qw_sign" # OS String "proper"

# Configure the strings the device presents itself as
mkdir -p "${this}/strings/0x409"
echo "${manufacturer}" > "${this}/strings/0x409/manufacturer"
echo "${model}" > "${this}/strings/0x409/product"
echo "${serial}" > "${this}/strings/0x409/serialnumber"

# Set up the ECM/CDC and RNDIS network interfaces as
#   configs/c.1 and configs/c.2 respectively.
for i in 1 2
do
  mkdir -p "${this}/configs/c.${i}/strings/0x409"
  echo "0xC0" > "${this}/configs/c.${i}/bmAttributes" # Self Powered
  echo "250" > "${this}/configs/c.${i}/MaxPower" # 250mA
done

# Add the Serial interface
mkdir -p "${this}/functions/acm.usb0"
ln -s "${this}/functions/acm.usb0" "${this}/configs/c.1/"

# Set up the ECM/CDC function
mkdir -p "${this}/functions/ecm.usb0"
echo "${ecm_mac_address_host}" > "${this}/functions/ecm.usb0/host_addr"
echo "${ecm_mac_address_dev}" > "${this}/functions/ecm.usb0/dev_addr"
echo "CDC" > "${this}/configs/c.1/strings/0x409/configuration"
ln -s "${this}/functions/ecm.usb0" "${this}/configs/c.1/"

mkdir -p "${this}/functions/rndis.usb0"
mkdir -p "${this}/functions/rndis.usb0/os_desc/interface.rndis"
echo "RNDIS" > "${this}/functions/rndis.usb0/os_desc/interface.rndis/compatible_id"
echo "5162001" > "${this}/functions/rndis.usb0/os_desc/interface.rndis/sub_compatible_id"
echo "${rndis_mac_address_host}" > "${this}/functions/rndis.usb0/host_addr"
echo "${rndis_mac_address_dev}" > "${this}/functions/rndis.usb0/dev_addr"
echo "RNDIS" > "${this}/configs/c.2/strings/0x409/configuration"
ln -s "${this}/configs/c.2" "${this}/os_desc"
ln -s "${this}/functions/rndis.usb0" "${this}/configs/c.2/"

udevadm settle -t 5 || true
```

### USB0

In the `/etc/network/interfaces.d/usb0` file, add :

```bash
auto usb0
allow-hotplug usb0
iface usb0 inet static
    address 192.168.25.1
    netmask 255.255.255.0
```

This configuration file sets up the `usb0` interface with a static IP address of `192.168.25.1`.

### USB1

In the `/etc/network/interfaces.d/usb1` file, add :

```bash
auto usb1
allow-hotplug usb1
iface usb1 inet static
    address 192.168.25.1
    netmask 255.255.255.0
```

At the end of `/etc/dnsmasq.d/99-interfaces.conf` file, add :

```bash
interface=usb1
```

This configuration file sets up the `usb1` interface with a static IP address of `192.168.25.1`

### DHCP

Pi-hole is used as a DHCP server for the `usb0` and `usb1` interfaces. Add the following lines to the `/etc/dnsmasq.d/02-pihole-dhcp.conf` file.

```bash
interface=usb1
interface=usb0
bind-interfaces
dhcp-authoritative
dhcp-range=usb0,192.168.25.2,192.168.25.254,24
dhcp-range=usb1,192.168.25.2,192.168.25.254,255.255.255.0,12h
dhcp-option=option:router,192.168.25.1
dhcp-option=usb0,option:router,192.168.25.1
dhcp-option=usb1,option:router,192.168.25.1
dhcp-option=usb0,option:router,192.168.25.1

dhcp-leasefile=/etc/pihole/dhcp.leases
#quiet-dhcp

domain=src-pi-hole
local=/src-pi-hole/
dhcp-rapid-commit
```

<div class="warning">
If any changes are made using the GUI of Pi-hole, the 02-pihole-dhcp.conf file will be overwritten.
</div>

### rc.local

rc.local is a script that is executed at the end of the boot process.

Add the following line to the `/etc/rc.local` file before the `exit 0` line.

```bash
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi
firewall-cmd --add-service=dhcp

iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
iptables -A FORWARD -i usb0 -o wlan0 -j ACCEPT
iptables -A FORWARD -i wlan0 -o usb0 -m state --state RELATED,ESTABLISHED -j ACCEPT

echo True > /etc/LedService/started.txt # only for the LED service
```

This script sets up IP masquerading and packet forwarding between the `usb0` and `wlan0` interfaces.  

### Systemd service

A systemd service can be created to run the script at boot. Create a file called `secureroam.service` in the `/etc/systemd/system` folder with the following content.

```toml
[Unit]
Description=Start Secure Roam Companion USB Gadget

[Service]
Type=oneshot
ExecStart=/root/usb.sh

[Install]
WantedBy=multi-user.target
```

This service can be enabled and started using the following commands.

```bash
systemctl enable secureroam
systemctl start secureroam
```

The Raspberry Pi 4 should now be able to share the internet connection with a computer using the USB-C port.

Uncomment the following lines inside `/etc/sysctl.conf` to enable IP forwarding.

```bash
ipv4.ipforward
```

## References

Based on a combination of: <http://www.isticktoit.net/?p=1383> and: <https://www.raspberrypi.org/forums/viewtopic.php?t=260107> and: <https://gist.github.com/schlarpc/a327d4aa735f961555e02cbe45c11667/c80d8894da6c716fb93e5c8fea98899c9aab8d89> and: <https://github.com/ev3dev/ev3-systemd/blob/02caecff9138d0f4dcfeb5afbee67a0bb689cec0/scripts/ev3-usb.sh>.

Thanks to Jon Spriggs for the original script and [blog post](https://jon.sprig.gs/blog/post/2243).
