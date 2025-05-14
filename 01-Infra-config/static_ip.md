# 📡 Static IP Configuration (Ubuntu Server with Wi-Fi)

This guide explains how to configure a **static IP address** on Ubuntu Server using **Netplan**, when connected via Ethernet or Wi-Fi and you don't have access to the router.

---

## 🧾 Requirements

- Ubuntu Server 20.04+ installed
- Ethernet interface (e.g., `enp2s0`)
- A wireless interface (e.g., `wlp3s0`)
- Wi-Fi credentials (SSID and password)
- Access to the terminal with `sudo` privileges

---

## 🚶 Step-by-Step

### 1. Identify Your Network Interface and Gateway

Run the following commands:

```bash
sudo lshw -C network
```
search for `logical name`

```bash
ip a
ip r
```

Note:

- Your Ethernet interface name (e.g., `enp2s0`)
- Your Wi-Fi interface name (e.g., `wlp3s0`)
- Your current gateway IP (e.g., `192.168.1.1`)
- A static IP you want to assign (e.g., `192.168.1.51` [Ethernet])
- A static IP you want to assign (e.g., `192.168.1.50` [Wi-Fi])

---

### 2. Create or Edit Netplan Configuration File

Open or create the Netplan config file:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Paste the following content (replace placeholders):

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp2s0:
      dhcp4: no
      addresses: [192.168.1.51/24]
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
      routes:
        - to: default
          via: 192.168.1.1
          metric: 100
  wifis:
    wlp3s0:
      dhcp4: no
      addresses: [192.168.1.50/24]
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
      routes:
        - to: default
          via: 192.168.1.1
          metric: 200
      access-points:
        "YOUR_WIFI_SSID 1":
          password: "YOUR_WIFI_PASSWORD 1"
        "YOUR_WIFI_SSID 2":
          password: "YOUR_WIFI_PASSWORD 2"
```

- `metric` to choose priority of connection (e.g. Preferential Ethernet [100] and Wi-Fi as fallback [200] )

Replace:

- `YOUR_WIFI_SSID` with your actual Wi-Fi name
- `YOUR_WIFI_PASSWORD` with your Wi-Fi password

> ⚠️ Make sure the IP addresses (`192.168.1.50` or/and `192.168.1.51`) is not used by other devices.

---

### 3. Secure File Permissions

```bash
sudo chmod 600 /etc/netplan/00-installer-config.yaml
```

---

### 4. Disable Conflicting Configurations

```bash
sudo mv /etc/netplan/50-cloud-init.yaml /etc/netplan/50-cloud-init.yaml.bak
```

---

### 5. Apply the Configuration

```bash
sudo netplan apply
```

To confirm the static IP:

```bash
ip a
```

---

### 6. Reboot (If Needed)

If the network doesn't reconnect:

```bash
sudo reboot
```

---

## ✅ Result

Your Ubuntu Server will now always use the static IP address `192.168.1.50` on your local network.

This is essential for internal DNS, reverse proxies, and local services.
