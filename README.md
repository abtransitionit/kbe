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

## 🖥️ Infrastructure Layout

Example of an architecture layout of a Kubernetes layout with 5 nodes.

| VMs/Nodes | SSH Hostname | Role          | External Access                      |
| ---- | -------- | ------------- | ------------------------------------ |
| o1   | `o1u`     | Control Plane | ✅ SSH (port 22) from cockpit VM only |
| o2   | `o2a`     | Worker Node   | ❌ No direct external access          |
| o3   | `o3r`     | Worker Node   | ❌ No direct external access          |
| o4   | `o4d`     | Worker Node   | ❌ No direct external access          |
| o5  | `o5f`     | Worker Node   | ❌ No direct external access          |

* Nodes are provisioned as VMs on **OVH**
* Full **internal network communication** is permitted between all nodes

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

# Test Environment

This section documents the **operating systems**, **kernel versions**, **tooling**, and **core configuration** used to validate the cluster.  It ensures transparency, reproducibility, and provides a reference baseline for future deployments.


## Tested Operating Systems

The Kubernetes cluster was tested on the following Linux distributions and kernel versions:


| VM/Node    | OS Family | Distro     | Version | Kernel Version                          |
|-------|-----------|------------|---------|------------------------------------------|
| o1u   | Debian    | Ubuntu     | 24.10   | 6.11.0-25-generic                         |
| o2a   | RHEL      | AlmaLinux  | 9.5     | 5.14.0-503.40.1.el9_5.x86_64              |
| o3r   | RHEL      | Rocky      | 9.5     | 5.14.0-503.40.1.el9_5.x86_64              |
| o4d   | Debian    | Debian     | 12.10   | 6.1.0-34-cloud-amd64                      |
| o6f   | Fedora    | Fedora     | 40      | 6.14.5-100.fc40.x86_64                    |


## Tooling Versions

The Kubernetes cluster was deployed and verified using the following tool versions to ensure compatibility and stability across components:


| Tool / Component                  | Version   | Notes                                                                 |
|----------------------------------|-----------|-----------------------------------------------------------------------|
| Kubernetes                       | `1.32.0`  | Main cluster version                                                  |
| CRI-O (via `dnf`/`apt`)          | `v1.32`   | Matches Kubernetes major/minor version                                |
| Cilium (Helm chart)              | `1.17`    | Compatible with Kubernetes `1.32.x` – `1.33.x`                         |
| Ingress NGINX (Helm chart)       | `4.12`    | Stable release for Kubernetes `1.32+`; deployed in NodePort mode      |


## Additional setup Configuration

The table below summarize the Kubernetes cluster component, setup and init configuration:

| Feature             | Value                           |
| ------------------- |---------------------------------|
| Runtime             | CRI-O                   |
| CNI Plugin          | Cilium                  |
| Ingress             | NGINX (NodePort)        |
| Firewall            | nftables                |
| Pod CIDR            | `192.168.0.0/16` |
| Service CIDR        | `172.16.0.0/16`  |
| Control Plane  | `o1`             |
| Worker Nodes        | `o2` ... `o5`|
| External Entrypoint | Cockpit VM → `o1` (SSH) |








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

## Future Enhancements

* [ ] Introduce a **LoadBalancer** (e.g., MetalLB or external reverse proxy)
* [ ] **Deploy MetalLB** as the preferred LoadBalancer solution for internal and external service exposure, enabling more robust and flexible ingress patterns.
* [ ] Fine-tune firewall rules to allow only essential inter-node ports
* [ ] **Fine-tune firewall rules** to:

  * Allow **only essential ports** for **Kubernetes control**, **Cilium**, and **ingress** communication between nodes.
  * **Explicitly block all other inter-node traffic**, even internally.
  * Follow **Cilium’s best practices** to permit only necessary ports such as:

    * `TCP 6443`: Kubernetes API
    * `TCP/UDP 8472`: VXLAN (if in use)
    * `TCP 2379-2380`: etcd (control plane)
    * `TCP 8443`: ingress controller
* [ ] **Harden ingress exposure**:

  * Public access is restricted to **port `8443`** only, across **all nodes**.
  * This port is required since **ingress pods may run on any node**.
  * All other ports are **firewalled externally** and internally, unless explicitly required.
* [ ] Enable **TLS termination** at the ingress layer for secure HTTP(S)




