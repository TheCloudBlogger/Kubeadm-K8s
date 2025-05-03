
# Important Kubernetes File Paths and Their Descriptions

## 🔧 Kubernetes Core Files

| File/Directory | Description |
|----------------|-------------|
| `/etc/kubernetes/manifests/` | Static Pod manifest files for core components like kube-apiserver, kube-scheduler, etc. These are managed by kubelet. |
| `/etc/kubernetes/kubelet.conf` | Kubelet configuration file containing its connection info to API server. |
| `/etc/kubernetes/admin.conf` | Admin kubeconfig used by `kubectl` to communicate with API server (contains certs, tokens). |
| `/etc/kubernetes/controller-manager.conf` | Config for the kube-controller-manager. |
| `/etc/kubernetes/scheduler.conf` | Config for the kube-scheduler. |
| `/etc/kubernetes/pki/` | Contains certificates and keys used by various Kubernetes components. |

## 📦 Container Runtime Interface (CRI) Specific Paths

### For `containerd`:
| File/Directory | Description |
|----------------|-------------|
| `/etc/containerd/config.toml` | Main config for containerd, includes CRI plugin configuration, runtime classes. |
| `/var/lib/containerd/` | containerd’s data directory, includes metadata, snapshots, images. |
| `/run/containerd/containerd.sock` | UNIX socket for CRI communication. Kubelet uses this to talk to containerd. |

### For `CRI-O`:
| File/Directory | Description |
|----------------|-------------|
| `/etc/crio/crio.conf` | Main configuration file for CRI-O runtime. |
| `/var/lib/containers/` | Stores images, logs, and CRI-O container state. |
| `/var/run/crio/crio.sock` | UNIX socket for communication with CRI-O. |

### For `Docker` (legacy):
| File/Directory | Description |
|----------------|-------------|
| `/var/lib/docker/` | Docker images, container layers, volumes. |
| `/run/docker.sock` | Docker socket, formerly used by Kubelet. |

## 🧵 Kubelet-specific Files

| File/Directory | Description |
|----------------|-------------|
| `/var/lib/kubelet/` | Stores volumes, pod state, plugins. |
| `/var/lib/kubelet/config.yaml` | Kubelet runtime config (if used). |
| `/var/log/pods/` | Log files of running pods managed by the kubelet. |
| `/var/log/containers/` | Symlinks to container log files. Useful for `kubectl logs`. |

## 🌐 Calico (Container Network Interface - CN)

| File/Directory | Description |
|----------------|-------------|
| `/etc/cni/net.d/` | Contains the CNI configuration file, often named `10-calico.conflist`. |
| `/opt/cni/bin/` | Binary plugins for Calico and other CNIs. |
| `/var/lib/calico/` | Calico node state and information. |
| `/var/run/calico/` | Socket or temporary runtime files. |
| `/etc/calico/calicoctl.cfg` | Optional config file for the Calico CLI tool. |
| Calico Custom Resources (CRDs) | Viewable with `kubectl get bgpconfiguration,ippools,felixconfigurations -A`, and stored in etcd. |

## 📄 etcd

| File/Directory | Description |
|----------------|-------------|
| `/var/lib/etcd/` | Data directory for etcd. |
| `/etc/kubernetes/manifests/etcd.yaml` | Static pod manifest for etcd (in single node setups). |
| `/etc/kubernetes/pki/etcd/` | Certificates used by etcd for client and peer communication. |

## 🛠️ Logging and Audit

| File/Directory | Description |
|----------------|-------------|
| `/var/log/kube-apiserver.log` | Logs for the API server. |
| `/var/log/kubelet.log` | Logs for kubelet (depends on logging system). |
| `/var/log/containers/` | Logs for individual containers (symlinks). |
| `/var/log/pods/` | Logs organized by pod UID. |

## 🔐 Certificate and Token Files

| File/Directory | Description |
|----------------|-------------|
| `/etc/kubernetes/pki/` | Contains all certs and keys for Kubernetes API server, etcd, etc. |
| `/var/run/secrets/kubernetes.io/serviceaccount/` | Token and CA for in-cluster pod communication. Mounted inside pods. |
