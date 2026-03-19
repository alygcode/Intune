# Microsoft Intune Best-Practice Guide

Enterprise guidance for a 5k-50k device environment with hybrid AD and Microsoft Entra ID history, moving to cloud-only management where practical. This guide is written to stand alone as a customer-ready README or to be split into chapters by section.

Last validated against Microsoft Learn guidance: 2026-03-19.

## Executive Summary

For enterprises at this scale, Microsoft Intune should be the long-term endpoint control plane. Use Intune to own device enrollment, configuration, compliance, app delivery, update orchestration, and endpoint security integration. Keep identity, access, compliance, and servicing decisions aligned so the environment behaves predictably.

For hybrid estates, co-management is usually a transition pattern, not the target architecture. New Windows endpoints should default to Microsoft Entra join, Windows Autopilot, Intune policy, and cloud-based access controls. Keep Configuration Manager only where a defined technical dependency still exists, and retire that dependency in planned waves.

The design bias in this document is:

- Simplify to one primary control plane.
- Standardize around reusable policy objects, not per-team exceptions.
- Use virtual groups, filters, scope tags, RBAC, profiles, baselines, and rings to scale operations.
- Validate licensing before design decisions are locked.
- Treat Windows 11 as the mainstream endpoint platform, with separate servicing tracks for LTSC and fixed-function devices.

## Quick-Start Checklist

- Confirm licensing, cloud availability, and whether any target workloads depend on Intune Suite features.
- Define device personas: corporate, BYOD, shared, frontline, EDU, kiosk, lab, VDI, and fixed-function.
- Set an explicit target state for new Windows devices: Microsoft Entra join, Intune enrollment, standardized apps, and cloud access controls.
- Set the MDM user scope intentionally. Do not leave automatic enrollment behavior to defaults or overlap it carelessly with WIP scope.
- Establish naming standards, RBAC boundaries, scope tags, dynamic group strategy, and assignment filter rules before broad rollout.
- Build pilot rings for enrollment, apps, security baselines, compliance, and Windows servicing before production targeting.
- Define a Windows configuration profile strategy (Settings Catalog first, templates only when needed) before broad assignments.
- Standardize Win32 packaging, detection rules, uninstall paths, dependencies, supersedence, and restart behavior.
- Define the co-management exit plan before moving workloads.
- Configure compliance first, test Conditional Access in report-only mode, and enforce only after pilot validation.
- Define lifecycle actions for retire, wipe, delete, reprovision, and stale-record cleanup.

## Audience, Scope, and Assumptions

This guide targets endpoint engineering, identity, security, architecture, and operations teams running large enterprise tenants. It assumes:

- A Windows-heavy estate with some macOS and mobile platforms.
- Historical dependence on Active Directory, Group Policy, Configuration Manager, PKI, or hybrid join.
- A strategic move toward cloud-only management where business and technical dependencies allow.
- Multiple device personas that require different enrollment, access, app, and compliance patterns.

One policy model will not fit all personas. Shared pools, frontline workflows, education environments, labs, kiosks, and fixed-function systems should be designed as explicit branches, not forced into the standard corporate endpoint model.

## Operating Principles

1. Intune is the primary management plane unless a documented exception exists.
2. One control should have one clear owner. Avoid split authority across Intune, Group Policy, and Configuration Manager.
3. Build for repeatability. Reuse profiles, baselines, rings, filters, and dynamic groups instead of cloning objects.
4. Pilot before enforcement. Use limited groups, report-only validation, and rollback criteria.
5. Reduce exception debt. Every exception should have an owner, expiry, and review date.
6. Optimize for operations, not just deployment. A design that cannot be monitored and supported at scale is not complete.

## Licensing and Intune Suite Considerations

Validate licensing before solution design. The most common design errors at enterprise scale come from assuming features are available everywhere, then redesigning late.

| Need | Common Licensing Path | Design Guidance |
| --- | --- | --- |
| Core MDM and MAM | Intune Plan 1 | Baseline requirement for most enterprise designs. |
| Device-based or user-based access controls | Microsoft Entra ID P1 or P2 | Required for many Conditional Access and identity controls. |
| Defender risk and security signal integration | Microsoft Defender for Endpoint | Strong fit where device risk should influence compliance or access. |
| Advanced endpoint privilege, remote support, certificate, or app catalog capabilities | Intune Suite or eligible add-ons | Buy for specific operational outcomes, not by default. |
| Shared device pools | Device licensing or correct user licensing model | Match licensing to the ownership and sign-in model. |
| Education-specific administration | Intune for Education plus supporting education licensing | Use education workflows where schools need simplified administration. |

