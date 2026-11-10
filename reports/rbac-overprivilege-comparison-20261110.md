# Kubernetes RBAC Security Assessment

Date: 10 November 2026

## Objective
Compare an overprivileged Kubernetes ServiceAccount
against a least-privilege RBAC configuration.

## Test Identity
ServiceAccount: app-service
Namespace: rbac-lab

## 1. Baseline
Before assigning additional permissions:
- Get pods: DENIED
- Get secrets: DENIED
- Create deployments: DENIED

## 2. Vulnerable Configuration
A ClusterRoleBinding assigned the cluster-admin ClusterRole
to app-service.

Observed results:
- Get pods in rbac-lab: ALLOWED
- Get secrets in rbac-lab: ALLOWED
- Create deployments in default: ALLOWED
- Delete nodes: ALLOWED

The binding was removed immediately after testing.

## 3. Hardened Configuration
A namespace-scoped Role named pod-reader was created.

Allowed permissions:
- get pods
- list pods
- watch pods

The Role was assigned to app-service using a RoleBinding.

Observed results:
- Get pods in rbac-lab: ALLOWED
- List pods in rbac-lab: ALLOWED
- Watch pods in rbac-lab: ALLOWED
- Get secrets: DENIED
- Create deployments: DENIED
- Get pods in default: DENIED
- Delete nodes: DENIED

## 4. Security Conclusion
The cluster-admin ClusterRoleBinding provided excessive
cluster-wide permissions to the application ServiceAccount.

The namespace-scoped Role and RoleBinding significantly
reduced its privileges and restricted access to pod-reading
operations within rbac-lab.

Authorization was verified using kubectl auth can-i
with ServiceAccount impersonation.

These checks evaluate authorization and do not independently
verify token-based authentication.
