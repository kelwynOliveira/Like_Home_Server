# 📦 Automated Backup with Email Notification on Ubuntu Server

This guide explains how to set up:

- ✅ Disk-to-disk backup (`/mnt/ssd` → `/mnt/hdd`) using `rsync`
- ✅ Versioned backup with `rsnapshot`
- ✅ Automatic scheduling via `cron`
- ✅ Email notifications using `Postfix`

---

## 📂 Structure

- **Source:** `/mnt/ssd` (e.g., `/dev/sdb1`)
- **Destination:** `/mnt/hdd` (e.g., `/dev/sdc1`)

---

## ⚙️ 1️⃣ Install dependencies

```bash
sudo apt update
sudo apt install rsync rsnapshot postfix
```

---

## ✉️ 2️⃣ Configure email delivery

During `postfix` installation:

- Select **Internet Site**
- Enter your server’s hostname

To receive cron job output by email, edit your crontab:

```bash
sudo crontab -e
```

Add this line at the top:

```cron
MAILTO="your-email@example.com"
```

---

## 🗂️ 3️⃣ Create a simple backup script (optional)

Create the script:

```bash
sudo nano /usr/local/bin/backup-ssd-to-hdd.sh
```

Script contents:

```bash
#!/bin/bash
rsync -av --delete /mnt/ssd/ /mnt/hdd/
```

`-a` = archive mode (preserves files, permissions, and is recursive)  
`-v` = verbose  
`--delete` = deletes files in the destination that have been removed from the source

Make it executable:

```bash
sudo chmod +x /usr/local/bin/backup-ssd-to-hdd.sh
```

Test it manually:

```bash
sudo /usr/local/bin/backup-ssd-to-hdd.sh
```

---

## ⏰ 4️⃣ Schedule the simple backup with cron

Edit your crontab:

```bash
sudo crontab -e
```

Add:

```cron
0 3 * * * /usr/local/bin/backup-ssd-to-hdd.sh >> /var/log/backup-ssd-to-hdd.log 2>&1
```

This runs the backup daily at **3 AM**.

---

## 🗂️ 5️⃣ Configure versioned backups with `rsnapshot`

Edit the config file:

```bash
sudo nano /etc/rsnapshot.conf
```

Key parts:

```conf
snapshot_root   /mnt/hdd/snapshots/

retain	daily	7
retain	weekly	4
retain	monthly	6

backup	/mnt/ssd/	localhost/
```

**Important:** Use **TABS**, not spaces, between `retain` and values.

Test the config:

```bash
sudo rsnapshot configtest
```

Run a snapshot manually:

```bash
sudo rsnapshot daily
```

---

## ⏰ 6️⃣ Schedule `rsnapshot`

Edit the **root** crontab:

```bash
sudo crontab -e
```

Add:

```cron
0 3 * * * /usr/bin/rsnapshot daily
0 3 * * 1 /usr/bin/rsnapshot weekly
0 3 1 * * /usr/bin/rsnapshot monthly
```

---

## ✅ 7️⃣ Verify everything

- Check logs: `tail -f /var/log/backup-ssd-to-hdd.log`
- Check snapshots: `ls -lh /mnt/hdd/snapshots/`
- Check your email inbox for cron reports