### License Dependency Matrix

Use this table to prevent control designs that exceed current entitlements.

| Capability Area | Minimum Common Dependency | Typical Enterprise Dependency Pattern | Notes |
| --- | --- | --- | --- |
| Baseline Conditional Access and app protection controls | Microsoft Entra ID P1 + Intune Plan 1 | E3/E5 with Entra P1 and Intune Plan 1 | Confirm exclusions, report-only rollout, and emergency account design. |
| Risk-based access with device risk signals | Defender for Endpoint + Microsoft Entra ID P2 | E5 Security / M365 E5 style bundles | Do not enforce device-risk-based CA until onboarding coverage is stable. |
| Advanced endpoint protection operations (ASR maturity, incident integration depth) | Defender for Endpoint Plan 2 (common pattern) | Security-focused E5-aligned stack | Validate SOC operating model before enabling strict enforcement. |
| Endpoint privilege management | Intune Suite add-on | Intune Plan 1 + Intune Suite | Fit to least-privilege and local admin reduction strategy. |
| Remote support with role-based controls | Intune Suite Remote Help add-on | Intune Plan 1 + Intune Suite | Use only where support workflow and audit requirements justify it. |
| Cloud PKI and certificate lifecycle simplification | Intune Suite Cloud PKI add-on | Intune Plan 1 + Intune Suite | Validate cloud and regulatory constraints, especially in sovereign environments. |

Licensing should be reviewed as part of every security design decision, not only during procurement. Keep a live dependency register for Conditional Access, Defender integration, Intune Suite features, and any role-based support tooling.

Use Intune Suite when there is a measurable problem to solve, such as privilege elevation, remote support, certificate lifecycle simplification, or third-party application standardization. Intune add-ons are currently not supported in sovereign clouds, so confirm cloud constraints before including them in target-state architecture.

## Tenant Hygiene and Governance

Large tenants fail operationally long before they fail technically. Governance should reduce ambiguity.

Recommended practices:

- Define naming standards for policies, profiles, apps, scripts, baselines, rings, groups, and filters.
- Use RBAC and scope tags to limit administrative blast radius and support distributed IT models.
- Use virtual groups, usually Microsoft Entra dynamic groups, only where the rule logic is stable and supportable.
- Use assignment filters to refine targeting by device properties instead of duplicating groups unnecessarily.
- Keep every object owned, reviewed, and retireable.
- Record exception decisions and link them to business owners.

Avoid:

- A single global admin pattern for all endpoint work.
- Dynamic groups with unclear logic or no lifecycle owner.
- Recreating nearly identical profiles for every business unit.
- Leaving everything on the default scope tag.

## Identity and Access Foundation

Identity is the control point for Zero Trust. Device management and access policy must be designed together.

Recommended approach:

- Default new corporate Windows devices to Microsoft Entra join.
- Use Conditional Access as the enforcement plane for MFA, device trust, session conditions, and app protection requirements.
- Keep emergency access accounts excluded from normal Conditional Access enforcement and test them regularly.
- Separate admin access policy from user access policy.
- Use passwordless methods where operationally mature.

The MDM user scope must be set intentionally. Set it to `Some` or `All` only for the users who should automatically enroll, keep WIP scope separate, and avoid accidental overlap that changes enrollment behavior.

## Enrollment Strategy

Design enrollment by device persona and ownership model, not by administrative convenience.

| Persona | Preferred Pattern | Notes |
| --- | --- | --- |
| Corporate Windows | Windows Autopilot plus automatic enrollment | Standard path for mainstream enterprise endpoints. |
| BYOD mobile | App protection without full device enrollment where possible | Lower friction, better privacy posture, and usually enough control for productivity apps. |
| Shared Windows | Shared device pattern with device-centric assignment | Avoid user-centric policy assumptions. |
| Frontline dedicated mobile | Automated enrollment with shared device mode where supported | Optimize for shift sign-in and fast reprovisioning. |
| Education | Intune for Education or school-managed workflow | Separate student, teacher, and classroom policies. |
| Legacy or specialty devices | Exception path with explicit controls | Do not force standard enrollment where the device cannot support it cleanly. |

