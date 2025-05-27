# Organization & standards

**Cloud Native Computing Foundation (CNCF)**
   - **Role:** The CNCF (which governs Kubernetes) provides best practices for production-grade Kubernetes.
   - **Key Resources:**
     - [Kubernetes Production-Grade Checklist](https://kubernetes.io/docs/setup/production-environment/) (official docs)
     - [Certified Kubernetes Conformance Program](https://www.cncf.io/certification/software-conformance/) (ensures compatibility)
   - **Example:** A CNCF-certified Kubernetes distribution (e.g., EKS, GKE, OpenShift) meets baseline production standards.


**CIS (Center for Internet Security) Kubernetes Benchmarks**
   - **Role:** Defines **security hardening** standards for production Kubernetes clusters.
   - **Key Resource:** [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes/)
   - **Example:** Tools like `kube-bench` audit clusters against CIS standards.


**NIST (National Institute of Standards and Technology)**
   - **Role:** Provides security guidelines for container orchestration (including Kubernetes).
   - **Key Resource:** [NIST SP 800-190 (Application Container Security)](https://csrc.nist.gov/publications/detail/sp/800-190/final)
   - **Example:** Recommends pod security policies (now replaced by PSA/PSS in K8s 1.25+).



**PCI DSS (Payment Card Industry Data Security Standard)**
   - **Role:** If Kubernetes handles payment data, PCI DSS compliance is required.
   - **Key Resource:** [PCI DSS v4.0](https://www.pcisecuritystandards.org/) (Section 6.3.4 for containers)
   - **Example:** Isolating payment processing pods with network policies.

**SOC 2 (Service Organization Control)**
   - **Role:** Audits Kubernetes clusters for security, availability, and confidentiality.
   - **Key Resource:** [AICPA SOC 2 Framework](https://www.aicpa.org/)
   - **Example:** Managed Kubernetes services (EKS, GKE) often provide SOC 2 reports.

**ISO/IEC 27001**
   - **Role:** International standard for information security management (applies to K8s).
   - **Key Resource:** [ISO 27001 Controls](https://www.iso.org/isoiec-27001-information-security.html)
   - **Example:** Encrypting `etcd` data and enabling audit logging.

**DoD (U.S. Department of Defense) STIGs**
   - **Role:** Security Technical Implementation Guides for federal/government Kubernetes use.
   - **Key Resource:** [DoD Kubernetes STIG](https://public.cyber.mil/stigs/)
   - **Example:** Disabling anonymous authentication in `kube-apiserver`.

**FinTech & Banking Regulators (e.g., FFIEC, GDPR)**
   - **Role:** Industry-specific rules for financial services or EU data protection.
   - **Example:** Using Kubernetes namespaces to isolate GDPR-regulated workloads.


