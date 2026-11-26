# Final Kubernetes Security Assessment

**Project:** Kubernetes Security: Vulnerability Assessment and Hardening
**Author:** Madhur Kanoje
**Environment:** Kali Linux, Kind, Kubernetes v1.34.0
**Networking:** Calico v3.33.0
**Runtime Security:** Falco v0.45.2
**Container Runtime:** containerd 2.1.3
**Assessment Date:** 26 November 2026
**Project Status:** M1–M6 Completed

---

## 1. Executive Summary

This project evaluates security weaknesses in a Kubernetes environment and demonstrates practical security controls for reducing risks associated with containerized workloads.

A local Kubernetes cluster was created using Kind. Calico was used for container networking and NetworkPolicy enforcement, while Falco was deployed for runtime threat detection.

The assessment focused on five primary security areas:

1. Kubernetes network security.
2. Role-Based Access Control (RBAC).
3. Pod Security Admission (PSA).
4. Container workload hardening.
5. Runtime threat detection using Falco.

Controlled security tests were conducted against vulnerable and hardened configurations.

The results demonstrated that:

- NetworkPolicies restricted unauthorized pod communication.
- Least-privilege RBAC reduced unnecessary permissions.
- Pod Security Admission rejected non-compliant privileged workloads.
- Hardened workloads operated with reduced container privileges.
- Falco detected selected suspicious process and file-access activities.

The project completed milestones M1 through M6, including security testing and evidence collection.

## 2. Project Environment

### 2.1 Infrastructure

| Component | Configuration |
|---|---|
| Operating System | Kali Linux |
| Kubernetes Platform | Kind |
| Kubernetes Server | v1.34.0 |
| Cluster Name | calico-cluster |
| CNI | Calico v3.33.0 |
| Runtime Security | Falco v0.45.2 |
| Container Runtime | containerd 2.1.3 |
| Cluster Node | calico-cluster-control-plane |

### 2.2 Security Tools

- kubectl: Kubernetes cluster administration and testing.
- Kind: Local Kubernetes cluster deployment.
- Calico: Container networking and NetworkPolicy enforcement.
- Falco: Runtime security event detection.
- Linux shell: Controlled security testing.
- Git and GitHub: Source code and evidence management.

## 3. Assessment Methodology

The assessment followed a controlled vulnerability assessment and hardening process.

The methodology consisted of:

1. Establishing a Kubernetes test environment.
2. Identifying insecure workload and access configurations.
3. Creating vulnerable and hardened configurations.
4. Applying security controls.
5. Executing controlled security tests.
6. Verifying expected and observed behavior.
7. Collecting logs and test evidence.
8. Documenting security findings and recommendations.

The tests were performed in a local Kubernetes laboratory environment.

The results represent the observed behavior of the tested configurations and do not constitute a complete security certification of Kubernetes.

## 4. Network Security Assessment

### 4.1 Objective

Evaluate Kubernetes pod-to-pod communication controls using Calico NetworkPolicies.

### 4.2 Configuration

Namespace: `k8s-net-test`

Test workloads:

- `net-client`
- `net-server`
- `unauthorized-client`

The server exposed TCP port 8080.

Two NetworkPolicies were used:

- `deny-server-ingress`: isolates the server from unsolicited ingress.
- `allow-client-to-server`: permits TCP port 8080 from pods labelled `app=net-client`.

### 4.3 Test Results

| Test | Expected Behavior | Observed Result |
|---|---|---|
| Authorized client to server TCP 8080 | Allow | Successful |
| Unauthorized client to server TCP 8080 | Block | Timed out |
| Authorized client to server TCP 8081 | Block | Timed out |

The authorized client received:

`Kubernetes networking test successful`

The unauthorized client and unauthorized port tests returned connection timeouts.

### 4.4 Finding

**Finding: Unrestricted pod communication can expose internal Kubernetes services.**

The implemented NetworkPolicies restricted ingress to the selected client and port.

