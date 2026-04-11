# fleet-free5gc

Fleet bundle to deploy free5GC 5G core on K3s clusters via Rancher Fleet.

## Dependencies (must be deployed first)
- `gtp5g-installer` — kernel module for GTP-U tunneling
- `multus-cni` — secondary network interfaces for UPF

## Network interfaces (all on enp1s0 via ipvlan)
| Interface | Subnet | Purpose |
|-----------|--------|---------|
| N2 | 10.100.50.248/29 | AMF ↔ gNB (NGAP) |
| N3 | 10.100.50.232/29 | UPF ↔ gNB (GTP-U) |
| N4 | 10.100.50.240/29 | UPF ↔ SMF (PFCP) |
| N6 | 10.100.100.0/24  | UPF ↔ Data Network |
| N9 | 10.100.50.224/29 | UPF ↔ UPF (inter-UPF) |

## Add to Fleet
Rancher → Fleet → Git Repos → Add Repository:

| Field          | Value                                        |
|----------------|----------------------------------------------|
| Name           | `fleet-free5gc`                              |
| Repository URL | `https://github.com/teo-tsou/fleet-free5gc`  |
| Branch         | `main`                                       |
| Paths          | `/`                                          |

## Verify deployment
```bash
kubectl get pods -n free5gc
kubectl get network-attachment-definitions -n free5gc
```
