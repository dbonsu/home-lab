# OPNsense Homelab Setup Guide with VLANs, Switch, and OpenWRT AP for Archer A7

---

I took a lot of inspiration from Dustin [https://homenetworkguy.com/how-to/install-and-configure-opnsense/](https://homenetworkguy.com/how-to/install-and-configure-opnsense/) and several you tube channels.

Hardware:
Dell Optiplex 5040, Intel core i5-6500, 8GB RAM, 128GB SSD (Ebay special $60)
Intel dual NIC 1350-GE-2T PCIe x4(Amazon $33)
TP-Link TL-SG108E 8 Port 1 GB managed switch (Amazon $24)
TP-Link Archer A7 (old donated item)

**Total** with tax but not including Archer A7 = **$121**

## 📊 Network Overview

I will be setting up a few VLANs to separate my internal network.

| VLAN Name | VLAN ID | Subnet       | Purpose                       | Access             |
| --------- | ------- | ------------ | ----------------------------- | ------------------ |
| Main      | 10      | 10.0.0.1/24  | Network devices,proxmox, etc. | internet, all VLAN |
| Guest     | 20      | 10.0.20.1/24 | Guest                         | internet only      |
| Services  | 30      | 10.0.30.1/24 | server/services, NAS/media    | internet           |

I will be exposing some ports on services using traefik(TBD)

## 🛠️ Step-by-Step Setup

### 1. Install OPNsense on Dell Optiplex 5040

1. Flash OPNsense to USB and boot Dell from USB.
2. Install OPNsense on internal SSD.
3. Assign:

   - WAN = One NIC (leave unplugged for now)
   - LAN = Second NIC (connect to laptop temporarily) and label physically

4. Reboot, login at `192.168.1.1`, user `root`, pass `opnsense` unless you changed it during the install.

### 2. ** Opnsense Settings**

Go through the Wizard. Change opnsense LAN IP to another range or leave it with the default

1. **General**
   |Options | value |
   |--------|-------|
   |Hostname| x.home (your choice)|
   |Domain| x.lan (your choice)|
   |DNS server options | check "Allow DNS server list to be overridden by DCHP/PPP on WAN"|
   |primary DNS| blank|
   |secondary DNS| blank|

2. **Administration**
   |Options | value |
   |--------|-------|
   |Protocol| HTTPS|
   |TCP Port| 443|
   |HTTP Redirect | uncheck|
   |DNS Rebind Check| uncheck|
   |HTTP Compression| Medium|
   |Listen Interfaces| "All(recommended)"|
3. **Misc**
   |Options | value |
   |--------|-------|
   |Thermal Sensor| Intel|
   |Periodic RDP Backup| 24 hours|
   |Periodic DHCP Leases |24 hours|
   |Periodic Netflow| 24 hours|
   |Power mode| Hiadaptive|

4. **Interface Settings**
   |Options | value |
   |--------|-------|
   |Hardware CRC| Check “Disable hardware checksum offload” (if not already checked)|
   |Hardware TSO| Check “Disable hardware TCP segmentation offload” (if not already checked)|
   |Hardware LRO |Check “Disable hardware large receive offload” (if not already checked)|
   |VLAN Hardware Filtering| Choose the “Disable VLAN Hardware Filtering” option|

---

### 3. **Bridge ISP Modem**

1. Plug laptop into ISP modem.
2. Access web UI (`192.168.0.1` or `192.168.100.1` or check values on the router).
3. Enable **bridge mode**.
   - Upon enabling IP passthrough (note from verizon):
   - All Wi-Fi radios are automatically disabled (and re-enabled upon disabling IP passthrough).
   - IP passthrough functions are enabled on the LAN2 Ethernet port on the bottom of the router.
   - internet access is no longer available through Wi-Fi or the LAN1 Ethernet port.
4. Steps on verizon web site:
   - Advanced --> Network Settings --> Network Connections --> Network(home/Office)
   - Click settings from this menu
   - Find Bridge Section and check IP passthrough
5. router should reboot after 1-2 minutes/Reboot modem.

---

### 4. **Connect WAN and Verify LAN**

1. Connect ISP modem to OPNsense **WAN port**.
2. Connect LAN port to laptop and verify you can `opnsense` .
3. Confirm internet access through bridged modem.
4. Update Opnsense: System -> Firmware

---

### 5. **Configure VLANs in OPNsense**

1. Go to **Interfaces > Other Types > VLANs**.
2. Create VLANs: 10, 20, 30, 40 with LAN NIC as parent.
   |Options | value |
   |--------|-------|
   |Device| leave blank|
   |Parent| LAN nic (confirm name)|
   |VLAN Tag |10|
   |Periodic Netflow| 24 hours|
   |Power mode| Hiadaptive|
3. Go to **Interfaces > Assignments**, assign each VLAN.
4. Enable each, set static IPs: use this example

   - VLAN 10: 10.0.10.1/24
   - VLAN 20: 10.0.20.1/24
   - VLAN 30: 10.0.30.1/24
   - VLAN 40: 10.0.40.1/24

---

### 5. **Configure DHCP and Firewall Rules in OPNsense**

- **Add `PrivateNetworks` Alias. this will include RCF 1918 networks `10.0.0.0/8,172.16.0.0/12,192.168.0.0/16`**
- **Services > [LAN]**

### 5.1 LAN

- Enable DHCP: 10.0.X.100 - .199 for each VLAN
- Firewall Rule for each VLAN:

  - Action: Pass
  - Source: VLAN net
  - Destination: any

#### Main VLAN Firewall Rules:

- Block all other VLANs from reaching 10.0.10.0/24.

---

### 6. **Configure TP-Link Managed Switch**

1. Create VLANs 10, 20, 30, 40.
2. Set Port 1 (to OPNsense) as **Tagged (T)** for all VLANs.
3. Assign Access Ports:

| Port | VLAN  | Mode     | Purpose           |
| ---- | ----- | -------- | ----------------- |
| 1    | 10-40 | Tagged   | Trunk to OPNsense |
| 2    | 10    | Untagged | Main              |
| 3    | 20    | Untagged | Guest             |
| 4    | 30    | Untagged | Services          |

---

### 7. **Set Up OpenWRT Archer A7 (Trunk + SSIDs)**

1. Flash OpenWRT, access via LAN cable.
2. Go to **Network > Switch**:

   - Create VLANs 10, 20, 30, 40 with CPU port tagged.

3. Go to **Network > Interfaces**, create interfaces:

   - `eth0.10`, `eth0.20`, `eth0.30`, `eth0.40`

4. Go to **Wireless**, create SSIDs:

   - `Main` → Interface: `eth0.10`
   - `Guest` → Interface: `eth0.20`

---

### 8. **Firewall Hardening Summary**

- OPNsense admin UI - 10.0.10.1
- Switch UI (10.0.10.2)
- OpenWRT UI (10.0.10.3)

- **Block all other VLANs from 10.0.0.0/24**

---

## 📆 Final Notes

- Consider Suricata or Zenarmor for traffic inspection
- Backup your configs on OPNsense, Switch, and OpenWRT after setup
- Ad guard home

---

**End of Guide**
