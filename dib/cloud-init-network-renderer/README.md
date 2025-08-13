Configures cloud-init to prefer NetworkManager for network configuration.

This element writes a drop-in at `/etc/cloud/cloud.cfg.d/90-network-manager-renderer.cfg` with:

```yaml
system_info:
  network:
    renderers: ['network-manager', 'sysconfig', 'eni', 'netplan', 'networkd']
```

This ensures cloud-init uses NetworkManager when available, with `sysconfig` and `netplan` etc as fallbacks.

The renderer list/order can be customized by setting `DIB_CLOUD_INIT_NETWORK_RENDERERS` to a YAML array format, e.g.:
```bash
export DIB_CLOUD_INIT_NETWORK_RENDERERS="['netplan', 'sysconfig', 'network-manager']"
```
