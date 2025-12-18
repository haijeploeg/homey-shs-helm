# Homey Self-Hosted Helm Chart

Deploy [Homey Self-Hosted](https://homey.app/en-us/homey-self-hosted-server/) in Kubernetes.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- PersistentVolume provisioner
- (Optional) MetalLB for LoadBalancer support

## Installation

### Add Helm Repository

```bash
helm repo add homey-shs https://haijeploeg.github.io/homey-shs-helm
helm repo update
```

### Install

```bash
# Basic installation
helm install homey homey-shs/homey-shs

# With custom values
helm install homey homey-shs/homey-shs -f values.yaml
```

### Install from Source

```bash
git clone https://github.com/haijeploeg/homey-shs-helm.git
cd homey-shs-helm
helm install homey .
```

### Example: MetalLB Configuration

```yaml
service:
  type: LoadBalancer
  annotations:
    metallb.universe.tf/loadBalancerIPs: 192.168.1.100
```

```bash
helm install homey homey-shs/homey-shs -f values.yaml
```

## Configuration

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `hostNetwork` | Use host network namespace | `false` |
| `securityContext.capabilities` | Required capabilities | `[NET_ADMIN, NET_RAW]` |
| `resources.requests.memory` | Memory request | `1Gi` |
| `resources.limits.memory` | Memory limit | `2Gi` |
| `persistence.size` | Storage size | `5Gi` |
| `persistence.storageClassName` | Storage class (empty = default) | `""` |
| `service.type` | Service type | `LoadBalancer` |
| `service.annotations` | Service annotations (e.g., MetalLB) | `{}` |
| `podDisruptionBudget.enabled` | Enable PDB | `false` |
| `ingress.enabled` | Enable ingress | `false` |

See [values.yaml](values.yaml) for all available options.

### Example Configuration

```yaml
resources:
  requests:
    memory: 2Gi
  limits:
    memory: 4Gi

persistence:
  size: 10Gi
  storageClassName: fast-ssd

service:
  annotations:
    metallb.universe.tf/loadBalancerIPs: 192.168.1.50

podDisruptionBudget:
  enabled: true

nodeSelector:
  kubernetes.io/hostname: home-server
```

## Security

This chart runs as **root** with `NET_ADMIN` and `NET_RAW` capabilities for:
- **dbus-daemon** - System message bus
- **avahi-daemon** - mDNS/Bonjour service discovery
- **rrdcached** - Database caching

These minimal capabilities are required and significantly more secure than privileged mode.

## Upgrading

```bash
helm repo update
helm upgrade homey homey-shs/homey-shs -f values.yaml
```

## Uninstallation

```bash
helm uninstall homey
```

⚠️ PersistentVolumeClaim is not automatically deleted:

```bash
kubectl delete pvc data-homey-0
```

## Troubleshooting

### Avahi Daemon Crashes
Logs show `Service:avahi-daemon Exited with code 255`:
- Verify `NET_ADMIN` and `NET_RAW` capabilities
- Try `hostNetwork: true`
- Check cluster network policy allows multicast

### Permission Denied
- Ensure container runs as root (no `runAsUser`)
- Verify capabilities are set
- Check PersistentVolume permissions

### Service Not Accessible
```bash
kubectl get svc homey
kubectl logs homey-0
```
- Verify MetalLB/LoadBalancer configuration
- Check firewall allows ports 4859-4862
- Increase `livenessProbe.initialDelaySeconds` if needed

## Publishing to GitHub Pages

To publish this chart to GitHub Pages:

1. **Package the chart:**
   ```bash
   helm package .
   ```

2. **Create/update index:**
   ```bash
   helm repo index . --url https://haijeploeg.github.io/homey-shs-helm
   ```

3. **Push to GitHub:**
   ```bash
   git add .
   git commit -m "Release chart version X.X.X"
   git push origin main
   ```

4. **Enable GitHub Pages** in repository settings (Settings → Pages → Source: main branch / root)

## Contributing

Contributions welcome! Please submit a Pull Request at [github.com/haijeploeg/homey-shs-helm](https://github.com/haijeploeg/homey-shs-helm).

## License

This Helm chart is open source. Homey Self-Hosted is © Athom B.V.
