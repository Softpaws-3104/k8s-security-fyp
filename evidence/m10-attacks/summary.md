# M10 - Controlled Kubernetes Attack Simulation

Date: 27 November 2026
Environment: Kali Linux, Kind, Kubernetes v1.34.0, Calico

## Objective

Evaluate Kubernetes security controls through controlled reconnaissance,
RBAC authorization checks, admission-policy testing, network-isolation
testing, and runtime threat detection.

## Test Results

| ID | Test | Result |
|---|---|---|
| A1 | Kubernetes API reconnaissance | API reachable; namespace listing denied (403) |
| A2 | ServiceAccount RBAC authorization | Pod listing allowed; Secret access denied |
| A3 | Privileged pod admission | Rejected by Pod Security Admission |
| A4 | Network isolation | Authorized traffic allowed; unauthorized and cross-namespace tests timed out |
| A5 | Secret access authorization | Secret access denied |
| A6 | Sensitive file access detection | Falco generated two Warning alerts |

## Security Controls Evaluated

- Kubernetes API authentication and authorization
- Least-privilege Role-Based Access Control
- Pod Security Admission restricted enforcement
- Calico-enforced Kubernetes NetworkPolicies
- Falco runtime threat detection

## Evidence

- A1.txt - API reconnaissance
- A2.txt - ServiceAccount RBAC authorization
- A3.txt - Privileged pod admission
- A4.txt - Network isolation
- A5.txt - Secret access authorization
- A6.txt - Sensitive file detection

## Scope and Limitations

1. A1 tested the local API endpoint. The version endpoint returned HTTP 200,
   while anonymous namespace listing was denied with HTTP 403.
2. A2 and A5 used kubectl authorization impersonation checks.
   They did not extract or use a ServiceAccount token.
3. A3 used server-side dry-run admission testing. No privileged pod was created.
4. A4 used connectivity tests. Timeouts indicate failed connections but do
   not independently prove the precise cause of every failure.
5. A6 reuses the Falco detection evidence originally collected during M5
   on 22 November 2026. It was not a new M10 execution.

## Conclusion

The controlled tests demonstrated that RBAC restricted unauthorized
resource access, Pod Security Admission rejected a privileged workload,
network policies restricted tested traffic, and Falco detected sensitive
file access inside a container.

These results demonstrate the operation of selected Kubernetes security
controls in the test environment. They do not establish that every possible
attack path is prevented.
