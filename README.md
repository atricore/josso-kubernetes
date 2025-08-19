# JOSSO Kubernetes Base

This project provides Kubernetes base configurations for JOSSO deployments using Kustomize. It supports two deployment flavors:

## Deployment Flavors

### 1. Single Server (`single-server.yaml`)
- Uses a Kubernetes Deployment
- Single replica for simple deployments
- Standard ClusterIP service
- Suitable for development, testing, or small production environments

### 2. HA Cluster (`ha-cluster.yaml`)
- Uses a Kubernetes StatefulSet
- Multiple replicas with stable network identities
- Headless service for peer discovery
- Automatic peer discovery via initContainer and environment variables
- Suitable for production high-availability deployments

## Project Structure

```
base/
├── common/                    # Shared resources
│   ├── secrets/              # Secret configuration
│   ├── console/              # JOSSO Console StatefulSet
│   └── kustomization.yaml
├── flavors/
│   ├── single-server/        # Single instance deployment
│   └── ha-cluster/          # HA cluster deployment
├── single-server.yaml       # Single server flavor base
├── ha-cluster.yaml          # HA cluster flavor base
└── kustomization.yaml       # Common resources only
```

## Usage

### For Single Server Deployment

```yaml
# your-env/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base/single-server.yaml

# Add your customizations here
```

### For HA Cluster Deployment

```yaml
# your-env/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base/ha-cluster.yaml

# Customize replica count
patches:
  - target:
      kind: StatefulSet
      name: josso-server
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
```

## HA Cluster Peer Discovery

The HA cluster flavor uses an initContainer strategy for peer discovery:

1. **StatefulSet**: Provides stable hostnames (`josso-server-0`, `josso-server-1`, etc.)
2. **Headless Service**: Enables DNS-based peer discovery
3. **initContainer**: Queries DNS to build peer list and sets `JOSSO_PEERS` environment variable
4. **Shared Volume**: Passes peer configuration from initContainer to main container

The peer discovery script automatically:
- Queries the headless service DNS for all pod IPs
- Formats the peer list as `IP:PORT,IP:PORT,...`
- Makes the peer list available to JOSSO via the `JOSSO_PEERS` environment variable

## Examples

- `envs/local/`: Single server deployment example
- `envs/ha-example/`: HA cluster deployment example with 5 replicas

## Configuration

Update secrets in `base/common/secrets/secrets.txt`:
- `josso_client_id`: Your JOSSO client ID
- `josso_client_secret`: Your JOSSO client secret  
- `josso_admin_usr`: Admin username
- `josso_admin_pwd`: Admin password