Enrollment rules:

- Use automatic enrollment deliberately, not broadly by habit.
- Keep corporate and BYOD paths separate.
- Tie enrollment decisions to access policy, app model, and support model.
- Do not require full MDM enrollment when app protection plus Conditional Access will meet the business requirement.

## Hybrid-to-Cloud Transition Guidance

Move from hybrid to cloud in waves. Do not announce a cloud-only end state until dependencies are inventoried and retirement sequencing is real.

Recommended sequence:

1. Inventory current join states, enrollment methods, GPOs, Configuration Manager workloads, PKI dependencies, scripts, VPN and Wi-Fi profiles, and legacy authentication dependencies.
2. Define the target state for new devices first.
3. Pilot Microsoft Entra join, Autopilot, Intune policy, update rings, compliance, and app packaging with a controlled cohort.
4. Migrate by persona, not by org chart alone.
5. Retire legacy controls after the cloud path is stable, measurable, and supportable.

Transition guidance:

- Start with new device provisioning and refresh cycles before in-place conversion at scale.
- Keep Group Policy analytics and settings rationalization as active workstreams.
- Rebuild only the settings that still matter; do not replicate old GPO sprawl in Intune.
- Keep certificate, wireless, VPN, and line-of-business authentication dependencies visible early.

## Co-Management vs Intune-Only Decision Guidance

Intune-only is the preferred destination for most modern enterprise endpoints. Co-management is usually the bridge.

| Choose Co-Management When | Choose Intune-Only When |
| --- | --- |
| Configuration Manager still owns a workload that cannot yet move cleanly. | Identity, apps, servicing, compliance, and support are ready for a single cloud-first control plane. |
| You need to transition workloads in phases with minimal disruption. | New device builds are Microsoft Entra joined and Intune-led by default. |
| You still depend on Configuration Manager content, tasking, or on-prem workflows. | Dual-management complexity no longer provides enough value. |
| Pilot workload shifts are needed before broad change. | Operations, reporting, and troubleshooting improve by removing split authority. |

Co-management rules:

- Move one workload at a time.
- Pilot every workload shift.
- Document the exit criteria for every retained Configuration Manager dependency.
- Do not leave workloads split indefinitely without a retirement plan.

## Windows Configuration Profiles Best Practices

For Windows, configuration profiles are the day-to-day policy delivery mechanism for device posture, UX standards, and operational controls. Treat profile design as an engineering system, not a one-time setup.

### Design Model

- Use Settings Catalog as the default authoring path for new Windows policy where possible.
- Use template-based profiles only when they provide a clear operational advantage.
- Keep one authoritative source per setting family to reduce conflicts across Settings Catalog, Administrative Templates, security baselines, and endpoint security policies.
- Prefer reusable profile layers:
	- Foundation baseline layer (tenant-wide standards)
	- Persona layer (corporate, shared, frontline, kiosk, EDU)
	- Exception layer (time-bounded and owner-approved)

### Assignment and Targeting Strategy

- Prefer device-based assignment for machine controls and shared-device scenarios.
- Use user-based assignment only when settings are truly user-context controls.
- Use assignment filters to refine targeting instead of cloning near-identical groups.
- Roll out profiles in rings (pilot -> broad -> holdback) with explicit success and rollback criteria.
- Keep assignment intent simple and auditable; avoid overlapping broad assignments that are hard to troubleshoot.

### Naming, Versioning, and Ownership

- Standardize naming so each profile includes platform, scope, purpose, and version (for example: `WIN11-DEVICE-BASELINE-v3`).
- Assign an owner and review cadence to every profile.
- Document change reason, expected impact, and rollback path before promotion from pilot to broad.
- Retire deprecated profiles promptly to avoid assignment drift and stale policy conflicts.

### Conflict and Drift Prevention

- Avoid managing the same setting from multiple policy channels unless intentionally documented.
- Review policy conflict reports and remediation states on a fixed cadence.
- Validate baseline and endpoint security overlaps before enabling broad assignments.
- Keep exception debt visible: every exception should have business owner, expiry date, and review date.

### Operational Rollout Checklist (Windows Profiles)

