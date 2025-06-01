# 🧰 Infra Setup

A complete guide to setting up a local home server for development, automation, and full-cycle infrastructure experiments using **Ubuntu Server** and **Docker**.

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

> Log out and back in (or run `su <your-user>`) to apply group permissions.

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

## 💾 Mounting External SSD (Persistent)

### 🔌 Steps

1. **Plug in the SSD**

2. **Identify device**:

   ```bash
   lsblk
   ```

3. **Format (if needed)**:

   ```bash
   sudo mkfs.ext4 /dev/sdX1
   ```

4. **Create mount point**:

   ```bash
   sudo mkdir /mnt/ssd
   ```

5. **Mount (temporary)**:

   ```bash
   sudo mount /dev/sdX1 /mnt/ssd
   ```

6. **Change ownership** (optional):

   ```bash
   sudo chown -R $USER:$USER /mnt/ssd
   ```

7. **Get UUID**:

   ```bash
   sudo blkid /dev/sdX1
   ```

8. **Edit `/etc/fstab`**:

   ```bash
   sudo nano /etc/fstab
   ```

   Add:

   ```fstab
   UUID=xxxx-xxxx  /mnt/ssd  ext4  defaults  0  2
   ```

9. **Test and confirm**:

   ```bash
   sudo umount /mnt/ssd
   sudo mount -a
   df -h | grep ssd
   ```

---

[📡 Static IP setup](./static_ip.md)

[🌐 Networking Setup](./networking.md)
