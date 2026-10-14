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

## MCP server (AI assistant)

The chart also runs `free5gc-mcp` (`charts/templates/free5gc-mcp.yaml`, code in `charts/files/free5gc_mcp.py`): the
platform's cluster assistant manages this core's subscribers through MCP. Its tools follow
[free5gc-MCP](https://github.com/q1317540161/free5gc-MCP) (same names and arguments, same default subscriber), ported
to Python and served the way the platform expects an edge app's MCP server:

| Tool | What it does | Approval |
|---|---|---|
| `subscriber_list`, `subscriber_get`, `tenant_users_get` | read | none (read-only) |
| `subscriber_create`, `subscriber_create_multiple` | add subscribers (default profile, free5GC test keys unless `key`/`opc` given) | per call |
| `subscriber_update`, `subscriber_patch` | change a profile (patch merges into the current one) | per call |
| `subscriber_delete`, `subscriber_delete_multiple` | remove subscribers | per call |

- `mcp-manifest.json` (repo root) declares the server and the tools; the platform reads it at the signed commit it
  deploys. `charts/files/mcp-manifest.json` is a byte-identical copy for the chart (keep both the same).
- Only the platform can call it: every request carries the platform's signed, single-use token
  (`X-Platform-Token`, ECDSA P-256, bound to this namespace and the request body). The key arrives through Fleet
  (`fleet.yaml`: `${ .ClusterValues.platformMcpKey }`); without it every call is refused.
- SIM keys (K, OPc, OP) are redacted from every result, so the assistant's model never sees them.
- Not included: the original server's shell, systemd, kubectl and Helm tools. The platform deploys and upgrades free5GC.
- The server logs into the WebConsole (`webui-service:5000`) with `mcp.webconsoleUser` / `mcp.webconsolePassword`
  (a Secret; free5GC's default admin by default). `mcp.enabled: false` removes it.
- Tests: `python3 tests/test_free5gc_mcp.py` (a fake WebConsole, real platform tokens).