### 4.5 Assessment

The recorded tests confirmed the expected NetworkPolicy enforcement behavior in the test namespace.

Evidence:

`evidence/m6-network/network-policy-test-results.txt`

Additional hardened workload assessment:

`reports/network-workload-hardening-20261110.md`

## 5. RBAC Security Assessment

### 5.1 Objective

Evaluate excessive Kubernetes permissions and implement least-privilege access.

### 5.2 Vulnerable Configuration

A controlled vulnerable configuration assigned the `cluster-admin` ClusterRole to the `app-service` ServiceAccount through a ClusterRoleBinding.

The earlier assessment recorded permissions including:

- Reading pods.
- Reading secrets.
- Creating deployments.
- Deleting nodes.

The overprivileged binding was removed after testing.

### 5.3 Hardened Configuration

A namespace-scoped Role named `pod-reader` was created in `rbac-lab`.

It grants only:

- `get`
- `list`
- `watch`

These permissions apply to pods.

The `app-service` ServiceAccount was connected to this Role through a RoleBinding.

### 5.4 Latest Permission Verification

| Operation | Observed Result |
|---|---|
| List pods in rbac-lab | Allowed |
| Get secrets in rbac-lab | Denied |
| Create pods in rbac-lab | Denied |

Additional earlier tests recorded denial of access to pods in other namespaces and denial of cluster-wide administrative operations.

The overprivileged ClusterRoleBinding was confirmed absent.

### 5.5 Finding

**Finding: Excessive RBAC permissions increase the potential impact of compromised service accounts.**

The hardened configuration reduced permissions from cluster-wide administration to namespace-scoped pod-reading operations.

### 5.6 Assessment

The recorded authorization tests confirmed the intended least-privilege behavior.

These checks used Kubernetes authorization testing and do not independently validate token-based authentication.

Evidence:

`evidence/m6-security/psa-rbac-hardening.txt`

Additional reports:

- `reports/rbac-assessment-summary.md`
- `reports/rbac-overprivilege-comparison-20261110.md`

## 6. Pod Security Admission Assessment

### 6.1 Objective

Evaluate Kubernetes Pod Security Admission enforcement and verify hardened workload admission.

### 6.2 Baseline and Restricted Policy Testing

The assessment included Baseline and Restricted policy tests.

The recorded privileged workload test was rejected by Kubernetes.

The Restricted policy test also identified missing security requirements, including:

- `allowPrivilegeEscalation` not set to false.
- Linux capabilities not dropped.
- `runAsNonRoot` not enabled.
- Seccomp RuntimeDefault not configured.

### 6.3 Hardened Configuration

The hardened pod was named `psa-secure-pod` in namespace `psa-lab`.

Its security configuration included:

| Security Setting | Configuration |
|---|---|
| runAsNonRoot | true |
| runAsUser | 1000 |
| runAsGroup | 1000 |
| allowPrivilegeEscalation | false |
| readOnlyRootFilesystem | true |
| capabilities.drop | ALL |
| seccompProfile | RuntimeDefault |

### 6.4 Test Results

- The privileged test pod was rejected by the Restricted policy.
- The hardened pod was admitted.
- The hardened pod reached Running state.
- The observed container UID was 1000.
- The effective capabilities were recorded as zero.
- The filesystem write test was denied.

### 6.5 Finding

**Finding: Privileged containers and insecure security contexts can weaken Kubernetes workload isolation.**

Pod Security Admission enforced the Restricted policy and rejected the recorded non-compliant workload.

### 6.6 Assessment

The tests demonstrated that admission control can prevent selected insecure workload configurations before they are deployed.

Evidence:

`evidence/m6-security/psa-rbac-hardening.txt`

Additional reports:

- `reports/pod-security-assessment.md`
- `reports/psa-security-assessment-20261110.md`

## 7. Container Workload Hardening

### 7.1 Objective

Compare original application workloads with hardened equivalents.

