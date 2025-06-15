# 🧰 Infra Setup

A complete guide to setting up a local home server for development, automation, and full-cycle infrastructure experiments using **Ubuntu Server** and **Docker**.

1. ⬇️ Ubuntu Server Installation
1. [📡 Static IP setup](./static_ip.md)
   (for Internal DNS and Reverse Proxies, ssh, port fowarding, ...)
1. [🌐 Networking Setup](./networking.md) ()
1. [🚫 Pi-hole DNS Ad Blocker Setup](./pihole.md)

---

## 💻 Hardware Specs

| Component        | Details               |
| ---------------- | --------------------- |
| **Device**       |                       |
| Brand            | Asus                  |
| Model            | S451L                 |
| **CPU**          |                       |
| Processor        | Intel Core i7-4500U   |
| **Memory**       |                       |
| RAM              | 6 GB                  |
| **Storage**      |                       |
| Internal Disk    | 500 GB HDD            |
| External SSD     | 1 TB (Docker volumes) |
| **Screen**       |                       |
| Size             | 14" Touch             |
| Resolution       | 1366 x 768            |
| **Connectivity** |                       |
| Wi-Fi            | ✅ Yes                |
| Ethernet         | ✅ Yes                |
| USB-A Ports      | ✅ Yes                |
| DVD Drive        | ✅ Yes                |
| **Weight**       | 2.2 kg                |

---

## 🔧 Ubuntu Server Installation

### 📺 References

- [📹 YouTube: Turn any PC into a server with Ubuntu Server](https://www.youtube.com/watch?v=DGTkPW42VxI)
- [📦 Ubuntu Server 24.04 LTS (Download)](https://ubuntu.com/download/server)
- [🛠️ Ventoy - Bootable USB Tool](https://www.ventoy.net/en/download.html)

### 🪛 Installation Steps

1. Flash Ubuntu Server ISO using [Ventoy](https://www.youtube.com/watch?v=11CkqZQ3scE)
2. Plug bootable USB and boot the notebook (usually `F2` for BIOS)
3. Set USB as first boot option
4. Select `Try or Install Ubuntu Server`
5. Choose `Ubuntu Server` as base system
6. Configure Wi-Fi or Ethernet (DHCP by default)
7. Customize storage (or use defaults)
8. Set up user profile and SSH access
9. Wait for install to finish, then reboot

---

## 🛜 Remote Access via SSH

1. Get the server's IP:

   ```bash
   ip a
   ```

2. Connect from another machine:

   ```bash
   ssh <your-user>@<ip-address>
   ```

---

## 💤 Lid & Display Behavior

Ensure the server remains **active when the lid is closed**, while also **turning off the display after inactivity**:

### 🛑 Prevent Suspend on Lid Close

Edit the logind configuration:

```bash
sudo nano /etc/systemd/logind.conf
```

Set:

```ini
HandleLidSwitch=ignore
HandleLidSwitchDocked=ignore
```

Restart the service:

```bash
sudo systemctl restart systemd-logind
```

---

### 🌙 Turn Off Screen After Inactivity

Create or edit `/etc/rc.local` to configure screen blanking:

```bash
sudo nano /etc/rc.local
```

Add before `exit 0`:

```bash
setterm --blank 5 --powerdown 10 --powersave on < /dev/tty0
exit 0
```

Then make the file executable:

```bash
sudo chmod +x /etc/rc.local
```

📝 **Explanation**:

- `--blank 5` → turn off screen after 5 minutes
- `--powerdown 10` → reduce power after 10 minutes
- `--powersave on` → enter power save mode

---

> ✅ Result: lid closed = screen off, system alive.
> ⏱️ No interaction = screen turns off.
> 🔼 Lid opened or key press = screen turns on.

## 🐳 Docker Installation

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io
```

### 👤 Add User to Docker Group

```bash
sudo groupadd docker     # Only if group doesn't exist
sudo usermod -aG docker <your-user>
sudo systemctl restart docker
```

> Log out and back in `sudo poweroff`, `sudo shutdown -P now` or `sudo init 0` (or run `su <your-user>`) to apply group permissions.

Check Docker is working:

```bash
docker ps
```

---

## 🗂️ Project File Structure

```bash
/home/<user>/
├── docker/         # Dockerfiles and docker-compose configs
├── projects/       # Code and applications
```

Docker volumes are stored on an external SSD for persistence.

---

## 💾 Mounting External SSD (Persistent Setup)

### 🔧 Why ext4?

`ext4` is the recommended filesystem for Linux-based servers. It offers high performance, journaling (to prevent data corruption), and full support for permissions and Docker volumes. However, it’s not natively compatible with macOS or Windows.

---

### 🔌 Steps

1. **Plug in the SSD**
   Connect the SSD to the server via USB or SATA.

---

2. **Identify the device**
   Find the correct device name (example `/dev/sdb`):

   ```bash
   lsblk
   ```

   Look for the drive that matches the SSD size.

---

3. **Delete existing partitions (optional but recommended)**
   If the disk has partitions from another OS (like APFS from macOS):

   ```bash
   sudo fdisk /dev/sdb
   ```

   - Type `d` to delete partitions (repeat if multiple).
   - Type `n` to create a new partition (press Enter to accept defaults).
   - Type `w` to write changes and exit.

---

4. **Format the new partition as ext4**
   Example for `/dev/sdb1`:

   ```bash
   sudo mkfs.ext4 -L SSD /dev/sdb1
   ```

   🔸 `-L SSD` sets a label (optional).

---

5. **Create a mount point**
   Example:

   ```bash
   sudo mkdir -p /mnt/ssd
   ```

---

6. **Mount temporarily (for testing)**

   ```bash
   sudo mount /dev/sdb1 /mnt/ssd
   ```

---

7. **Set permissions** (optional but recommended)
   Give your user ownership:

   ```bash
   sudo chown -R $USER:$USER /mnt/ssd
   ```

   🔸 Or, for open permissions (less secure but useful for personal/home servers):

   ```bash
   sudo chmod 777 /mnt/ssd
   ```

---

8. **Get the UUID for permanent mount**

   ```bash
   sudo blkid /dev/sdb1
   ```

   Example output:

   ```
   /dev/sdb1: UUID="1234-5678-ABCD-XYZ" TYPE="ext4"
   ```

---

9. **Configure `/etc/fstab` for automatic mount**

   Open the file:

   ```bash
   sudo nano /etc/fstab
   ```

   Add this line at the end (replace `UUID=...` with your UUID):

   ```fstab
   UUID=1234-5678-ABCD-XYZ  /mnt/ssd  ext4  defaults  0  2
   ```

---

10. **Test the fstab configuration before reboot**

```bash
sudo umount /mnt/ssd
sudo mount -a
df -h | grep ssd
```

✅ If no errors appear and the SSD is listed, it’s properly configured.

✅ Now the SSD is mounted persistently at `/mnt/ssd` and ready for use in your applications, backups, and Docker volumes.

---

Next: [📡 Static IP setup](./static_ip.md)
