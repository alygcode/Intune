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
| Intune licensing | https://learn.microsoft.com/en-us/intune/intune-service/fundamentals/licenses |
| Assign Intune licenses | https://learn.microsoft.com/en-us/intune/fundamentals/licensing/assign-licenses |
| Intune add-ons and sovereign cloud limitation | https://learn.microsoft.com/en-us/intune/intune-service/fundamentals/intune-add-ons |
| Windows automatic enrollment and MDM user scope | https://learn.microsoft.com/en-us/intune/intune-service/enrollment/windows-enroll |
| Assignment filters | https://learn.microsoft.com/en-us/mem/intune/fundamentals/filters |
| RBAC and scope tags | https://learn.microsoft.com/en-us/intune/intune-service/fundamentals/scope-tags |
| Co-management overview | https://learn.microsoft.com/en-us/intune/configmgr/comanage/overview |
| Win32 app management, IME behavior, 30 GB limit, and Autopilot caution | https://learn.microsoft.com/en-us/mem/intune/apps/apps-win32-app-management |
| Windows Autopilot device preparation FAQ | https://learn.microsoft.com/en-us/autopilot/device-preparation/faq |
| Windows feature update policy guidance | https://learn.microsoft.com/en-us/intune/device-updates/windows/feature-update-policy |
| Device cleanup rules | https://learn.microsoft.com/en-us/intune/intune-service/fundamentals/device-cleanup-rules |
| Intune security baselines | https://learn.microsoft.com/en-us/mem/intune/protect/security-baselines |
| Device compliance policy guidance | https://learn.microsoft.com/en-us/mem/intune/protect/device-compliance-get-started |
| Require device compliance with Conditional Access | https://learn.microsoft.com/en-us/mem/intune/protect/create-conditional-access-intune |
| App protection policy overview | https://learn.microsoft.com/en-us/mem/intune/apps/app-protection-policy |
| App-based Conditional Access | https://learn.microsoft.com/en-us/intune/intune-service/protect/app-based-conditional-access-intune-create |
| Windows shared PC mode settings | https://learn.microsoft.com/en-us/intune/intune-service/configuration/shared-user-device-settings-windows |
| Shared device mode enrollment for iOS/iPadOS | https://learn.microsoft.com/en-us/intune/intune-service/enrollment/device-enrollment-program-enroll-ios |
| Intune for Education overview | https://learn.microsoft.com/en-us/intune-education/what-is-intune-for-education |
| School enrollment guidance | https://learn.microsoft.com/en-us/intune-education/how-should-i-enroll-devices |
| Endpoint analytics | https://learn.microsoft.com/en-us/intune/analytics/overview |
| Windows 10 end of support and migration context | https://learn.microsoft.com/en-us/lifecycle/announcements/windows-10-end-of-support |