### 7.2 Baseline Observations

The original application workloads in `k8s-net-test` were observed running as UID 0.

Their container security contexts were unspecified, and default service-account tokens were automatically mounted.

### 7.3 Hardened Implementation

A separate namespace named `k8s-net-hardened` was created.

The hardened workloads included:

- `net-client`
- `net-server`
- `unauthorized-client`

Security controls included:

- Non-root execution.
- Explicit UID and GID.
- Disabled privilege escalation.
- Dropped Linux capabilities.
- Read-only root filesystem.
- RuntimeDefault seccomp.
- Disabled automatic service-account token mounting.

The server used an `emptyDir` volume mounted at `/www` for web content.

### 7.4 Verification Results

| Security Check | Observed Result |
|---|---|
| Process identity | UID 1000, GID 1000 |
| Effective capabilities | Zero |
| Service-account token | Not mounted |
| Root filesystem write test | Denied |

All three hardened workloads passed the recorded runtime checks.

### 7.5 Finding

**Finding: Default container privileges and unnecessary service-account token mounting can increase workload exposure.**

The hardened configuration reduced privileges and restricted unnecessary access.

### 7.6 Assessment

The original baseline namespace was preserved separately from the hardened implementation.

Evidence and documentation:

`reports/network-workload-hardening-20261110.md`

## 8. Falco Runtime Threat Detection

### 8.1 Objective

Deploy Falco to monitor runtime activity and detect selected suspicious events inside containers.

### 8.2 Deployment

Falco was deployed in the Kubernetes environment.

The recorded deployment status showed:

- Falco pod: 2/2 Running.
- Falco DaemonSet: 1 desired, 1 ready, 1 available.
- Syscall event source enabled.
- Modern BPF probe selected.
- Health webserver listening on port 8765.

These are historical deployment observations from the captured status files.

### 8.3 Custom Detection Rules

Two custom rules were configured.

**Rule 1: Shell Spawned in Container**

Detects shell processes started inside containers.

Monitored processes include:

- bash
- sh
- ash
- dash
- zsh

**Rule 2: Package Manager Executed in Container**

Detects selected package-management processes inside containers.

Monitored processes include:

- apt
- apt-get
- apk
- yum
- dnf
- pip
- pip3
- dpkg
- rpm

### 8.4 Shell Detection Test

A controlled shell execution was performed inside the `net-client` container.

Falco generated a Warning alert:

- Rule: `Shell Spawned in Container`
- Process: `sh`
- Container: `client`
- Pod: `net-client`
- Namespace: `k8s-net-test`
- Source: `syscall`

A built-in Falco rule also detected a shell spawned with an attached terminal.

### 8.5 Package Manager Detection Test

A controlled process execution test was performed using:

`dpkg --version`

Falco generated a Warning alert:

- Rule: `Package Manager Executed in Container`
- Process: `dpkg`
- Command: `dpkg --version`
- Container: `client`
- Pod: `net-client`
- Namespace: `k8s-net-test`

**Testing limitation:** The BusyBox `dpkg` applet did not support the supplied `--version` option and the command returned an error. Falco nevertheless detected the process execution. This demonstrates process detection, not successful package installation or package management.

### 8.6 Sensitive File Access Detection

An earlier Falco runtime test generated a Warning alert for:

`Read sensitive file untrusted`

The observed event involved:

- File: `/etc/shadow`
- Process: `cat`
- Command: `cat /etc/shadow`
- Container: `client`
- Pod: `net-client`
- Namespace: `k8s-net-test`

This event demonstrates detection of sensitive-file access inside the test container. It does not establish host compromise or successful extraction of host credentials.

### 8.7 Finding

**Finding: Suspicious process execution and sensitive-file access inside containers require runtime monitoring.**

Falco detected the selected test events and generated corresponding alerts.

### 8.8 Evidence

