# Endpoint Security Expansion Plan

Assessment date: 2026-03-19  
Target source to expand: `README.md` section `Security Baselines and Endpoint Protection`

## Objective

Expand the guide from high-level endpoint-security principles into an enterprise-ready operating model for 5k-50k device environments. The additions below are designed to fit the current README structure and terminology.

## Current State Summary

The guide currently states the right direction:

- start with Intune security baselines,
- refine with endpoint security policies and Settings Catalog,
- integrate Microsoft Defender for Endpoint where risk and response should align,
- avoid overlapping baselines and profiles.

That is correct but incomplete. The missing content is the implementation and operations layer: scope, rings, exceptions, recovery, monitoring, and incident-handling integration.

## Proposed Expansion Structure

1. Defender for Endpoint onboarding scope
2. Antivirus and Defender policy ownership
3. Attack Surface Reduction rollout strategy by rings
4. Firewall baseline strategy
5. BitLocker key escrow and recovery operations
6. Tamper protection caveats
7. Windows LAPS governance
8. Incident response and SIEM or Sentinel integration
9. KPIs and SLOs

## 1. Defender for Endpoint Onboarding Scope

### Content to add

- Define which device populations must onboard to Defender for Endpoint on day 0:
  - corporate Windows 11 endpoints,
  - privileged admin workstations,
  - internet-facing or high-value executive devices,
  - shared and frontline Windows devices where supported and operationally justified.
- Separate onboarding scope by platform:
  - required,
  - recommended,
  - exception-based,
  - unsupported or deferred.
- State whether onboarding is tenant-wide baseline policy or persona-specific.
- Document the dependency chain:
  - onboarding,
  - sensor health,
  - device risk evaluation,
  - compliance signal consumption,
  - Conditional Access enforcement.

### Decision guidance

- Default corporate Windows devices into onboarding unless there is a documented exception.
- Do not enable device-risk-based access until onboarding coverage and sensor-health reporting are stable.
- Use a named exception process for servers, labs, kiosk-style fixed-function devices, and unsupported legacy endpoints.

### Tables to add

| Table | Purpose |
| --- | --- |
| Platform onboarding matrix | Shows required, optional, and exception populations by OS and persona. |
| Risk signal dependency matrix | Shows which downstream controls depend on MDE onboarding and licensing. |

### Microsoft Learn references

- Intune security baselines: https://learn.microsoft.com/en-us/mem/intune/protect/security-baselines
- Require device compliance with Conditional Access: https://learn.microsoft.com/en-us/mem/intune/protect/create-conditional-access-intune
- Defender for Endpoint onboarding with Intune: validate URL
- Microsoft Defender for Endpoint onboarding overview: validate URL

## 2. Antivirus and Defender Policy Ownership

### Content to add

- Define one authoritative policy model for Microsoft Defender Antivirus settings.
- Separate baseline policy from exception policy.
- Avoid stacking security baseline, endpoint security, and Settings Catalog objects that manage the same Defender settings without a conflict review.
- Document ownership for:
  - real-time protection,
  - cloud-delivered protection,
  - sample submission,
  - scan schedule,
  - exclusions,
  - remediation defaults.

### Decision guidance

- Use endpoint security policy for day-to-day antivirus management where possible.
- Keep exclusions rare, time-bounded, documented, and approved jointly by app owner and security owner.
- Track exclusion debt as a measurable risk item, not a hidden local exception.

### Microsoft Learn references

- Intune security baselines: https://learn.microsoft.com/en-us/mem/intune/protect/security-baselines
- Microsoft Defender Antivirus policy in Intune: validate URL

## 3. ASR Rollout Strategy by Rings

### Content to add

- Add a dedicated ASR rollout model with four rings:
  - lab,
  - pilot,
  - broad,
  - enforced exception-holdback.
- Start most new ASR rules in audit mode.
- Promote to block mode only after telemetry review, app-owner validation, and documented rollback criteria.
- Keep high-risk admin populations earlier in the rollout, not later, unless a blocking dependency exists.
- Build a rule-level exception model instead of disabling the whole ASR profile for a device set.

### Ring model

| Ring | Goal | Typical duration | Exit criteria |
| --- | --- | --- | --- |
| Lab | Validate rule behavior on controlled devices | 1-2 weeks | No critical app or admin workflow breakage |
| Pilot | Test with IT, security, and engineering cohorts | 2-4 weeks | Acceptable audit findings and documented mitigations |
| Broad | Enforce across standard corporate population | 4-8 weeks by wave | Help-desk volume and block telemetry remain within threshold |
| Holdback | Temporary exception population only | Time-bounded | Exception retired or alternate control documented |

