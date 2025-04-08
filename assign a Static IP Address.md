Setting a Static IP Address

# Setting a Static IP Address

## Rocky Linux / CentOS (Using `nmtui`)

1. Open the network configuration tool:
   ```bash
   sudo nmtui
   ```

2. Go to **"Edit a connection"**, select your network interface (like `ens33` or `eth0`), and hit Enter.

3. Change **IPv4 Configuration** to `Manual`.

4. Enter the following values (example):
   - **Address**: `192.168.1.50/24`
   - **Gateway**: `192.168.1.1`
   - **DNS**: `8.8.8.8, 8.8.4.4`

5. Save the configuration and return to the main menu.

6. Restart the network manager:
   ```bash
   sudo systemctl restart NetworkManager
   ```

7. Confirm your IP:
   ```bash
   ip a
   ```

Your static IP should now be active and persistent across reboots.

---

## Ubuntu (Using Netplan)

1. Check your network interface name:
   ```bash
   ip a
   ```

2. Edit the Netplan configuration:
   ```bash
   sudo nano /etc/netplan/01-netcfg.yaml
   ```

3. Replace or add the following configuration (update the interface name if needed):

   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       ens33:
         dhcp4: no
         addresses:
           - 192.168.1.51/24
         gateway4: 192.168.1.1
         nameservers:
           addresses: [8.8.8.8, 8.8.4.4]
   ```

4. Apply the changes:
   ```bash
   sudo netplan apply
   ```

5. Verify the IP address:
   ```bash
   ip a
   ```


   ## extra ---------------------------------------------------------------------


# Static IP Configuration on Various Linux Distros

## Rocky Linux / CentOS

### 1. Using `nmcli`
```bash
nmcli con show
nmcli con mod "Wired connection 1" ipv4.addresses 192.168.1.50/24
nmcli con mod "Wired connection 1" ipv4.gateway 192.168.1.1
nmcli con mod "Wired connection 1" ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con mod "Wired connection 1" ipv4.method manual
nmcli con up "Wired connection 1"
```

### 2. Using `ifcfg` (CentOS 7 and below)
```bash
sudo nano /etc/sysconfig/network-scripts/ifcfg-ens33
```
```
DEVICE=ens33
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.1.50
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
```
```bash
sudo systemctl restart network
```

## Ubuntu / Debian

### 1. `/etc/network/interfaces` (Older systems)
```bash
sudo nano /etc/network/interfaces
```
```
auto ens33
iface ens33 inet static
  address 192.168.1.51
  netmask 255.255.255.0
  gateway 192.168.1.1
  dns-nameservers 8.8.8.8 8.8.4.4
```
```bash
sudo systemctl restart networking
```

### 2. GUI Method
- Open **Settings > Network**
- Click gear icon next to interface
- Go to **IPv4**, set Method to **Manual**
- Enter IP, Netmask, Gateway, DNS
- Save and reconnect

## Arch Linux

```bash
sudo nano /etc/systemd/network/20-wired.network
```
```
[Match]
Name=enp0s3

[Network]
Address=192.168.1.60/24
Gateway=192.168.1.1
DNS=8.8.8.8
```
```bash
sudo systemctl enable systemd-networkd
sudo systemctl restart systemd-networkd
```

## Alpine Linux

```bash
sudo vi /etc/network/interfaces
```
```
auto eth0
iface eth0 inet static
    address 192.168.1.61
    netmask 255.255.255.0
    gateway 192.168.1.1
```
```bash
/etc/init.d/networking restart
```
```