- `evidence/m6-falco/controlled-shell-detection.log`
- `evidence/m6-falco/package-manager-detection.log`
- `reports/falco-runtime-detection.log`
- `reports/falco-custom-shell-detection.log`
- `reports/falco-pod-status.txt`
- `reports/falco-daemonset-status.txt`

## 9. Consolidated Security Test Results

| Security Area | Test | Result |
|---|---|---|
| Network Security | Authorized client communication | Passed |
| Network Security | Unauthorized client communication | Blocked |
| Network Security | Unauthorized port access | Blocked |
| RBAC | Namespace-scoped pod listing | Allowed |
| RBAC | Secret access | Denied |
| RBAC | Pod creation | Denied |
| PSA | Privileged workload admission | Rejected |
| PSA | Hardened workload admission | Passed |
| Workload Hardening | Non-root execution | Verified |
| Workload Hardening | Read-only root filesystem | Verified |
| Workload Hardening | Dropped capabilities | Verified |
| Falco | Shell execution detection | Detected |
| Falco | Package manager process detection | Detected |
| Falco | Sensitive-file access detection | Detected |

The results reflect the recorded controlled tests and evidence available in the repository.

## 10. Security Findings and Recommendations

### 10.1 Network Security

**Finding:** Unrestricted pod communication can expose internal services.

**Recommendation:** Apply default-deny NetworkPolicies and explicitly allow only required application communication.

### 10.2 RBAC

**Finding:** Cluster-wide administrative permissions can increase the impact of a compromised ServiceAccount.

**Recommendation:** Use namespace-scoped Roles and grant only the required API permissions.

### 10.3 Pod Security

**Finding:** Privileged containers and insecure security contexts increase workload risk.

**Recommendation:** Enforce Pod Security Admission Restricted policies where appropriate.

### 10.4 Container Hardening

**Finding:** Running containers as root and retaining unnecessary capabilities increases the potential impact of compromise.

**Recommendation:** Use non-root identities, drop unnecessary capabilities, disable privilege escalation, and use read-only root filesystems where compatible with the application.

### 10.5 Runtime Monitoring

**Finding:** Suspicious process execution and sensitive-file access may go unnoticed without runtime monitoring.

**Recommendation:** Maintain Falco monitoring, review alerts, tune custom rules, and integrate events with centralized logging and alerting.

## 11. Assessment Limitations

This assessment has the following limitations:

1. Testing was performed in a local Kind Kubernetes environment.
2. Only selected network, RBAC, admission-control, workload-hardening, and runtime detection scenarios were tested.
3. Falco detection results demonstrate the selected rules and events, not detection of every possible attack.
4. The package-manager test demonstrated process detection, not successful package management.
5. The sensitive-file event was observed inside a test container and does not prove host compromise.
6. Captured pod and DaemonSet status files represent the time at which the evidence was collected.
7. The assessment does not constitute a full production penetration test or independent security audit.

## 12. Overall Conclusion

The Kubernetes Security Vulnerability Assessment and Hardening project demonstrated practical security controls for protecting Kubernetes workloads.

Calico NetworkPolicies successfully restricted unauthorized communication. RBAC testing demonstrated the value of least-privilege access. Pod Security Admission rejected selected non-compliant workloads. Hardened configurations reduced container privileges, and Falco detected selected runtime security events.

The project successfully completed milestones M1 through M6, including the implementation and testing of its primary security controls.

The next stage is to complete final documentation, review the project evidence, and prepare the final academic demonstration and submission.

## 13. Evidence Directory

The principal M6 evidence files are:

```text
evidence/
├── m6-network/
│   └── network-policy-test-results.txt
├── m6-falco/
│   ├── controlled-shell-detection.log
│   └── package-manager-detection.log
└── m6-security/
    └── psa-rbac-hardening.txt
```

Additional assessment records are maintained in the `reports/` directory.

---

**End of Assessment Report**

**Author:** Madhur Kanoje
**Project:** Kubernetes Security: Vulnerability Assessment and Hardening
