## Next Steps

Need to learn how to configure ethernet P2P with computer usiing Ethernet Point-to-Point Connection Between BeagleBone Black and Windows PC

This document describes how to configure a direct Ethernet connection between a **BeagleBone Black (BBB)** running a Buildroot-based Linux system and a **Windows 10 PC**.

This setup is useful when USB Ethernet (RNDIS) is unavailable or unsupported. The Ethernet interface (`eth0`) provides a reliable communication channel for:

* SSH access
* File transfer using SCP
* Remote debugging
* Application deployment

---

# 1. Hardware Setup

Connect the devices directly:

```
+-------------------+             +----------------------+
|   Windows PC      |             |  BeagleBone Black    |
|                   |             |                      |
| Ethernet port     | <=========> | Ethernet port        |
|                   |             |                      |
+-------------------+             +----------------------+
```

A standard Ethernet cable can be used.

---

# 2. Configure BeagleBone Black Network Interface

Access the BBB using UART console.

Check the available interfaces:

```bash
ip link
```

The Ethernet interface should be:

```text
eth0
```

Assign a static IP address:

```bash
ip addr add 192.168.7.2/24 dev eth0
```

Enable the interface:

```bash
ip link set eth0 up
```

Verify the configuration:

```bash
ip addr show eth0
```

Expected output:

```text
inet 192.168.7.2/24
```

---

# 3. Verify BBB Routing Configuration

Check the routing table:

```bash
ip route
```

Expected output:

```text
192.168.7.0/24 dev eth0 scope link src 192.168.7.2
```

This means the BBB knows how to reach the local Ethernet network.

---

# 4. Configure Windows Ethernet Interface

Open:

```
Control Panel
 └── Network and Internet
      └── Network Connections
           └── Ethernet
                └── Properties
                     └── Internet Protocol Version 4 (TCP/IPv4)
```

Configure the following static address:

```
IP address:
192.168.7.1

Subnet mask:
255.255.255.0

Default gateway:
(empty)

DNS:
(empty)
```

The final network configuration:

```
Windows PC                 BeagleBone Black

192.168.7.1  <----------->  192.168.7.2
              Ethernet
```

---

# 5. Enable ICMP Ping on Windows Firewall

By default, Windows Firewall may block incoming ICMP requests.

Open PowerShell as Administrator:

```powershell
netsh advfirewall firewall add rule name="Allow ICMPv4 Echo Request" protocol=icmpv4:8,any dir=in action=allow
```

This allows the BBB to ping the Windows machine.

---

# 6. Test the Connection

## Test from Windows to BBB

Open Command Prompt:

```cmd
ping 192.168.7.2
```

Expected:

```text
Reply from 192.168.7.2
```

---

## Test from BBB to Windows

On the BBB:

```bash
ping 192.168.7.1
```

Expected:

```text
64 bytes from 192.168.7.1
```

---

# 7. Verify Ethernet Layer (ARP)

On the BBB:

```bash
arp -n
```

Expected:

```text
? (192.168.7.1) at XX:XX:XX:XX:XX:XX [ether] on eth0
```

This confirms that the BBB successfully discovered the Windows Ethernet MAC address.

---

# 8. Enable SSH Access (Optional)

For development workflows, SSH is recommended.

Check if SSH server is running:

```bash
ss -tln
```

Look for:

```text
:22
```

If using Buildroot, enable Dropbear:

```
make menuconfig

Target packages
 └── Networking applications
      └── dropbear
```

Rebuild the image after enabling the package.

---

# 9. Transfer Applications Using SCP

After cross-compiling an application using the Buildroot SDK:

From WSL:

```powershell
scp -O hello_bbb root@192.168.7.2:/tmp
```

On the BBB:

```bash
./hello_bbb
```

---

# 10. Development Workflow

With this Ethernet setup, the development workflow becomes:

```
+----------------+
| PC             |
|                |
| Buildroot SDK  |
| Cross Compiler |
+-------+--------+
        |
        | SCP / SSH
        |
        v
+----------------+
| BeagleBone     |
|                |
| Linux          |
| Application    |
+----------------+
```

Typical workflow:

1. Build firmware using Buildroot
2. Generate SDK
3. Cross compile applications
4. Transfer binaries using SCP
5. Execute and debug on the BBB

---

# 11. Make Network Configuration Permanent

The commands:

```bash
ip addr add 192.168.7.2/24 dev eth0
ip link set eth0 up
```

are temporary and disappear after reboot.

For a permanent configuration, add the interface configuration to your Buildroot system.

Example:

```
/etc/network/interfaces
```

Configuration:

```text
auto eth0

iface eth0 inet static
    address 192.168.7.2
    netmask 255.255.255.0
```

After rebuilding the Buildroot image, the Ethernet interface will start automatically.

---

## Summary

This setup provides a simple and reliable Ethernet communication channel between a Windows development machine and a BeagleBone Black without requiring USB RNDIS drivers.

It enables a professional embedded Linux workflow using:

* Buildroot SDK
* Cross compilation
* SSH
* SCP
* Remote debugging
* Application deployment
ng ethernet cable.