# My-Private-Cloud
Building a bare-metal Kubernetes cluster on Raspberry Pis

# Hardwares

- [2x Raspberry Pi 4 Model B (8GB RAM)](https://www.amazon.com/dp/B08957PMS2?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_2&th=1)
2x microSD cards (32GB+ recommended)
Cat6 Ethernet cables (5-pack)
TP-Link TL-SG1005P PoE Switch
TP-Link AC750 Travel Router (TL-WR902AC)
Xfinity Arris TG1682P home router


# Concepts
- **Forward Proxy**: Sits between clients and the internet. Clients connect through it to reach external servers. **Hides client identity**.
- **Reverse Proxy**: Sits between the internet and your servers. External clients connect to it, it forwards to your backend. **Hides server identity/topology**.
- **Cloudflare Tunnel** exposes our cluster to internet