- Validate prerequisites (enrollment state, edition support, and licensing dependencies).
- Test settings in a pilot ring with representative hardware and business apps.
- Confirm reboot behavior, user impact, and help-desk readiness before broad rollout.
- Monitor deployment status, error codes, and remediation trends daily during rollout waves.
- Freeze expansion when repeated failures exceed threshold, then run root-cause analysis.

### Common Windows Profile Anti-Patterns

- Creating many near-duplicate profiles per business unit with no lifecycle owner.
- Mixing device and user settings in the same rollout without clear scope intent.
- Deploying broad assignments before pilot telemetry is stable.
- Allowing legacy GPO intent to be copied directly without cloud-era simplification.
- Leaving old profiles assigned after replacement, causing unpredictable effective settings.

## App Lifecycle and Win32 Packaging

At enterprise scale, application management needs engineering discipline, not ad hoc packaging.

Recommended practices:

- Prefer Win32 apps for complex Windows application delivery.
- Standardize packaging inputs, install commands, uninstall commands, detection rules, dependencies, supersedence, and restart expectations.
- Keep packaging ownership separate from software approval ownership.
- Use Delivery Optimization and content design intentionally for bandwidth-heavy deployments.
- Build versioned app baselines so rollback and supersedence remain predictable.

Operational specifics to preserve:

- The Intune Management Extension installs automatically when a Win32 app or a PowerShell script is assigned to a user or device.
- The Win32 app size limit is 30 GB per app.
- In classic Windows Autopilot enrollment that uses the Enrollment Status Page, avoid mixing Win32 and line-of-business apps in the same provisioning flow because app installation can fail. Win32 and line-of-business app mixing is supported in Windows Autopilot device preparation.

Packaging checklist:

- Use deterministic detection logic.
- Define a tested uninstall path.
- Model dependencies explicitly.
- Use supersedence for version transitions.
- Document whether installs are device-context or user-context.

## Update Rings and Feature Updates

Windows servicing should be opinionated and ring-based.

Use a Windows 11-first strategy for mainstream endpoints. Windows 10 should remain only on transition exceptions, ESU-covered populations, or hardware that cannot yet move. Keep LTSC and other fixed-function devices on separate servicing tracks with their own policy, app, and support assumptions.

| Track | Purpose | Typical Population |
| --- | --- | --- |
| Pilot ring | Validate updates and app compatibility early | IT, engineering, support champions |
| Broad ring | Standard enterprise servicing | Most corporate user endpoints |
| Holdback ring | Short-term containment for known issues | Limited exception populations |
| LTSC or fixed-function track | Stability-first servicing | Kiosk, OT-adjacent, fixed-purpose, validated devices |

Servicing rules:

- Use separate policies for quality updates, feature updates, drivers, and firmware strategy.
- If both an update ring and a feature update policy apply to a device, set the ring feature update deferral to `0`.
- Keep rollback, pause, and communication procedures documented before broad deployment.
- Validate app readiness and device readiness before moving feature update waves.

## Security Baselines and Endpoint Protection

Use Intune security baselines as a starting point, then refine with endpoint security policies and Settings Catalog profiles where needed.

Recommended practices:

- Start with Microsoft security baselines, then customize only where operationally justified.
- Keep a baseline ownership model and version review cadence.
- Use endpoint security profiles for antivirus, firewall, disk encryption, attack surface reduction, and account protection.
- Integrate Microsoft Defender for Endpoint where risk, compliance, and response workflows should influence one another.
- Use Intune Suite capabilities such as Endpoint Privilege Management, Remote Help, Cloud PKI, or Enterprise App Management only where the operating model clearly benefits.

Avoid deploying overlapping baselines and profiles without conflict review.

### Endpoint Security Operating Model

Endpoint security needs a deploy-operate-measure model, not just profile assignment.

#### 1) Ownership and policy layering

- Define one owner for each control family: antivirus, ASR, firewall, disk encryption, account protection, and local admin governance.
- Separate baseline policy from exception policy. Every exception should include owner, reason, expiry, and review date.
- Avoid conflicts by assigning each setting family to one authoritative policy type where practical, then document intentional overlaps.

#### 2) Defender for Endpoint onboarding scope

- Roll out onboarding in rings with explicit exit criteria:
	- Ring 0: security engineering and endpoint engineering pilot devices.
	- Ring 1: standard corporate managed endpoints.
	- Ring 2: shared, frontline, and fixed-function populations with explicit exception handling.
