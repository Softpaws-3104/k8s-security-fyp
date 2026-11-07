# Kubernetes RBAC Security Assessment

## 1. Objective

Assess excessive Kubernetes Role-Based Access Control (RBAC)
permissions and implement least-privilege access.

## 2. Vulnerable Configuration

The application ServiceAccount was assigned cluster-admin
permissions through a ClusterRoleBinding.

### Security tests

The following operations were permitted:

- List pods across all namespaces.
- Read secrets across all namespaces.
- Create ClusterRoleBindings.

This demonstrated excessive privileges and unnecessary
cluster-wide access.

## 3. Hardened Configuration

The cluster-admin binding was removed.

A namespace-scoped Role named `pod-reader` was created
in the `rbac-lab` namespace.

The Role grants only:

- get
- list
- watch

These permissions apply to pods only.

A RoleBinding connects the `app-service` ServiceAccount
to this Role.

## 4. Hardened Validation Results

| Test | Expected result | Observed result |
|---|---|---|
| List pods in rbac-lab | Allowed | Allowed |
| List pods in default | Denied | Denied |
| Read secrets in rbac-lab | Denied | Denied |
| Create ClusterRoleBindings | Denied | Denied |

## 5. Conclusion

The assessment demonstrated how excessive cluster-admin
permissions can increase Kubernetes security risk.

Replacing cluster-wide administrative access with
namespace-scoped, least-privilege permissions reduced
the ServiceAccount's access.

The hardened configuration passed the recorded
authorization tests.
