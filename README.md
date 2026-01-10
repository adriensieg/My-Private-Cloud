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
- **SSH**
- **cgroup**
- **Router**
- **Switch**
- **Modem**
- **POE Injector**
- **DHCP**

- **Forward Proxy**: Sits between clients and the internet. Clients connect through it to reach external servers. **Hides client identity**.
- **Reverse Proxy**: Sits between the internet and your servers. External clients connect to it, it forwards to your backend. **Hides server identity/topology**.
- **Cloudflare Tunnel** exposes our cluster to internet
    - Cloudflare Tunnel offers two routing options: **DNS route** and **IP route**

-	Difference between **CloudFare tunnel** vs. **VPN**
  
- **Internet Aware Proxy (IAP)**: Intercepts traffic for multiple protocols, including SSH, Kubernetes, HTTPS, and databases, and ensures that only authenticated clients can connect to target resources.

- **Thread-Safe Backend** - Race condition and deadlock prevention




# Implementation

- [A. Generate **SSH Keys**](https://github.com/adriensieg/My-Private-Cloud/blob/master/README.md#a-generate-ssh-keys)

- [B. Flash Both **MicroSD Cards**](https://github.com/adriensieg/My-Private-Cloud/blob/master/README.md#b-flash-both-microsd-cards)
  
- [C. Configure **TP-Link Travel Router**](https://github.com/adriensieg/My-Private-Cloud/tree/master?tab=readme-ov-file#cconfigure-tp-link-travel-router)
  
- [D. **Physical Assembly**]()

- [E. First **Boot** - Master Node]()
- [F. First **Boot** - Worker Node]()
- [G. **Update** Both Nodes]()
- [H. Install **K3s Master**]()
- [I. Install **K3s Worker**]()
- [J. **Verify** Cluster]()
- [K. Setup Local **kubectl Access**]()
- [L. Deploy **Test Application**]()

### A. Generate **SSH Keys**

1. Generate SSH key
```
ssh-keygen -t ed25519 -C "k3s-cluster" -f ~/.ssh/k3s-cluster
```
2. Display public key (we'll need this)
```
cat ~/.ssh/k3s-cluster.pub
```
3. Copy the entire output - it starts with `ssh-ed25519 AAAA...`

### B. Flash Both MicroSD Cards

##### For Master Node (Card 1)
1. Insert first microSD card into compute
2. Open Raspberry Pi Imager
3. Click "Choose OS" → "Other general-purpose OS" → "Ubuntu" → "Ubuntu Server 22.04.3 LTS (64-bit)"
4. Click "Choose Storage" → Select your microSD card
5. Click ⚙️ gear icon (Advanced Options)
6. Configure:
    - ❌ Uncheck "Set hostname"
    - ❌ Uncheck "Enable SSH" (we'll use cloud-init)
    - ❌ Uncheck "Set username and password"
    - ❌ Uncheck "Configure wireless LAN"
    - ❌ Uncheck "Set locale settings"
7. Click "Save"
8. Click "Write" → Confirm
9. Wait for flashing to complete
10. Do NOT eject yet

##### Modify Cloud-Init for Master (Card 1 for Master Node)
11. Remove and reinsert the microSD card
12. Open the "system-boot" partition (should auto-mount)
13. Find and open file: `user-data`
14. Replace entire contents with:

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

15. Replace `OUR_SSH_PUBLIC_KEY_HERE` with our public key
16. Save and close
17. Edit network-config file:

```YAML
version: 2
ethernets:
  eth0:
    dhcp4: true
    optional: true
```

18. Save and eject card
19. Label card: "MASTER"

##### For Worker Node (Card 2)

20. Insert second microSD card
21. Repeat steps 2-10 above
22. Modify user-data:

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

hostname: adsieg-k3s-node-1

# Enable cgroups
bootcmd:
  - sed -i 's/$/ cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory/' /boot/firmware/cmdline.txt

runcmd:
  - systemctl reboot
```

Replace `OUR_SSH_PUBLIC_KEY_HERE` with our public key
Save and close
Edit network-config (same as master):

```yaml
version: 2
ethernets:
  eth0:
    dhcp4: true
    optional: true
```

26. Save and eject card
27. Label card: **"NODE-1"**

### C.Configure TP-Link Travel Router

##### Physical Setup
1. Plug TP-Link TL-WR902AC into power
2. Wait for WiFi LED to stabilize

##### Connect to Router
3. On your laptop, connect to WiFi: **"TP-LINK_XXXX"** (check router label for exact name)
4. Default password is on the router label
5. Open browser → Go to: **http://tplinkwifi.net** or **192.168.0.1**
6. Login with default credentials (usually admin/admin)

##### Basic Configuration
7. Go to **"Quick Setup"**
8. Select **"Travel Router Mode (Default)"** → Next
9. Choose **"WiFi Connection"**
10. Select your Xfinity home WiFi network
11. Enter your home WiFi password
12. Click **"Next"** → **"Finish"**
13. Wait for router to reboot (2-3 minutes)

##### Reconnect to Travel Router
14. Reconnect to **"TP-LINK_XXXX"** WiFi
15. Login again to **http://tplinkwifi.net**

##### Change Admin Password
16. Go to **"System Tools"** → **"Password"**
17. Set new strong password
18. Save

##### Configure LAN Settings
19. Go to **"Network"** → **"LAN"**
20. Change IP Address to: **10.42.42.1**
21. Subnet Mask: **255.255.255.0**
22. Save
23. Router will reboot
24. Wait 2 minutes

##### Reconnect with New IP
25. Disconnect from WiFi
26. Reconnect to TP-LINK WiFi
27. New router address: **http://10.42.42.1**
28. Login with new password

##### Setup DHCP Reservations
29. Go to **"DHCP"** → **"Address Reservation"**
30. Click **"Add New"**
31. **For Master:**
    - MAC Address: (leave blank for now)
    - Reserved IP: **10.42.42.100**
    - Status: **Enabled**
    - Click **"Save"**
32. Click **"Add New"** again
33. **For Worker:**
    - MAC Address: (leave blank)
    - Reserved IP: **10.42.42.101**
    - Status: **Enabled**
    - Click **"Save"**

### D.Physical Assembly

##### PoE Switch Setup
1. Place PoE switch on flat surface
2. Connect switch to power
3. **Do NOT connect anything yet**

##### Connect Router to Switch
4. Take one Cat6 cable
5. Plug one end into TP-Link Router **LAN port**
6. Plug other end into PoE Switch **port 1**

##### Note on Raspberry Pi Power
⚠️ **Important:** Our Raspberry Pi 4 Model B units **DO NOT support Power over Ethernet** by default. We need:
- Either: Official USB-C power supplies for each Pi
- Or: PoE HAT adapters (hardware accessory that adds PoE capability)

**For this guide, we'll assume you have USB-C power supplies.**



# Tricks

#### When I am on my Mac OS - I can access `user-data`
```
cd /Volumes/system-boot 
code user-data   
code network-config
```

#### My personal Router Web Interface is available under http://10.0.0.1/
<img width="50%" height="50%" alt="image" src="https://github.com/user-attachments/assets/fc9ed6c0-ce6a-4f10-a8d8-e3336fc71a63" />


#### How to authenticate to my raspberry pi

```
ssh -i ~/.ssh/k3s-cluster adsieg@169.254.1.50
ssh -i ~/.ssh/k3s-cluster adsieg@10.0.0.100
```
















