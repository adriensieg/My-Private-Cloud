# On the road of my private Cloud
Building a bare-metal Kubernetes cluster on Raspberry Pis

# Hardwares

- [2x Raspberry Pi 4 Model B (8GB RAM)](https://www.amazon.com/dp/B08957PMS2?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_2&th=1)
- 2x microSD cards (32GB+ recommended)
- [Cat6 Ethernet cables (5-pack)](https://www.amazon.com/gp/product/B0CSS9VS3S/ref=ox_sc_act_title_1?smid=A17K4O9J62HKE5&psc=1)
- [Cluster Case for Raspberry Pi](https://www.amazon.com/gp/product/B07D5MJ7PQ/ref=ox_sc_act_title_2?smid=A3R613EW5HTD95&th=1)
- [TP-Link TL-SG1005P PoE Switch](https://www.amazon.com/gp/product/B076HZFY3F/ref=ox_sc_act_title_4?smid=ATVPDKIKX0DER&th=1)
- [TP-Link AC750 Travel Router (TL-WR902AC)](https://www.amazon.com/gp/product/B01N5RCZQH/ref=ox_sc_act_title_3?smid=ATVPDKIKX0DER&psc=1)
- Xfinity Arris TG1682P home router


# Concepts
- **Forward Proxy**: Sits between clients and the internet. Clients connect through it to reach external servers. **Hides client identity**.
- **Reverse Proxy**: Sits between the internet and your servers. External clients connect to it, it forwards to your backend. **Hides server identity/topology**.
- **Cloudflare Tunnel** exposes our cluster to internet

# Implementation

A. [Generate **SSH Keys**]()
B. [Flash Both **MicroSD Cards**]
    - For Master Node (Card 1)
    - For Worker Node (Card 2)
C. [Configure **TP-Link Travel Router**]
    - Setup DHCP Reservations
D. [**Physical Assembly**]
    - PoE Switch Setup
    - Connect Router to Switch
E. [First **Boot** - Master Node]
F. [First **Boot** - Worker Node]
G. [**Update** Both Nodes]
H. [Install **K3s Master**]
I. [Install **K3s Worker**]
J. [**Verify** Cluster]
K. [Setup Local **kubectl Access**]
L. [Deploy **Test Application**]

#### A. Generate **SSH Keys**

1. Generate SSH key
```
ssh-keygen -t ed25519 -C "k3s-cluster" -f ~/.ssh/k3s-cluster
```
2. Display public key (we'll need this)
```
cat ~/.ssh/k3s-cluster.pub
```
3. Copy the entire output - it starts with `ssh-ed25519 AAAA...`

#### B. Flash Both MicroSD Cards

##### For Master Node (Card 1)
1. Insert first microSD card into compute
2. Open Raspberry Pi Imager
3. Click "Choose OS" → "Other general-purpose OS" → "Ubuntu" → "Ubuntu Server 22.04.3 LTS (64-bit)"
4. Click "Choose Storage" → Select your microSD card
5. Click ⚙️ gear icon (Advanced Options) - Configure:
    - ❌ Uncheck "Set hostname"
    - ❌ Uncheck "Enable SSH" (we'll use cloud-init)
    - ❌ Uncheck "Set username and password"
    - ❌ Uncheck "Configure wireless LAN"
    - ❌ Uncheck "Set locale settings"
6. Click "Save"
7. Click "Write" → Confirm
8. Wait for flashing to complete
9. Do NOT eject yet

##### Modify Cloud-Init for Master (Card 1 for Master Node)
10. Remove and reinsert the microSD card
11. Open the "system-boot" partition (should auto-mount)
12. Find and open file: `user-data`
13. Replace entire contents with:

```yaml
#cloud-config
ssh_pwauth: false

groups:
  - ubuntu: [root, sys]

users:
  - default
  - name: adsieg
    gecos: adsieg
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo
    lock_passwd: true
    shell: /bin/bash
    ssh_authorized_keys:
      - YOUR_SSH_PUBLIC_KEY_HERE

hostname: adsieg-k3s-master

# Enable cgroups
bootcmd:
  - sed -i 's/$/ cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory/' /boot/firmware/cmdline.txt

runcmd:
  - systemctl reboot
```
