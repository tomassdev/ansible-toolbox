# 🛡️ WireGuard VPN – Ansible Playbook

## 📖 Overview

This Ansible playbook provisions and manages a **WireGuard VPN server** with an **nftables-based firewall**, automated key generation, and optional client configuration. It supports **Debian**, **Ubuntu**, and **Amazon Linux** distributions and is designed to be **fully idempotent**, **secure**, and **cross-distro compatible**.

## ✨ Features

- ⚙️ Installs, configures, and starts WireGuard automatically on Debian, Ubuntu, and Amazon Linux.
- 🧱 Manages nftables firewall with IPv4 NAT and forwarding rules for VPN traffic.
- 🔑 Generates and stores persistent private/public keys for server and clients.
- 🔄 Allows selective key rotation per client.
- 👥 Supports multiple clients with individual IP, DNS, and keepalive configuration.
- 📱 Creates QR codes for easy import on mobile WireGuard applications.
- 🔒 Applies secure defaults: strict file permissions, IPv6 disabled, and `no_log` for secrets.

## 🧩 Requirements

- **Ansible Core:** 2.20+
- **Python:** 3.14+ on target host

## 🚀 Usage

1. **Set up your inventory and variables**

   Define your server connection in [`inventories/hosts.yaml`](inventories/hosts.yaml).

   All WireGuard VPN configuration defaults are already defined in [`group_vars/all.yaml`](inventories/group_vars/all.yaml). See [Configuration Example](#🔧-configuration-example) below.

2. **Run the playbook**

   ```bash
   ansible-playbook main.yaml
   ```

3. **After successful deployment**

   - The VPN server is automatically configured and started.

   - For each defined client, WireGuard configuration and QR code files are generated on the **server** under:
     ```
     /etc/wireguard/clients/<client_name>
     ```
   - These same files are automatically **fetched locally** in:
     ```
     {{ wg_fetch_dir }}/<client_name>
     ```

4. **Connect clients**

   - Import the `client.conf` into the WireGuard desktop app, **or**
   - Scan the `client.png` QR code using the mobile WireGuard app.

5. **Verify connection**

   - Run the [Connectivity Test](#🧪-connectivity-test) checks below to confirm your VPN is working correctly.

## 🔧 Configuration Example

```yaml
wg_interface: wg0
wg_interface_cidr: 10.50.50.1/24
wg_port: 51820
wg_public_address: "{{ ansible_host }}"

wg_clients:
  - name: phone
    interface_cidr: 10.50.50.2/32
    allowed_ips: 0.0.0.0/0
    dns: 1.1.1.1
    persistent_keepalive: 25
    rotate_keys: false
  - name: laptop
    interface_cidr: 10.50.50.3/32
    allowed_ips: 0.0.0.0/0
    dns: 1.1.1.1
    persistent_keepalive: 25
    rotate_keys: false

wg_fetch_dir: ./wg_clients
```

Detailed variable descriptions are documented in [`roles/wireguard/meta/argument_specs.yaml`](roles/wireguard/meta/argument_specs.yaml).

## 🧪 Connectivity Test

After deploying the WireGuard VPN and connecting a client, open one of the following in your browser to verify both your public IP and DNS routing:

- https://ipleak.net
- https://dnsleaktest.com
- https://browserleaks.com/dns

✅ **Expected:**

- The displayed **IP address** should match your VPN server’s public IP (for example, the cloud region where your instance is hosted).
- All detected **DNS servers** should belong to your VPN’s provider or cloud region — not your ISP.
- **IPv6** should be inactive or missing, confirming no IPv6 leaks.

## 🪪 License

Distributed under the **MIT License**. See [LICENSE](../LICENSE) for details.
