# Certified-Kubernetes-Security-Specialist-Preparation
Certified Kubernetes Security Specialist (CKS) Preparation based on [CNCF](https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist/?utm_source=cncf&amp;utm_medium=blog&amp;utm_campaign=cks0720) preliminary content.

### Approach to this preparation guide
This guide is being authored before actual certification has went live. The best approach to be best prepared for this certification is to know all of the required domains and competences really well. I've copied an available list of competence from [here](https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist/?utm_source=cncf&utm_medium=blog&utm_campaign=cks0720) and plan to enrich each domain with background reading, practical exercises and diagrams (where possible). I do not aim to author 100% of the content for the CKS preparation, instead, I will put effort to put together the best of already available material and only fill-in remaining gaps myself.


# Domains & Competencies

## Cluster Setup – 10%
* Use Network security policies to restrict cluster level access
  * [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) Official documentation - kubernetes.io
  * [Get started with Kubernetes network policy](https://docs.projectcalico.org/security/kubernetes-network-policy) - calico is the leading implementation of Network Policy in kubernetes and comes as default option on managed Kubernetes like GKE. There are 3 tutorials to practice your Network Policy skills. - projectcalico.org
  * [kubernetes-network-policy-recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes) Excellent collection of Network Policy examples by ahmetb - github.com
  * [Exploring Network Policies in Kubernetes](https://banzaicloud.com/blog/network-policy/) - thorough exploration of Netowork Polcies - banzaicloud.com
* Use CIS benchmark to review the security configuration of Kubernetes components (etcd, kubelet, kubedns, kubeapi)
* Properly set up Ingress objects with security control
* Protect node metadata and endpoints
* Minimize use of, and access to, GUI elements
* Verify platform binaries before deploying


## Cluster Hardening – 15%
* Restrict access to Kubernetes API
* Use Role Based Access Controls to minimize exposure
* Exercise caution in using service accounts e.g. disable defaults, minimize permissions on newly created ones
* Update Kubernetes frequently
* System Hardening – 15%
* Minimize host OS footprint (reduce attack surface)
* Minimize IAM roles
* Minimize external access to the network
* Appropriately use kernel hardening tools such as AppArmor, seccomp


## Minimize Microservice Vulnerabilities – 20%
* Setup appropriate OS level security domains e.g. using PSP, OPA, security contexts
* Manage Kubernetes secrets
* Use container runtime sandboxes in multi-tenant environments (e.g. gvisor, kata containers)
* Implement pod to pod encryption by use of mTLS


## Supply Chain Security – 20%
* Minimize base image footprint
* Secure your supply chain: whitelist allowed registries, sign and validate images
* Use static analysis of user workloads (e.g.Kubernetes resources, Docker files)
* Scan images for known vulnerabilities


## Monitoring, Logging and Runtime Security – 20%
* Perform behavioral analytics of syscall process and file activities at the host and container level to detect malicious activities
* Detect threats within physical infrastructure, apps, networks, data, users and workloads
* Detect all phases of attack regardless where it occurs and how it spreads
* Perform deep analytical investigation and identification of bad actors within environment
* Ensure immutability of containers at runtime
* Use Audit Logs to monitor access