### Operational requirements

- Review ASR telemetry weekly during rollout waves.
- Maintain a rule register with:
  - rule name,
  - mode,
  - owner,
  - exception groups,
  - review date.
- Publish rollback steps before block-mode promotion.

### Microsoft Learn references

- Intune security baselines: https://learn.microsoft.com/en-us/mem/intune/protect/security-baselines
- Attack surface reduction rules in Intune: validate URL
- Defender for Endpoint advanced hunting or reporting for ASR outcomes: validate URL

## 4. Firewall Baseline Strategy

### Content to add

- Define Windows Defender Firewall as enabled by default for domain, private, and public profiles unless an exception is approved.
- Use a baseline-deny posture for inbound traffic with approved, documented rules only.
- Separate firewall policy into:
  - baseline common rules,
  - line-of-business app exceptions,
  - temporary troubleshooting exceptions.
- Tie every firewall exception to an application owner, business justification, source or destination requirement, and expiry date.

### Governance model

- Do not allow teams to request blanket firewall disablement.
- Review local firewall rule merge behavior intentionally.
- Keep server-style exception thinking out of the standard client policy unless a client workload genuinely requires it.

### Tables to add

| Table | Purpose |
| --- | --- |
| Firewall exception register | Tracks approved inbound rules and retirement dates. |
| Persona firewall profile matrix | Shows where standard, shared, kiosk, or lab devices deviate. |

### Microsoft Learn references

- Intune security baselines: https://learn.microsoft.com/en-us/mem/intune/protect/security-baselines
- Intune endpoint security firewall policy: validate URL

## 5. BitLocker Key Escrow and Recovery Operations

### Content to add

- State the required escrow target for recovery keys before enforcement.
- Define encryption scope by persona:
  - standard corporate devices,
  - shared devices,
  - kiosk or fixed-function devices,
  - removable media if in scope.
- Document recovery operations:
  - who can retrieve keys,
  - audit expectations,
  - approval path,
  - emergency process,
  - post-recovery follow-up.
- Include rotation or regeneration expectations after key use where the platform supports it.

### Operational safeguards

- Do not enforce encryption at scale until escrow reporting proves healthy.
- Restrict recovery-key access through RBAC, scope tags, and privileged access procedures.
- Log key retrieval events and review them regularly.
- Define when wipe, retire, or reprovision is preferred over repeated recovery events.

### Tables to add

| Table | Purpose |
| --- | --- |
| BitLocker readiness checklist | Confirms TPM, escrow, user comms, and support readiness before rollout. |
| Recovery runbook summary | Gives help desk and security teams a shared operating path. |

### Microsoft Learn references

- Intune security baselines: https://learn.microsoft.com/en-us/mem/intune/protect/security-baselines
- BitLocker policy with Intune: validate URL
- Recovery key retrieval and RBAC guidance: validate URL

## 6. Tamper Protection Caveats

### Content to add

- Explain that tamper protection is designed to resist local weakening of security controls and should be enabled for mainstream managed Windows endpoints unless a documented support case requires otherwise.
- Call out that some troubleshooting and change workflows can fail silently or appear inconsistent when tamper protection blocks local changes.
- Require a documented support process for approved temporary disablement scenarios, including security approval and time-bounded rollback.

### Caveats to document

- Local admin access is not a valid reason to bypass tamper protection.
- Legacy tooling that expects to modify Defender settings directly must be identified before rollout.
- Security teams and EUC teams need a shared troubleshooting script so incidents are not misdiagnosed as policy drift.

### Microsoft Learn references

- Microsoft Defender tamper protection overview: validate URL
- Intune security baselines: https://learn.microsoft.com/en-us/mem/intune/protect/security-baselines

## 7. Windows LAPS Governance

### Content to add

- Define whether Windows LAPS is mandatory for all Microsoft Entra joined and hybrid-joined corporate Windows devices.
- State the approved local administrator account model:
  - built-in administrator renamed or managed,
  - dedicated support admin account,
  - no standing local admin for standard users.
- Set rotation cadence, password complexity policy, authorized retrieval roles, and retrieval logging expectations.
- Document break-glass use, approval path, and post-use review.

