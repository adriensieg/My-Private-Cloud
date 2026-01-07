# My-Private-Cloud
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