- Treat sensor health and onboarding coverage as prerequisites before using risk-based access enforcement broadly.
- Keep rollback controls documented so containment actions can be scoped by ring when false-positive activity spikes.

#### 2a) Antivirus and Defender policy ownership

- Use one authoritative policy model for Microsoft Defender Antivirus settings to avoid hidden conflicts.
- Separate baseline policy from exception policy and keep exception scope minimal.
- Document owners for real-time protection, cloud-delivered protection, sample submission, scan schedule, exclusions, and remediation defaults.
- Keep exclusions time-bounded, jointly approved by application and security owners, and tracked as exception debt.

#### 3) Attack Surface Reduction rollout

- Start new ASR rules in audit mode for pilot populations.
- Promote rules to block mode in waves after compatibility review and app-owner validation.
- Maintain a rule register with mode, owner, exception list, and retirement date for exceptions.
- Do not disable an entire ASR profile for one incompatible app; isolate to a targeted exception pattern.

| ASR Ring | Goal | Typical Duration | Exit Criteria |
| --- | --- | --- | --- |
| Lab | Validate behavior in controlled devices | 1-2 weeks | No critical app or admin workflow breakage |
| Pilot | Validate with IT, security, and engineering cohorts | 2-4 weeks | Acceptable audit findings and documented mitigations |
| Broad | Enforce on standard corporate populations in waves | 4-8 weeks by wave | Help-desk and block telemetry remain within threshold |
| Holdback | Temporary exception population only | Time-bounded | Exception retired or alternate control documented |

ASR rollout operations:

- Review ASR telemetry weekly during rollout waves.
- Promote to broad enforcement only after rollback steps are tested.
- Keep a per-rule audit-to-block change log with owner and approval record.

#### 4) Firewall baseline strategy

- Keep Windows Defender Firewall enabled for domain, private, and public profiles unless an exception is approved.
- Use baseline inbound deny posture with explicit allow rules tied to business-owned requirements.
- Maintain a time-bounded emergency exception workflow and review temporary rules on a fixed cadence.

#### 5) Disk encryption and recovery operations

- Enforce BitLocker with recovery key escrow validation before broad rollout.
- Define help-desk and security runbooks for identity verification, key retrieval approval, logging, and post-recovery review.
- Track escrow coverage as an operational metric and treat non-escrowed encrypted devices as policy exceptions.

#### 6) Tamper protection and Windows LAPS governance

- Enable tamper protection by default for supported managed endpoints and use a documented, time-bounded process for rare support exceptions.
- Use Windows LAPS with explicit RBAC, retrieval auditing, and break-glass approvals.
- Require post-use review for local admin password retrieval in privileged or incident contexts.

#### 7) Incident response and SIEM integration

- Define escalation flow across endpoint engineering, SOC, and identity teams.
- Correlate Intune compliance state, Defender incident context, and sign-in/access signals in Sentinel or another SIEM.
- Preserve device evidence before destructive lifecycle actions when a security incident is active.

| Security Scenario | Intune Action | Defender/SOC Action | Identity/Access Action |
| --- | --- | --- | --- |
| Suspected malware on standard endpoint | Collect device context and isolate if policy allows | Validate incident severity and containment path | Evaluate sign-in risk and apply step-up controls |
| Confirmed compromise on privileged workstation | Pause lifecycle deletion and capture investigation artifacts | Escalate to high-severity response workflow | Revoke sessions, force stronger auth, evaluate account impact |
| Repeated policy tamper indicators | Validate policy assignments and device state | Investigate tamper events and persistence behaviors | Restrict risky access paths during investigation |

Incident RACI baseline:

| Function | Primary Responsibility |
| --- | --- |
| Endpoint engineering (EUC) | Policy deployment, device containment execution, lifecycle control gates |
| SOC | Detection triage, incident coordination, escalation decisions |
| Identity team | Conditional Access response, account/session risk actions |
| Help desk | Approved recovery workflows and ticket-linked operational support |

## Compliance and Conditional Access Alignment

Compliance policy is the device posture signal. Conditional Access is the enforcement layer. They must be designed together.

Recommended practices:

- Build compliance policies before enabling compliant-device access requirements.
- Set the tenant compliance default so devices with no compliance policy assigned are treated as not compliant when compliance drives access.
- Use report-only mode for Conditional Access before enforcement.
- Exclude emergency access accounts from normal enforcement.
- Use device risk signals where Defender integration is in scope.

Access design guidance:

- Corporate managed devices: require compliant device where the app and user workflow justify it.
- BYOD productivity access: prefer app-based Conditional Access and app protection policy where full enrollment is unnecessary.
- Administrative access: use stricter controls than standard user access.

### Device Risk and Access Decision Patterns

Use explicit decision patterns so report-only and enforcement phases remain predictable.

| Scenario | Recommended Conditional Access Pattern | Operational Notes |
| --- | --- | --- |
| Managed corporate access to high-value apps | Require compliant device, then add risk-based controls where licensing and onboarding support it | Roll out in report-only mode first and validate blast radius. |
| BYOD productivity access | Require approved client app and app protection policy instead of full MDM where possible | Reduces friction while preserving data controls. |
| Privileged admin access | Require stronger controls than standard users (device trust, strong auth, narrow exclusions) | Keep admin access policy separated from user policy. |
| Shared or non-persistent device workflows | Use persona-specific exceptions with compensating controls and short review cadence | Do not copy standard named-user policy without validation. |

Risk-driven access enforcement should not be treated as day-1 default. Enable only after onboarding coverage, sensor health, and exception governance are mature.

## Shared, Frontline, and Education Design Branches

These scenarios should be designed as explicit branches of the environment.

### Windows Shared PC Mode

Use Windows shared PC mode for Windows devices that are intentionally shared by multiple users one at a time. Favor device-based app assignment, minimal user personalization, predictable sign-in behavior, and simplified recovery. For shared pools, operationally prefer reprovisioning over hand-tuning drifted devices.

### Mobile Shared Device Mode

Use mobile shared device mode for eligible frontline iOS and iPadOS devices that pass between workers during the day. Build for fast sign-in and sign-out, device-based deployment, minimal required apps, and predictable session reset behavior.

### Frontline BYOD

For personally owned frontline devices, default to app protection policy plus app-based Conditional Access before considering full MDM enrollment. Protect data at the app layer, require approved client apps where appropriate, and avoid imposing a corporate device-management model where it is not required.

### Intune for Education or School-Managed Workflows

Use Intune for Education or school-managed workflows where the environment is optimized for student, teacher, and classroom administration rather than general enterprise endpoint operations. Separate student shared devices, teacher productivity devices, and classroom or assessment workflows. Use school-specific enrollment methods and Apple School Manager or education enrollment processes where they fit the operating model.

## Device Lifecycle and Cleanup

Lifecycle hygiene keeps assignments, reporting, and support accurate.

Recommended practices:

- Define when to retire, wipe, delete, or reprovision.
- Align offboarding with HR, asset, and support processes.
- Keep reassignment workflows explicit for shared pools and classroom sets.
- Review stale device records regularly.

Operational specifics to preserve:

- Cleanup rules hide inactive records from the Intune admin experience.
- Cleanup rules do not wipe or retire devices.
- Hidden devices can reappear if they check in again before certificate expiry.

### Recovery, Forensics, and Incident Preservation

Lifecycle actions should account for active security investigations.

- Define a hold process that pauses wipe, delete, and reprovisioning for devices under active incident investigation.
- Preserve required evidence before destructive actions, including timeline, device state, and ticket linkage.
- Separate rapid containment workflows from final remediation workflows so forensic value is not lost.
- Document when to choose isolate, retire, wipe, or reprovision based on incident severity and recovery objectives.

## Reporting and Monitoring

Use reporting to operate the environment, not just to present status.

Recommended KPIs:

- Enrollment success rate
- Compliance rate
- App deployment success rate
- Update latency by ring
- Security baseline drift
- Support volume after major policy changes
- Stale device count and cleanup trend

Recommended tooling:

- Built-in Intune operational reporting for day-to-day troubleshooting
- Windows feature update readiness and organizational update reporting
- Endpoint analytics for performance and experience trends
- Log Analytics or other enterprise reporting platforms for cross-domain views

### Security KPIs and SLOs

Add measurable targets so endpoint security operations can enforce gates, not just report trends.

