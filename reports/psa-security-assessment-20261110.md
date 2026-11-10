# Kubernetes Pod Security Admission Assessment

Date: 10 November 2026
Cluster: Kind
Kubernetes Server: v1.34.0

## 1. Objective
Evaluate Kubernetes Pod Security Admission using Baseline
and Restricted enforcement policies.

## 2. Baseline Policy Test

Namespace: psa-lab
Enforcement: baseline

Test:
A pod configured with privileged: true was submitted.

Result:
Rejected by Kubernetes.

Violation:
Privileged containers are prohibited by Baseline.

## 3. Restricted Policy Test

Namespace: psa-lab
Enforcement: restricted

Test:
A pod without the required Restricted security settings
was submitted.

Result:
Rejected by Kubernetes.

Detected violations:
- allowPrivilegeEscalation was not false.
- Linux capabilities were not dropped.
- runAsNonRoot was not true.
- Seccomp RuntimeDefault was not configured.

## 4. Hardened Pod Test

Pod: psa-secure-pod
Namespace: psa-lab

Security configuration:
- runAsNonRoot: true
- runAsUser: 1000
- runAsGroup: 1000
- allowPrivilegeEscalation: false
- readOnlyRootFilesystem: true
- capabilities.drop: ALL
- seccompProfile: RuntimeDefault

Result:
Pod successfully admitted and reached Running state.

## 5. Runtime Verification

Container identity:
UID 1000, GID 1000.

Filesystem test:
Writing to /security-test was denied.

Capabilities:
CapEff: 0000000000000000

Namespace enforcement:
Restricted.

## 6. Conclusion

Pod Security Admission successfully rejected workloads
that violated Baseline and Restricted security requirements.

The hardened workload satisfied Restricted admission
requirements and passed the runtime checks performed.

This demonstrates the value of Kubernetes admission
control and container security contexts.
