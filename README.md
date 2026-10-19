# Kubernetes Lab on Vagrant + VirtualBox

A two-node Kubernetes cluster (1 control plane + 1 worker) built from scratch with `kubeadm` on Ubuntu 24.04 VMs, with an nginx app exposed through a Traefik Ingress.

## Architecture

```
Browser -> hosts file (nginx.local) -> Worker NodePort 30080
        -> Traefik (Ingress Controller) -> Ingress rule
        -> nginx Service (ClusterIP) -> nginx Pods
```

| Component | Choice |
|---|---|
| Hypervisor / provisioning | VirtualBox + Vagrant |
| OS | Ubuntu 24.04 (`bento/ubuntu-24.04`) |
| Kubernetes | v1.33 via kubeadm |
| Container runtime | containerd (SystemdCgroup) |
| CNI | Flannel (`--iface=eth1`) |
| Ingress | Traefik (Helm, NodePort 30080/30443) |

| VM | IP | CPU | RAM |
|---|---|---|---|
| k8s-master | 192.168.56.101 | 2 | 3 GB |
| k8s-worker | 192.168.56.102 | 2 | 3 GB |

## Prerequisites

- VirtualBox, Vagrant, Git
- ~8 GB free RAM

## Setup

### 1. Create the VMs
```powershell
vagrant up
vagrant ssh master    # and vagrant ssh worker in another terminal
```

### 2. On both nodes
- Disable swap, load `overlay` and `br_netfilter`, enable `net.ipv4.ip_forward`
- Install containerd and set `SystemdCgroup = true`
- Install `kubelet`, `kubeadm`, `kubectl`
- Set the node IP: `KUBELET_EXTRA_ARGS=--node-ip=<node host-only IP>` in `/etc/default/kubelet`

### 3. Master: initialise the cluster
```bash
sudo kubeadm init --apiserver-advertise-address=192.168.56.101 --pod-network-cidr=10.244.0.0/16
```

### 4. Install Flannel
Download `kube-flannel.yml`, add `--iface=eth1` to the flannel container args, then `kubectl apply -f kube-flannel.yml`.

### 5. Worker: join
Run the `kubeadm join ...` command printed by `kubeadm init`.

### 6. Install Traefik
```bash
helm repo add traefik https://traefik.github.io/charts
helm install traefik traefik/traefik -n traefik --create-namespace \
  --set service.type=NodePort \
  --set ports.web.nodePort=30080 --set ports.websecure.nodePort=30443
```

### 7. Deploy nginx
```bash
cd /vagrant/nginx-app
kubectl apply -f namespace.yml
kubectl apply -f .
```

### 8. Access
Add to your hosts file (as Administrator): `192.168.56.102  nginx.local`, then open `http://nginx.local:30080`.

## Lessons learned

- **Vagrant gives every VM the same NAT IP (10.0.2.15).** Kubelet must be told the host-only IP with `--node-ip`, and Flannel needs `--iface=eth1`, otherwise nodes and pods cannot talk to each other.
- **kubelet failed to start** when `/etc/default/kubelet` contained a placeholder instead of a real IP.
- **`ingress-nginx` is retired (March 2026)**, so this lab uses Traefik instead.
- `kubectl` runs on the master; the worker has no kubeconfig.

## Project structure

```
.
├── Vagrantfile
└── nginx-app/
    ├── namespace.yml
    ├── deployment.yml
    ├── service.yml
    └── ingress.yml
```

## Next steps

- Deploy a .NET API container
- ConfigMaps, Secrets, HTTPS (TLS)
- Rolling updates and rollbacks
- Automate node setup with provisioning scripts