| Metric | Example Target | Review Cadence | Trigger Action |
| --- | --- | --- | --- |
| Defender onboarding coverage (target populations) | >= 98% | Weekly | Pause new risk-based CA rollout if below target for two periods. |
| Sensor health coverage | >= 97% of onboarded in-scope endpoints reporting healthy telemetry | Weekly | Investigate onboarding drift and connector health. |
| ASR production-rule health | No critical exception older than 30 days | Weekly | Escalate exception owner and require remediation plan. |
| BitLocker escrow coverage on encrypted corporate devices | >= 99% | Weekly | Block broader enforcement waves until escrow gap is closed. |
| Firewall baseline compliance | >= 98% on managed corporate endpoints | Weekly | Review conflicting policies and exception sprawl. |
| LAPS rotation and retrieval audit health | >= 99% successful rotation; 100% audited retrieval events | Monthly | Investigate role misuse and adjust RBAC scope tags. |
| Mean time to triage (high severity) | <= 30 minutes | Weekly | Trigger SOC and endpoint staffing review. |
| High-severity endpoint incident MTTR | <= 4 business hours baseline | Weekly | Trigger joint endpoint/SOC RCA and containment playbook review. |
| Exception debt | 0 critical exceptions past review date | Weekly | Freeze expansion in affected ring until debt is reduced. |

SLOs should be ring-aware. Pilot rings can tolerate exploratory variance; broad rings should enforce stricter thresholds and freeze expansion when thresholds are repeatedly missed.

Example security SLOs:

- Critical onboarding failures are triaged within 1 business day.
- High-severity incidents are owned within 30 minutes and containment begins within 2 hours.
- Recovery key access events are reviewed within 1 business day.
- ASR exceptions are reviewed monthly and expire unless renewed.
- Sustained firewall or BitLocker drift above threshold triggers service review.

## Practical Implementation Checklists

### Design Checklist

- Validate licensing, cloud type, and Intune add-on availability.
- Define target personas, support boundaries, and exceptions.
- Decide the target state for new Windows devices.
- Establish group, filter, scope tag, and RBAC strategy.
- Map legacy dependencies that block cloud-only management.

### Pilot Checklist

- Validate automatic enrollment and device registration behavior.
- Validate Autopilot and enrollment experiences for each target persona.
- Validate app install, update, uninstall, and reboot behavior.
- Validate compliance evaluation and Conditional Access impact in report-only mode.
- Validate update ring and feature update behavior in pilot cohorts.

### Rollout Checklist

- Communicate the new support and access model.
- Expand by ring, persona, and business criticality.
- Monitor enrollment, app, compliance, and update telemetry daily.
- Pause on repeated failure patterns.
- Track every exception and assign an owner.

### Operations Checklist

- Review stale records, cleanup rules, and device lifecycle actions.
- Review policy overlap, filter sprawl, and dynamic group drift.
- Review ring health, rollout timing, and feature-update readiness.
- Review packaging standards, supersedence chains, and failed deployments.
- Review privileged access, break-glass health, and support escalation quality.

## Common Anti-Patterns

- Treating Intune as a one-for-one GPO replacement instead of a cloud control plane.
- Leaving the MDM user scope broad or ambiguous.
- Keeping co-management indefinitely because no one owns the exit plan.
- Allowing the same setting to be managed by Intune, Group Policy, and Configuration Manager at the same time.
- Designing one enrollment pattern for every persona.
- Forcing full MDM enrollment on BYOD when app protection would meet the requirement.
- Mixing Win32 and line-of-business apps during classic Windows Autopilot ESP enrollment without understanding the constraint.
- Running the entire Windows estate on a single servicing ring.
- Using Windows 10 as the default mainstream enterprise endpoint platform after October 14, 2025.
- Assuming cleanup rules retire or wipe stale devices.
- Buying Intune Suite capabilities without a defined operating problem to solve.

## Source Map

Official Microsoft Learn references used to align this guide:

