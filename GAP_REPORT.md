# Gap Report for `README.md`

Assessment date: 2026-03-19  
Source documents reviewed: `AGENTS.md`, `README.md`

## Coverage Matrix by Section

| AGENTS.md required section | Current README coverage | Status | Notes |
| --- | --- | --- | --- |
| Tenant hygiene | `Tenant Hygiene and Governance` | Covered | Governance, naming, RBAC, scope tags, dynamic groups, and ownership are present. |
| Enrollment strategy | `Enrollment Strategy` | Covered | Persona-led enrollment guidance is present and reasonably complete. |
| App lifecycle and Win32 packaging | `App Lifecycle and Win32 Packaging` | Covered | Good operational guidance, including IME behavior, app limit, and Autopilot caution. |
| Update rings and feature updates | `Update Rings and Feature Updates` | Covered | Ring model is clear; rollback and deferral interaction are called out. |
| Security baselines | `Security Baselines and Endpoint Protection` | Partial | High-level baseline guidance exists, but endpoint-security depth is materially thin. |
| Compliance policies | `Compliance and Conditional Access Alignment` | Covered | Good design linkage to Conditional Access and compliance defaults. |
| Conditional Access alignment | `Identity and Access Foundation`, `Compliance and Conditional Access Alignment` | Covered | Strong conceptual alignment, but advanced access scenarios are not expanded. |
| Device lifecycle and cleanup | `Device Lifecycle and Cleanup` | Covered | Cleanup behavior and lifecycle actions are present. |
| Reporting and monitoring | `Reporting and Monitoring` | Covered | Basic KPI set exists; security operations metrics are underdeveloped. |
| Co-management vs Intune-only | `Co-Management vs Intune-Only Decision Guidance` | Covered | Strong decision framing and exit-plan guidance. |
| Licensing (E3/E5/Intune Suite) | `Licensing and Intune Suite Considerations` | Partial | Intune Suite is covered, but E3/E5 security dependency mapping is not explicit. |

## Missing Subtopics

### Cross-guide gaps

| Area | Missing or underdeveloped subtopics | Impact |
| --- | --- | --- |
| Security baselines and endpoint protection | Defender for Endpoint onboarding scope, AV policy ownership, ASR ring design, firewall strategy, BitLocker escrow and recovery operations, tamper protection caveats, Windows LAPS governance, account protection layering | The guide names endpoint security, but it does not yet tell an enterprise how to implement or operate it. |
| Compliance and Conditional Access | Device risk integration decision points, report-only sequencing examples, exclusion governance, privileged admin access branch, non-persistent/shared-device exceptions | Access policy guidance is conceptually sound but not operationally complete. |
| Licensing | E3 vs E5 dependency mapping for Defender, Conditional Access, risk-based access, and Intune Suite add-ons | Readers may design controls that their license set cannot support. |
| Reporting and monitoring | Security KPIs, SLOs, drift thresholds, MDE alert flow, Sentinel integration, incident ownership model | The current KPI list is useful, but too general for endpoint-security operations. |
| Tenant hygiene | Policy lifecycle review cadence, change control expectations, documentation standards, exception register schema | Governance is directionally correct but lacks an operating model. |
| Enrollment strategy | Security prerequisites by persona, enrollment restrictions, local admin posture for Autopilot devices, secure provisioning checkpoints | Enrollment is covered, but hardening dependencies are not tied in. |
| Update rings | Security-control compatibility checks with feature updates, emergency servicing path, driver governance with security tools | Ring design does not yet connect servicing to security tooling outcomes. |
| Shared/frontline/education | Security control exceptions for shared and frontline modes, local admin avoidance, credential handling, logging constraints | These branches exist but need risk-specific handling. |
| Device lifecycle | Recovery key retention, evidence preservation during incident handling, wipe vs retain decision matrix | Lifecycle guidance is operational, but not security-incident aware. |

### Highest-priority missing security chapters

1. Endpoint security architecture and ownership model.
2. Defender for Endpoint integration and risk signal usage.
3. Attack Surface Reduction rollout by ring.
4. Firewall and BitLocker operational guidance.
5. Windows LAPS governance and privileged local account controls.
6. Incident response workflow with Sentinel or alternate SIEM.
7. Security KPIs and service-level objectives.

