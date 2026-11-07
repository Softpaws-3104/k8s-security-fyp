# Kubernetes Pod Security Assessment

## Vulnerability
Insecure Container Security Configuration

## Vulnerable Configuration
- Container executed as root (UID 0).
- Privilege escalation enabled.
- Root filesystem writable.
- Default Linux capabilities retained.

## Vulnerable Test Results
- Container identity: UID 0.
- Filesystem write: Successful.
- Privilege escalation: Enabled.

## Hardened Configuration
- runAsNonRoot: true
- runAsUser: 1000
- allowPrivilegeEscalation: false
- readOnlyRootFilesystem: true
- capabilities.drop: ALL
- seccompProfile: RuntimeDefault

## Hardened Test Results
- Container identity: UID 1000.
- Filesystem write: Denied.
- Security configuration: Verified.

## Conclusion
The hardened configuration reduces container privileges,
prevents privilege escalation, restricts filesystem writes,
and removes unnecessary Linux capabilities.