| Topic | Microsoft Learn |
| --- | --- |
| Intune overview and planning | https://learn.microsoft.com/en-us/intune/ |
| Intune licensing | https://learn.microsoft.com/en-us/intune/fundamentals/licensing/ |
| Assign Intune licenses | https://learn.microsoft.com/en-us/intune/fundamentals/licensing/assign-licenses |
| Intune add-ons and sovereign cloud limitation | https://learn.microsoft.com/en-us/intune/intune-service/fundamentals/intune-add-ons |
| Windows automatic enrollment and MDM user scope | https://learn.microsoft.com/en-us/intune/intune-service/enrollment/windows-enroll |
| Assignment filters | https://learn.microsoft.com/en-us/intune/intune-service/fundamentals/filters |
| RBAC and scope tags | https://learn.microsoft.com/en-us/intune/intune-service/fundamentals/scope-tags |
| Co-management overview | https://learn.microsoft.com/en-us/intune/configmgr/comanage/overview |
| Win32 app management, IME behavior, 30 GB limit, and Autopilot caution | https://learn.microsoft.com/en-us/intune/intune-service/apps/apps-win32-app-management |
| Windows Autopilot device preparation FAQ | https://learn.microsoft.com/en-us/autopilot/device-preparation/faq |
| Windows feature update policy guidance | https://learn.microsoft.com/en-us/intune/device-updates/windows/feature-update-policy |
| Device cleanup rules | https://learn.microsoft.com/en-us/intune/governance/device-cleanup-rules |
| Intune security baselines | https://learn.microsoft.com/en-us/intune/intune-service/protect/security-baselines |
| Endpoint security policy framework in Intune | https://learn.microsoft.com/en-us/intune/intune-service/protect/endpoint-security-policy |
| Manage attack surface reduction policy in Intune | https://learn.microsoft.com/en-us/intune/intune-service/protect/endpoint-security-asr-policy |
| Manage firewall policy in Intune endpoint security | https://learn.microsoft.com/en-us/intune/intune-service/protect/endpoint-security-firewall-policy |
| Manage disk encryption policy in Intune endpoint security | https://learn.microsoft.com/en-us/intune/intune-service/protect/endpoint-security-disk-encryption-policy |
| Account protection and Windows LAPS policy in Intune | https://learn.microsoft.com/en-us/intune/intune-service/protect/endpoint-security-account-protection-policy |
| Device compliance policy guidance | https://learn.microsoft.com/en-us/intune/intune-service/protect/device-compliance-get-started |
| Require device compliance with Conditional Access | https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-all-users-device-compliance |
| Use Defender for Endpoint with Intune | https://learn.microsoft.com/en-us/intune/intune-service/protect/mde-security-integration |
| Microsoft Defender tamper protection overview | https://learn.microsoft.com/en-us/defender-endpoint/prevent-changes-to-security-settings-with-tamper-protection |
| App protection policy overview | https://learn.microsoft.com/en-us/intune/intune-service/apps/app-protection-policy |
| App-based Conditional Access | https://learn.microsoft.com/en-us/intune/intune-service/protect/app-based-conditional-access-intune-create |
| Windows device profiles overview | https://learn.microsoft.com/en-us/intune/intune-service/configuration/device-profiles |
| Settings Catalog in Intune | https://learn.microsoft.com/en-us/intune/intune-service/configuration/settings-catalog |
| Administrative Templates for Windows in Intune | https://learn.microsoft.com/en-us/intune/intune-service/configuration/administrative-templates-windows |
| Windows shared PC mode settings | https://learn.microsoft.com/en-us/intune/intune-service/configuration/shared-user-device-settings-windows |
| Shared device mode enrollment for iOS/iPadOS | https://learn.microsoft.com/en-us/intune/intune-service/enrollment/device-enrollment-program-enroll-ios |
| Intune for Education overview | https://learn.microsoft.com/en-us/intune-education/what-is-intune-for-education |
| School enrollment guidance | https://learn.microsoft.com/en-us/intune-education/how-should-i-enroll-devices |
| Endpoint analytics | https://learn.microsoft.com/en-us/intune/endpoint-analytics/ |
| Microsoft Sentinel Intune data connector | https://learn.microsoft.com/en-us/azure/sentinel/data-connectors-reference#microsoft-intune |
| Microsoft Sentinel Defender XDR connector | https://learn.microsoft.com/en-us/azure/sentinel/data-connectors-reference#microsoft-defender-xdr |
| Microsoft Defender XDR incident response overview | https://learn.microsoft.com/en-us/defender-xdr/investigate-incidents |
| Windows 10 end of support and migration context | https://learn.microsoft.com/en-us/lifecycle/announcements/windows-10-end-of-support |
