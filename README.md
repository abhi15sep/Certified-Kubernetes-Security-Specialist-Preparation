# Certified Kubernetes Security Specialist Preparation
Certified Kubernetes Security Specialist (CKS) Preparation based on [CNCF](https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist/?utm_source=cncf&amp;utm_medium=blog&amp;utm_campaign=cks0720) CKS curriculum.

### Approach to this preparation guide
This guide is being authored before actual certification has went live. The best approach to be best prepared for this certification is to know all of the required domains and competences really well. I've copied an available list of competence from [here](https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist/?utm_source=cncf&utm_medium=blog&utm_campaign=cks0720) and plan to enrich each domain with background reading, practical exercises and diagrams (where possible). I do not aim to author 100% of the content for the CKS preparation, instead, I will put effort to aggregate together the best of already available material and only fill-in remaining gaps myself.


# Domains & Competencies

## Cluster Setup – 10%

* Use Network security policies to restrict cluster level access
  * [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) - Official documentation - *kubernetes.io*
  * [Get started with Kubernetes network policy](https://docs.projectcalico.org/security/kubernetes-network-policy) - calico is the leading implementation of Network Policy in kubernetes and comes as default option on managed Kubernetes like GKE. There are 3 tutorials to practice your Network Policy skills. - *projectcalico.org*
  * [kubernetes-network-policy-recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes) Excellent collection of Network Policy examples by ahmetb - *github.com*
  * [Kubernetes Network Policies Best Practices](https://blog.alcide.io/kubernetes-network-policies-best-practices) - article on Network Policies best practices - *alcide.io*
  * [Exploring Network Policies in Kubernetes](https://banzaicloud.com/blog/network-policy/) - thorough exploration of Netowork Polcies - *banzaicloud.com*

* Use CIS benchmark to review the security configuration of Kubernetes components (etcd, kubelet, kubedns, kubeapi)
  * [CIS Benchmarks](https://www.cisecurity.org/benchmark/kubernetes/) can be found on official CIS website and unfortunately are not available in bite-size form - *cisecurity.org*
  * It is unreasonable to expect examinee to know all of the available (and always growing) CIS Benchmarks. It is likely that exam will expect you to perform CIS benchmark analysis using Open Source tools. One that comes to mind is [kube-bench](https://github.com/aquasecurity/kube-bench#cis-kubernetes-benchmark-support). It is even mentioned on CNCF [Cloud Native Landscape](https://landscape.cncf.io/selected=kube-bench) - *cncf.io*
  * [Alcide Advisor](https://github.com/alcideio/advisor) is another tool that can be run against the cluster to evaluate CIS Benchmarks - *github.com*

* Properly set up Ingress objects with security control
* Protect node metadata and endpoints
* Minimize use of, and access to, GUI elements
* Verify platform binaries before deploying


## Cluster Hardening – 15%

* Restrict access to Kubernetes API
* Use Role Based Access Controls to minimize exposure
  * [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) - Official documentation - *kubernetes.io*
  * Two types of scopes - ClusterRole and Role
    * **ClusterRole** is a powerful, cluster-wide resource, it has to be matched with **ClusterRoleBinding**. Always prefer to use a regular Role, unless you really need a uniform access across all namespaces.
    * **Role** is namespaced resource, meaning that it can only impact permissions within a namespace where it exists. It has to be matched with **RoleBinding**.
  * It is likely that you will be tasked to configure the least privilege RBAC. Things to be mindful of:
* Exercise caution in using service accounts e.g. disable defaults, minimize permissions on newly created ones
* Update Kubernetes frequently

## System Hardening – 15%
* Minimize host OS footprint (reduce attack surface)
* Minimize IAM roles
* Minimize external access to the network
* Appropriately use kernel hardening tools such as AppArmor, seccomp


## Minimize Microservice Vulnerabilities – 20%

* Setup appropriate OS level security domains e.g. using PSP, OPA, security contexts
  * [Pod Security Policy](https://blog.alcide.io/pod-security-policy) - Article on Pod Security Policy - *alcide.io*
* Manage Kubernetes secrets
* Use container runtime sandboxes in multi-tenant environments (e.g. gvisor, kata containers)
* Implement pod to pod encryption by use of mTLS

## Supply Chain Security – 20%

* Minimize base image footprint
  * We want to Minimize attack surface area
    * Do not include build utilities in the final image by using [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)
    ```bash
    FROM golang:1.15.0-alpine3.12
    # Set the working directory
    WORKDIR /go/src/github.com/hugomd/go-example
    # Copy our main file
    COPY main.go .
    # Build the Golang app
    RUN CGO_ENABLED=0 GOOS=linux go build -o app .
    # Create the second stage of our build
    FROM alpine
    WORKDIR /app
    # Copy from the first stage (--from=0)
    COPY --from=0 /go/src/github.com/hugomd/go-example/app .
    CMD ["./app"]
    ```
    * Aim for smaller distributions with least amount of utility bloat (think of using Alpine vs full-sized distributions)
    * Distroless `scratch` images are most secure as they don't add anything but application itself. Modified example of a Go app which now runs distroless. Learn more about [distroless](https://github.com/GoogleContainerTools/distroless).
    ```bash
    FROM golang:1.15.0-alpine3.12
    # Set the working directory
    WORKDIR /go/src/github.com/hugomd/go-example
    # Copy our main file
    COPY main.go .
    # Build the Golang app
    RUN CGO_ENABLED=0 GOOS=linux go build -o app .
    # Create the second stage of our build
    FROM scratch
    WORKDIR /app
    # Copy from the first stage (--from=0)
    COPY --from=0 /go/src/github.com/hugomd/go-example/app .
    CMD ["./app"]
    ```
    * Use `.dockerignore` to avoid pulling in unnecessary files, secrets
    * Don't run image as `root` user. Add least privilege user, group.
    * Use `COPY` instead of `ADD` as `ADD` on URLs can result in arbitrary MITM attacks. Add is also susceptible to [Zip Slip Vulnerability](https://snyk.io/research/zip-slip-vulnerability)
  * [3 simple tricks for smaller Docker images](https://learnk8s.io/blog/smaller-docker-images) Well written explanation on how to minimise size and security risks of your Docker images. - *learnk8s.io*
  * [Docker Image Security Best Practices](https://res.cloudinary.com/snyk/image/upload/v1551798390/Docker_Image_Security_Best_Practices_.pdf) - concise pdf with best practices to follow. *snyk.io*
* Secure your supply chain: whitelist allowed registries, sign and validate images
  * Sign and validate
    * We can use combination of [Kritis Signer](https://github.com/grafeas/kritis/blob/master/docs/signer_install.md) to sign images in CI/CD and [Kritis](https://github.com/grafeas/kritis) to validate that images are signed on admission. Kritis is using [Validating Admission Webhook](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers) as a final arbiter.
    * In GKE, you can use [Authorized Binaries](https://cloud.google.com/solutions/binary-auth-with-cloud-build-and-gke) feature which is a lot simpler to setup and run when compared to Kritis. Shameless plug - [video from Cloud Next 2019](https://youtu.be/0PY0QDNjgJ4) showcasing Binary Authorization.
    * [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper) can be used to build policies for Allowing / Denying images into the cluster. [Conftest](https://github.com/open-policy-agent/conftest) can be used to run Gatekeeper policies in CI/CD and validate registries before it gets to a running cluster.
    * [ImagePolicyWebhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook) can be used to implement validation mechanisms. [kube-image-bouncer](https://github.com/flavio/kube-image-bouncer) is an example of such implementation. Read more about Admission Controllers [here](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/).
* Use static analysis of user workloads (e.g.Kubernetes resources, Docker files)
* Scan images for known vulnerabilities


## Monitoring, Logging and Runtime Security – 20%

* Perform behavioral analytics of syscall process and file activities at the host and container level to detect malicious activities
* Detect threats within physical infrastructure, apps, networks, data, users and workloads
* Detect all phases of attack regardless where it occurs and how it spreads
* Perform deep analytical investigation and identification of bad actors within environment
* Ensure immutability of containers at runtime
  * What does this mean and why does it matter?
  Immutable means that a container won't be modified during its life: no updates, no patches, no configuration changes. If you must update the application code or apply a patch, you build a new image and redeploy it. Immutability makes deployments safer and more repeatable. If you need to roll back, you simply redeploy the old image. This approach allows you to deploy the same container image in every one of your environments, making them as identical as possible. [Source](https://cloud.google.com/solutions/best-practices-for-operating-containers#immutability)
  * Immutability of Volumes (Secrets, ConfigMaps, VolumeMounts) can be achieved with `readOnly: true` field on the mount.
  ```yaml
  volumeMounts:
  - name: cloudsql-instance-credentials
    mountPath: /secrets/cloudsql
    readOnly: true
  ```
* Use Audit Logs to monitor access
  * [Kubernetes Audit: Making Log Auditing a Viable Practice Again.](https://blog.alcide.io/kubernetes-audit-making-log-auditing-a-viable-practice-again) explanation of Kubernetes audit logging - *alcide.io*
