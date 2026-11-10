# Kubernetes Network Workload Hardening Assessment

**Date:** 10 November 2026  
**Project:** Kubernetes Security FYP  
**Environment:** Kali Linux, Kind, Kubernetes v1.34.0, Calico CNI

## 1. Objective

Compare the original application workloads in `k8s-net-test` with hardened equivalents in `k8s-net-hardened`, while preserving the original baseline.

## 2. Baseline observations

The original application pods in `k8s-net-test` were observed running as UID 0.

Their container security contexts were unspecified, and their default service-account tokens were automatically mounted.

## 3. Hardened implementation

A separate namespace, `k8s-net-hardened`, was created with Pod Security Admission Restricted enforcement, audit, and warning labels.

Three application pods were deployed:
- net-client
- net-server
- unauthorized-client

The hardened workloads use:
- runAsNonRoot: true
- runAsUser: 1000
- runAsGroup: 1000
- seccompProfile: RuntimeDefault
- allowPrivilegeEscalation: false
- readOnlyRootFilesystem: true
- capabilities.drop: ALL
- automountServiceAccountToken: false

The server uses an emptyDir volume mounted at `/www` for web content, allowing the HTTP service to run with a read-only root filesystem.

## 4. Network security configuration

A ClusterIP Service exposes the server on TCP port 8080.

Two NetworkPolicies were configured:
- deny-server-ingress: isolates the server from unsolicited ingress.
- allow-client-to-server: permits TCP port 8080 from pods labelled app=net-client.

## 5. Network connectivity tests

| Test | Expected result | Observed result |
|---|---|---|
| net-client to net-server | HTTP success | Successful |
| unauthorized-client to net-server | Blocked | Request timed out |

The authorized client received:
`Hardened Kubernetes networking test successful`

The unauthorized client received:
`wget: download timed out`

## 6. Runtime security verification

All three hardened pods were checked.

| Security check | Observed result |
|---|---|
| Process identity | UID 1000, GID 1000 |
| Effective capabilities | 0000000000000000 |
| Service-account token | Not mounted |
| Root filesystem write test | Read-only; write denied |

All three workloads passed the runtime checks.

## 7. Conclusion

The hardened workloads successfully demonstrate non-root execution, least privilege at the container level, read-only root filesystems, disabled automatic service-account token mounting, and restricted network communication.

The original `k8s-net-test` namespace was preserved as the baseline. The hardened implementation was deployed separately in `k8s-net-hardened`.

This assessment demonstrates the effectiveness of combining Kubernetes Pod Security Admission, container security contexts, and Calico NetworkPolicies.