## Stale-Guidance Risk Checks

| Topic | Current README state | Risk level | Why it can stale quickly | Check cadence |
| --- | --- | --- | --- | --- |
| Intune licensing and add-ons | Present, validated 2026-03-19 | High | SKU entitlements, Intune Suite packaging, and sovereign cloud support can change. | Quarterly |
| Windows servicing position | Present, includes Windows 10 end-of-support context | High | Release cadence, safeguard holds, and support dates shift. | Monthly |
| Autopilot and app-install caveats | Present | Medium | Enrollment Status Page behavior and Autopilot device preparation guidance may change. | Quarterly |
| Compliance and Conditional Access design | Present | Medium | New CA conditions, templates, and enforcement patterns evolve regularly. | Quarterly |
| Security baselines | Present at high level only | High | Baseline versions, recommended settings, and policy conflicts change as Windows and Defender evolve. | Quarterly |
| Defender integration references | Mentioned, not expanded | High | Onboarding methods, portal flows, risk signal mappings, and licensing guidance change often. | Quarterly |
| Shared/frontline/education guidance | Present | Medium | Platform capabilities and enrollment methods can change by OS release. | Semiannual |
| Reporting and analytics | Present | Medium | Intune reports, endpoint analytics fields, and Sentinel connectors can change. | Quarterly |

## Prioritized Recommendations

### Priority 0: expand endpoint-security coverage before treating the guide as enterprise-ready

Add a dedicated endpoint-security chapter or a major expansion of `Security Baselines and Endpoint Protection` covering:

- Defender for Endpoint onboarding scope and licensing dependencies.
- ASR policy rollout by pilot, broad, and holdback rings.
- Firewall baseline design and exception governance.
- BitLocker escrow, recovery, rotation, and help-desk runbooks.
- Tamper protection limitations and support caveats.
- Windows LAPS governance and break-glass controls.
- Security monitoring, incident response, and Sentinel integration.
- Security KPIs and SLOs.

### Priority 1: make licensing and access dependencies explicit

Add a matrix that maps E3, E5, Defender for Endpoint, Entra P1/P2, and Intune Suite capabilities to:

- compliance-driven access,
- device risk integration,
- advanced endpoint controls,
- privileged admin controls,
- remote support and remediation workflows.

### Priority 2: strengthen operational governance

Add explicit operating-model content for:

- policy review cadence,
- exception ownership and expiry,
- baseline version control,
- rollout approval gates,
- rollback triggers,
- documentation minimums for new security controls.

### Priority 3: mature reporting from generic health to security operations

Extend the KPI section to include:

- MDE onboarding coverage,
- ASR block-to-audit conversion rate,
- BitLocker escrow coverage,
- firewall profile compliance,
- LAPS rotation success rate,
- time to triage and time to containment,
- incident-to-remediation closure time.

## Recommended Section Inserts

| Insert location in README | Recommended addition | Reason |
| --- | --- | --- |
| After `Security Baselines and Endpoint Protection` | New subsection: `Endpoint Security Operating Model` | Converts a generic section into actionable enterprise guidance. |
| After `Compliance and Conditional Access Alignment` | New subsection: `Device Risk and Access Decision Patterns` | Connects Defender, compliance, and Conditional Access more explicitly. |
| After `Reporting and Monitoring` | New subsection: `Security KPIs and SLOs` | Makes security operations measurable. |
| After `Device Lifecycle and Cleanup` | New subsection: `Recovery, Forensics, and Incident Preservation` | Prevents destructive lifecycle actions during active incidents. |
| Inside `Licensing and Intune Suite Considerations` | New table: `License Dependency Matrix` | Reduces late-stage design rework. |

## Overall Assessment

`README.md` is strong as a broad Intune architecture guide and already covers most AGENTS.md top-level sections. Its main gap is not structural coverage, but depth: endpoint security is only summarized, not operationalized. The guide is closest to complete for enrollment, apps, servicing, compliance, and co-management; it is least complete for enterprise endpoint protection operations and the licensing dependencies that support those controls.