### Governance requirements

- Local admin rights must be exception-based and time-bounded.
- LAPS password retrieval must be auditable and restricted to approved roles.
- Shared devices and kiosks need explicit handling because break-glass patterns differ from named-user devices.

### Tables to add

| Table | Purpose |
| --- | --- |
| LAPS governance matrix | Maps device persona to local admin model, retrieval rights, and rotation expectations. |
| Break-glass workflow | Shows who approves retrieval and what evidence is retained. |

### Microsoft Learn references

- Windows LAPS with Intune: validate URL
- Account protection policy in Intune: validate URL

## 8. Incident Response and SIEM or Sentinel Integration

### Content to add

- Define minimum incident workflow integration points:
  - Intune device record,
  - Defender incident or alert context,
  - identity context,
  - ticketing system,
  - SIEM or Sentinel case management.
- Explain how endpoint actions map to incident stages:
  - isolate,
  - investigate,
  - collect evidence,
  - revoke access,
  - wipe or reprovision only after containment and evidence review.
- Require a join-up model between endpoint engineering, SOC, and identity teams.

### Operating model

- Use Sentinel or another SIEM to centralize cross-domain investigation where available.
- Preserve device evidence before lifecycle actions that destroy forensic value.
- Define when Intune actions are operator-led versus SOC-led.
- Document the escalation path for compromised privileged workstations separately from standard endpoints.

### Tables to add

| Table | Purpose |
| --- | --- |
| Incident action matrix | Maps security scenario to Intune, Defender, identity, and SIEM actions. |
| RACI for endpoint incidents | Clarifies SOC, EUC, identity, and help-desk responsibilities. |

### Microsoft Learn references

- Endpoint analytics: https://learn.microsoft.com/en-us/intune/analytics/overview
- Microsoft Sentinel integration guidance for Defender and Intune data: validate URL
- Microsoft Defender XDR incident response guidance: validate URL

## 9. KPIs and SLOs

### Content to add

Add a security-specific KPI and SLO subsection after `Reporting and Monitoring`.

### Recommended KPIs

| KPI | Definition | Suggested target direction |
| --- | --- | --- |
| MDE onboarding coverage | Percent of in-scope devices successfully onboarded | Approach 100% for required populations |
| Sensor health coverage | Percent of onboarded devices reporting healthy telemetry | Maintain high and stable trend |
| ASR enforcement coverage | Percent of in-scope devices on approved ASR rule set in intended mode | Increase by ring until broad enforcement is reached |
| BitLocker escrow coverage | Percent of encrypted in-scope devices with recoverable keys in the approved directory | Near-total coverage before broad enforcement |
| Firewall compliance | Percent of devices with expected firewall profiles enabled | Maintain consistently high posture |
| LAPS rotation success | Percent of in-scope devices rotating and storing passwords successfully | Maintain consistently high posture |
| Mean time to triage | Time from alert creation to ownership assignment | Reduce over time |
| Mean time to containment | Time from confirmed malicious activity to containment action | Reduce over time |
| Exception debt | Count of active security exceptions past review date | Drive toward zero |

### Example SLOs

- Critical endpoint onboarding failures are triaged within 1 business day.
- High-severity endpoint incidents are owned within 30 minutes and containment starts within 2 hours.
- Recovery-key access events are reviewed within 1 business day.
- ASR exceptions are reviewed at least monthly and expire unless renewed.
- Firewall or BitLocker policy drift above agreed threshold triggers service review.

## Recommended README Insert Plan

| README location | Insert or expansion | Priority |
| --- | --- | --- |
| `Security Baselines and Endpoint Protection` | Replace short summary with a multi-subsection endpoint-security chapter | P0 |
| After `Compliance and Conditional Access Alignment` | Add `Device Risk and Access Enforcement` subsection | P1 |
| After `Reporting and Monitoring` | Add `Security KPIs and SLOs` subsection | P1 |
| After `Device Lifecycle and Cleanup` | Add `Incident Preservation and Recovery Operations` subsection | P1 |

## Draft Recommendation Summary

The current guide is credible at the architecture level but not yet sufficient for endpoint-security operations in a large enterprise. The minimum viable improvement is to add a dedicated endpoint-security expansion that operationalizes Defender onboarding, ASR rollout, firewall and BitLocker management, LAPS governance, tamper-protection handling, incident integration, and measurable service outcomes.
