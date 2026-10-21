# Security assessment

## Checkov 3.3.30 results

Scope: application YAML in manifests/.
Calico vendor configuration and live control-plane settings were not assessed.

| Scan | Passed | Failed | Skipped |
|---|---:|---:|---:|
| Baseline | 275 | 8 | 0 |
| After remediation | 279 | 4 | 0 |

These counts are scanner check results, not a count of independently
verified cluster controls or a production compliance certification.

## Remediation

- Changed three workloads from UID/GID 101 to 10001.
- Verified the web deployment runs as UID 10001 with zero effective
  capabilities, NoNewPrivs enabled and seccomp filtering active.
- Recreated the test pods and verified secret-file access and connectivity.
- Added an ingress-deny policy selecting the network client.
- Confirmed the client's permitted outgoing web connection still succeeds.
  Unsolicited incoming traffic to that client was not separately tested.

## Four remaining findings

CKV_K8S_8 and CKV_K8S_9 apply to both test pods:

- lab6-client: a disposable command-execution pod that sleeps during
  network testing; it does not serve application traffic.
- lab6-secret-check: a one-shot file-readability check; successful
  process completion is its test result.

Readiness and liveness probes are intentionally omitted for these test
utilities. These findings remain visible and are not suppressed.
The web application has HTTP readiness and liveness probes.

## Evidence boundaries

- RBAC tests used administrator impersonation, not a service-account login.
- Pod deletion permission was checked with auth can-i; no deletion was attempted as the auditor.
- The unsafe pod was rejected through a server-side dry run.
- Network tests verified baseline access, denial, approved-client access
  and denial after removing the approved label.
- Network policies restrict ingress; outbound traffic remains unrestricted.
- Pod labels are not cryptographic identities; permissions to change
  workloads and labels must be controlled.
- The demonstration secret was generated at runtime and mounted read-only.
  Its value was not printed or saved in project files.
- Kubernetes Secret storage encryption at rest was not configured or verified.
- No container vulnerability scan has been completed at this stage.
