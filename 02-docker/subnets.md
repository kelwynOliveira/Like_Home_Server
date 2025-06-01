# 🛠️ Fix: Running Out of Usable Subnets in Docker

Avoid IP exhaustion in Docker when deploying multiple stacks via Portainer or Docker Compose. This guide shows how to customize Docker’s default address pool to use smaller, manageable subnets — keeping your local networks clean and under control.

---

## 📚 References

- [Video: Quick Tips - Don’t Run Out of IPs in Docker](https://www.youtube.com/watch?v=rxZh8DXWrHw)
- Docker Docs: [Configure the Docker daemon](https://docs.docker.com/config/daemon/)

---

## 🤔 Problem

By default, Docker assigns each new network a `/16` subnet — meaning **65,536 IPs per network**. If you're frequently deploying stacks (especially via Portainer), you'll quickly run out of usable subnets in the `172.x.x.x` private range.

If you're frequently using Docker — especially with **Portainer** or **Docker Compose** — you might run into an error like:

> `ERROR: could not find an available, non-overlapping IPv4 address pool among the defaults to assign to the network`

This happens because Docker, by default, creates **very large internal networks** (using `/16` subnets). Over time, as more stacks or containers are deployed, you may **run out of usable subnet space**, even if you're only working locally.

## ❓ Why Does This Happen?

Docker assigns a default IP range for internal networks. Each time a new network is created (e.g., when deploying a new stack), Docker allocates a **large block** of IPs — up to **65,536 addresses per network** using a `/16` subnet.

That’s overkill for most local setups.

As a result:

- You quickly exhaust the available private IP ranges.
- Docker can’t find free space for new networks.
- You get frustrating IP-related errors.

---

## 💡 Solution

We can tell Docker to use **smaller networks** by default — for example, `/24` subnets (256 IPs each) — which is more than enough for most use cases.

To do this, we configure the `default-address-pools` setting in Docker's configuration file (`daemon.json`).

This change lets you deploy hundreds of isolated networks without IP conflicts or exhaustion.

---

## 🛠️ Step-by-Step Instructions

1. **SSH into your Docker server**:
   If you're working locally, just open a terminal. If you're working on a remote server (e.g. a home lab or VPS), connect to it using SSH:

   ```bash
   ssh your-user@your-server
   ```

2. **Edit the Docker daemon configuration**:
   Open Docker's configuration file:

   ```bash
   sudo nano /etc/docker/daemon.json
   ```

3. **Paste the following config** (customize `base` if needed):
   Add (or replace) the following content

   ```json
   {
     "default-address-pools": [
       {
         "base": "172.30.0.0/16",
         "size": 24
       }
     ]
   }
   ```

   Make sure the base IP block you choose is not already in use by any running containers or your LAN network.
   ✅ This tells Docker: "Use the `172.30.x.x` range, in `/24` chunks (256 IPs per subnet)."

   > 🔍 Explanation:
   >
   > - base: The starting private IP range for Docker to use.
   >
   > - size: Each new network will now be a /24 (256 IPs), instead of the default /16 (65k IPs).

4. **Restart Docker**:
   Apply the changes by restarting Docker:

   ```bash
   sudo systemctl restart docker
   ```

   Note: You might get logged out from Portainer or lose existing networks — redeploying them usually solves that.

5. **Verify the change**:

   Create a new Docker stack or compose file. Check the assigned subnet:

   ```bash
   docker network ls
   docker network inspect your_network_name
   ```

   Or you can deploy a new stack (e.g. via Portainer) and inspect the network. You should see the subnet looking like:

   ```
     172.30.0.0/24
     172.30.1.0/24
     172.30.2.0/24
     ...
   ```

   Each new network gets its own /24 block, and you now have 255+ smaller subnets before IPs run out.

---

## 🔍 Example Output

Creating multiple stacks will now increment the third octet (`172.30.1.0/24`, `172.30.2.0/24`, etc.), avoiding exhaustion of the 172.16.0.0/12 range.

---

## ✅ Benefits

- Prevents exhaustion of Docker's private subnet space
- Avoids overlapping network issues in large setups
- Keeps things clean when using **Portainer**, **Docker Compose**, or **Stacks**

---

## 📌 Notes

- Ensure the `base` IP (`172.30.0.0/16`) doesn’t conflict with existing Docker networks.
- You can define **multiple address pools** if needed.
- If you get errors after restarting Docker, double-check your JSON syntax and pool conflicts using `docker network ls`.

---

## 🎯 When Should You Do This?

- You run many Docker stacks or containers (especially via Portainer).
- You want to avoid subnet collision or IP exhaustion.
- You're building a local server, homelab, or development environment.
