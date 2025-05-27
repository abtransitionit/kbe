# KBE  
**K**u**B**ernetes **E**asy is a CLI (**C**ommand **L**ine **I**nterface) powered by [LUC](https://github.com/abtransitionit/luc). It creates and manages Kubernetes Clusters (on a set of VMs). The cluster is built with a focus on **security**, **observability**, and **network-level debugging control**.

[![LICENSE](https://img.shields.io/badge/license-Apache_2.0-blue.svg)](https://choosealicense.com/licenses/apache-2.0/) 
[![Cilium](https://img.shields.io/badge/Cilium-eBPF%20Networking-blue)](https://cilium.io)
[![CRI-O](https://img.shields.io/badge/CRI--O-Container%20Runtime-red)](https://cri-o.io) 
[![nftables](https://img.shields.io/badge/nftables-Packet%20Filtering-orange)](https://wiki.nftables.org) 
[![Ingress-NGINX](https://img.shields.io/badge/Ingress--NGINX-K8s%20Ingress%20Controller-success)](https://kubernetes.github.io/ingress-nginx)

---

# Component Stack

The Kubernetes cluster is composed of modular, production-grade components, each chosen for its reliability, compatibility, and role in achieving a secure and observable infrastructure. The table below summarizes the key building blocks and their core functions:

| Layer              | Component    | Description / Value                                                                    |
| ------------------ | ------------ | -------------------------------------------------------------------------------------- |
| Container Runtime  | **CRI-O**    | Lightweight, OCI-compliant runtime optimized for Kubernetes; secure by design          |
| CNI Plugin         | **Cilium**   | eBPF-based plugin offering high-performance networking and fine-grained policy control |
| Ingress Controller | **NGINX**    | Acts as an edge router for services; deployed in NodePort mode for cluster access      |
| Host Firewall      | **nftables** | Modern (low-level) packet filtering framework; compatible with all linux distribution|
| Cluster Init Tool  | `kubeadm`    | Bootstraps the cluster with custom init file (eg. defined Pod/Service CIDRs, ...)  for predictable networking|

---



# Access and Security Model

## Host-Level Security

* **nftables** is configured on each node
* Only `o1` (control plane) is accessible externally, restricted to SSH (port 22)
* Worker nodes are **fully isolated** from external networks

## External Access Security strategy

The cluster follows a **controlled and minimal exposure** model for external access:

- **SSH Access**

  - Only the **control plane**  (node `o1`) is accessible via **SSH (port 22)** from the trusted **cockpit VM**.
  - **Worker nodes** (`o2` .. `o5`))** have **no SSH access** from outside.

- **Ingress Access**

  - **Port `8443`** is the **only publicly accessible port**, open on **all nodes**.
  - This is required because **ingress controller pods may be scheduled on any node**.
  - The port is tightly restricted by **nftables** to limit exposure to ingress traffic only.

- **General External Access**

  * No other public-facing services or ports are exposed.
  * **Worker nodes** have **no direct internet access**, except for the controlled ingress on port `8443`.


---


# Cluster Initialization (`kubeadm`)

The cluster is bootstrapped using `kubeadm` with explicitly defined **Pod** and **Service** custom CIDRs for enhanced network control:

```yaml
networking:
  podSubnet: 192.168.0.0/16
  serviceSubnet: 172.16.0.0/16
```

Benefits of this setup:

- Defines custom CIDR for pods and servcies to simplify debugging of **inter-pod** and **service** traffic flows
- Ensures **Cilium** integrates seamlessly with predictable IP ranges
- Avoids conflicts with default network ranges on external systems

---


---

# Getting Started  

## Create a cluster
- Prerequisite: having SSH access to a set of VMS.  This access is configured via a configuration file in `~/.config.d/`
- create a `yaml` file to configure the cluster
- create the cluster

```bask
kbe create runall --force
```

---

# Contributing  

We welcome contributions! Before participating, please review:  
- **[Code of Conduct](.github/CODE_OF_CONDUCT.md)** – Our community guidelines.  
- **[Contributing Guide](.github/CONTRIBUTING.md)** – How to submit issues, PRs, and more.  


----


# Release History & Changelog  

Track version updates and changes:  
- **📦 Latest Release**: `vX.X.X` ([GitHub Releases](#))  
- **📄 Full Changelog**: See [CHANGELOG.md](CHANGELOG.md) for detailed version history.  

---
----

