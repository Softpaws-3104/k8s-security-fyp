# M9 – Vulnerable Baseline Security Scan

**Date:** 27 November 2026  
**Project:** Kubernetes Security FYP  
**Environment:** Kind cluster, Kubernetes v1.34.0, single control-plane node

## 1. Scan Scope and Tools

Three security tools were used to establish the vulnerable baseline.

| Tool | Version | Scan scope |
|---|---|---|
| Trivy | 0.76.2 | Local manifests in `vulnerable/` |
| Kubescape | 4.0.21 | Local manifests in `vulnerable/`, NSA and MITRE frameworks |
| kube-bench | 0.16.0 | Running Kind Kubernetes node configuration |

## 2. Trivy Results

Trivy scanned four vulnerable Kubernetes manifests.

| Manifest | Finding occurrences |
|---|---:|
| insecure-pod.yaml | 16 |
| overprivileged-rbac.yaml | 1 |
| psa-privileged-pod.yaml | 16 |
| psa-restricted-test.yaml | 16 |
| **Total** | **49** |

These are finding occurrences across the scanned manifests, not necessarily 49 unique weaknesses.

Evidence:
- `trivy-vulnerable.txt`
- `trivy-vulnerable.json`

## 3. Kubescape Results

Frameworks scanned: NSA and MITRE.

| Metric | Result |
|---|---:|
| Overall compliance score | 70 |
| Controls assessed | 30 |
| Passed controls | 20 |
| Failed controls | 10 |
| Action required | 0 |
| Critical resource findings | 0 |
| High resource findings | 7 |
| Medium resource findings | 16 |
| Low resource findings | 3 |
| NSA compliance score | 54.58 |
| MITRE compliance score | 92.16 |

Failed control areas included privileged containers, CPU and memory limits, non-root execution, privilege escalation, network restrictions, service account mapping, Linux hardening and immutable container filesystems.

Evidence:
- `kubescape-vulnerable.json`

## 4. kube-bench Results

kube-bench assessed the running Kind node configuration.

| Result | Checks |
|---|---:|
| PASS | 63 |
| FAIL | 12 |
| WARN | 56 |
| **Total** | **131** |

The report includes failed checks relating to API server audit logging, profiling configuration and kubelet configuration file permissions.

Warnings require manual review and should not automatically be interpreted as confirmed vulnerabilities.

Evidence:
- `kube-bench-vulnerable.txt`
- `kube-bench-job.yaml`

## 5. Scope and Limitations

- Trivy and Kubescape assessed local YAML manifests in the `vulnerable/` directory. These results do not represent a live-cluster scan.
- kube-bench assessed the running Kind node, not the local vulnerable manifests.
- Kind uses a containerized Kubernetes node based on Debian 12. Some kube-bench checks may need interpretation in this environment.
- Trivy finding occurrences, Kubescape failed controls and kube-bench check results measure different things and must not be added together or compared as equivalent metrics.
- WARN results are not the same as FAIL results.

## 6. M9 Conclusion

The vulnerable baseline has been captured using three complementary security scanners. The raw outputs are retained under `evidence/m9-baseline/` for comparison with the hardened-state scans planned in M12.

**M9 baseline scanning evidence collection completed.**
