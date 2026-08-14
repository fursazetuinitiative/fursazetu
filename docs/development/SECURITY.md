# SECURITY.md

Version: 1.0

Status: Active

Project: FursaZetu

Website: https://fursazetu.gt.tc


# 1. Purpose

This document defines FursaZetu's security policy: how access, credentials, data, and AI-assisted workflows are protected across the platform.

It was flagged as a missing document during the initial project review and again in ARCHITECTURE.md Section 16, which referenced it as a placeholder. This document replaces that placeholder.

This document governs how FursaZetu is operated securely. It does not re-document the internal security engineering already built into the Royal MCP plugin itself (API key hashing, rate limiting, per-tool WordPress capability checks, OAuth 2.1 with PKCE, SSRF protection, sensitive-key redaction) — that is Royal MCP's own responsibility and is already covered in its readme and changelog. This document covers everything upstream and downstream of that: who gets credentials, what roles exist, how backups and updates are handled, and how AI-assisted curation is kept inside safe boundaries.


---

# 2. Security Philosophy

Security at FursaZetu follows the same discipline as every other part of the project: proportionate, not paranoid, and never a substitute for editorial judgment.

- **Trust is the product.** A security failure that lets bad content reach a user is functionally the same failure EDITORIAL_POLICY.md exists to prevent — inaccurate or fraudulent information reaching someone trying to build their future. Security and editorial integrity are the same goal approached from different directions.
- **Least privilege by default.** Every account, API key, and automated workflow should have the minimum access it needs to do its job — not the maximum access that happens to be convenient.
- **Verify before trusting automation.** Per OPPORTUNITY_MODEL.md Section 14, AI-assisted workflows are powerful but bounded; this document extends that boundary into account- and credential-level policy.
- **Assume mistakes will happen; make them recoverable.** Backups, audit logs, and an Archived (not deleted) content lifecycle exist so that a mistake — human or AI — is a correction, not a disaster.


---

# 3. Scope

This document covers:

- WordPress user roles and capability policy for FursaZetu specifically.
- How Royal MCP access (API keys, OAuth connections) is issued and governed.
- Backup, update, and recovery policy.
- Data privacy for the limited personal data FursaZetu collects.
- Incident response and audit practices.

This document does not cover:

- Royal MCP's internal implementation (already documented by the plugin itself).
- Hosting-provider-level infrastructure security (firewalls, server hardening) — a hosting-specific addendum should be added once a production host is finalized, without needing to revise this document's policy layer.


---

# 4. WordPress User Roles & Access Control

| Role | Who holds it | Capabilities | Royal MCP access |
|---|---|---|---|
| Administrator | Project owner(s) only | Full site control, plugin/theme management, option writes | Full — including toggles like "Allow AI to write WordPress options" (kept off per ARCHITECTURE.md Section 16) |
| Editor (Curator) | Content curators | Create, edit, publish opportunities and pages; manage taxonomies and media | Scoped — see Section 6 on OAuth vs. API key |
| Contributor | Reserved for future external-submission workflow (ARCHITECTURE.md Section 19) | Create drafts only; cannot publish | Not applicable until that workflow exists |
| Subscriber | Unused | No admin-area access | None |

No account is created above the minimum role its holder needs. Administrator accounts are limited to the smallest practical number of people, consistent with WordPress's own guidance that every additional admin account is a larger attack surface.


---

# 5. AI / Royal MCP Access Policy

Royal MCP is the mechanism by which AI tools act on the WordPress site. Two authentication paths exist, and they carry meaningfully different risk:

## Static API key

Per Royal MCP's own documentation, requests authenticated with the static API key run as an **administrator**, regardless of who is actually using the key. This means a static API key is an admin-equivalent credential in practice, even if the intent behind issuing it was narrower.

**Policy:** the static API key is treated as an Administrator secret. It is issued only for trusted automation (e.g. a scheduled task, or a single trusted operator's tooling), never distributed to multiple curators as a shared convenience credential, and rotated (via Royal MCP's Regenerate button) if there is any reason to suspect it has been exposed.

## OAuth (per-user connector)

When a curator connects via Claude.ai's or Claude Desktop's native OAuth connector flow, Royal MCP's per-tool WordPress capability checks (`current_user_can`) apply against **that curator's own WordPress account**, not against an administrator identity. This means an Editor-role curator connected via OAuth is correctly restricted to Editor-level actions even when working through an AI tool.

**Policy:** OAuth is the preferred connection method for any curator who is not themselves an Administrator. This is the practical mechanism by which "AI access should match the human's actual role" is enforced, rather than relying on the AI to self-restrict.

## General MCP policy

- The "Allow AI to write WordPress options" and "Allow AI to modify theme appearance" toggles (Royal MCP → Settings) remain **off** by default, per ARCHITECTURE.md Section 16. Nothing in the standard opportunity workflow requires either.
- Destructive tools (`wp_delete_post`, `wc_delete_*`, etc.) are not part of the standard curation workflow described in ARCHITECTURE.md Section 13; Archived status is the correct outcome for content that should no longer be public.
- If a new MCP-driven workflow is proposed that would need either toggle enabled, it should be reviewed against this document and OPPORTUNITY_MODEL.md Section 14 before being turned on — not enabled reactively to unblock a single task.


---

# 6. Credential Management

- WordPress admin passwords follow standard strong-password practice (unique, high-entropy, stored in a password manager — not reused across services).
- The Royal MCP static API key (Section 5) is stored the same way — never in plaintext in documentation, chat logs, or version control.
- OAuth client credentials, when a static Client ID/Secret is configured rather than Dynamic Client Registration, follow the same handling as the API key.
- `wp-config.php` and all local secrets are excluded from version control via `.gitignore` (already in place) and are never committed, including in example/placeholder form that could be mistaken for a real value.
- If a credential is suspected compromised: rotate the API key immediately via Royal MCP's Regenerate button, and use the "Reset OAuth State" action to invalidate all issued OAuth tokens and force re-authorization.


---

# 7. Content Integrity & Verification Security

Security and editorial trust intersect directly in the opportunity publishing pipeline:

- Only Editor-role-or-above accounts (Section 4) can move an opportunity from Pending Verification to Published, matching OPPORTUNITY_MODEL.md Section 3's rule that only a human curator makes that transition.
- The Verification Status field (`opp_verification_status`) is not writable by the automated Deadline Engine (ARCHITECTURE.md Section 10) — it is a purely editorial field, and its access should be treated the same as a publish action, not a routine content edit.
- Future external-submission workflows (ARCHITECTURE.md Section 19) must create records in Draft status only, with no path that allows a submitter's content to reach Published without passing through the same Publishing Checklist (OPPORTUNITY_MODEL.md Section 17) as curator-sourced content.


---

# 8. Data Privacy

FursaZetu currently collects limited personal data, consistent with its early-phase feature set (SITE_STRUCTURE.md, Contact page; ROADMAP.md Phase 2, Newsletter):

- **Contact form submissions**: name, email, message. Retained only as long as needed to respond, not used for unrelated marketing without explicit opt-in.
- **Newsletter signups**: email address only. Requires clear opt-in and an unsubscribe path, per general good practice and in anticipation of future compliance needs as the audience grows beyond Kenya.
- **No user accounts exist yet** (Phase 3 of ROADMAP.md). When accounts are introduced, this document should be revised to cover authentication policy, password storage, and account-data retention before that phase ships.
- A Privacy Policy page (already listed in SITE_STRUCTURE.md's footer) should state plainly what is collected and why, matching this section — the footer link should not be a placeholder once real data collection (contact form, newsletter) is live.


---

# 9. Backup & Recovery

- Automated backups run on a schedule independent of any AI session — the same principle as the Deadline Engine (ARCHITECTURE.md Section 10): recovery capability must not depend on someone remembering to trigger it.
- If Royal MCP's SiteVault integration is the backup mechanism in use, scheduled backups (`sv_get_schedules`) are the primary mechanism; AI-triggered on-demand backups (`sv_create_backup`) are a supplement before a risky bulk operation, not a replacement for the schedule.
- Backups should cover the database (all opportunity content, taxonomies, ACF field data) and uploads (media) at minimum, on a frequency proportionate to publishing volume — daily is a reasonable default while volume is low, per PROJECT.md's guidance to avoid premature complexity.
- Recovery should be tested periodically, not assumed to work because a backup file exists.


---

# 10. Plugin & Theme Update Policy

- The Blogsy parent theme, Royal MCP, ACF, and any SEO plugin should be kept current — Royal MCP's own changelog shows a consistent pattern of security-relevant fixes shipped in patch releases (e.g. the 1.4.26 capability-check hardening), so timely updates are not optional for a plugin actively granting AI write access to the site.
- Before updating Royal MCP specifically, review its changelog for capability or authentication changes that could affect the access policy in Section 5.
- Updates are tested on a local (XAMPP) environment before being applied to production, consistent with README.md's stated tech stack.


---

# 11. Incident Response

If unexpected content changes, unauthorized access, or a compromised credential is suspected:

1. Rotate the Royal MCP API key and reset OAuth state (Section 6) immediately — this cuts off any AI/automation access using the old credentials.
2. Change the Administrator password(s).
3. Review Royal MCP's Activity Log for the relevant time window to identify what tools were called and by which credential.
4. Restore from the most recent known-good backup (Section 9) if content integrity is in question, after confirming the cause has been closed off — restoring before rotating credentials risks the same issue recurring.
5. Record what happened and the resolution, consistent with CHANGELOG.md's principle of preserving project history — a security incident is exactly the kind of decision that document exists to record.


---

# 12. Audit & Logging

- Royal MCP's Activity Log is the primary audit trail for all AI-assisted actions on the site (every `tools/call:*` entry, OAuth handshake events, and admin actions like Reset OAuth State).
- Per ARCHITECTURE.md Section 16, periodic review of the Activity Log is the practical enforcement mechanism behind OPPORTUNITY_MODEL.md Section 14's automation boundary — it exists to catch a tool call that shouldn't have happened, not only to prevent one in advance.
- WordPress's own user activity (logins, content edits made directly in wp-admin) should be covered by a lightweight logging plugin if not already visible through hosting-level logs, so that non-MCP edits are equally auditable.


---

# 13. Third-Party Integrations & External Services

- Any outbound AI provider configuration in Royal MCP's "Outbound AI Provider Configuration" (distinct from the inbound MCP server access covered above) should only be enabled for providers actually in use, with unused provider slots left unconfigured rather than pre-filled with unused API keys.
- Per Royal MCP's own "External Services" disclosure, no opportunity content is sent to a third-party AI service unless a platform is explicitly configured and enabled — this should remain true; outbound provider configuration should not be enabled speculatively.
- Any future integration (payment processors for a premium tier, analytics platforms, form plugins) should be evaluated against PROJECT.md's Decision Framework before adoption, with a note added here on what data, if any, it shares externally.


---

# 14. Vulnerability Disclosure

Until FursaZetu has meaningful production traffic and a dedicated security contact, the practical policy is:

- Security-relevant reports about the FursaZetu site itself (not the Royal MCP plugin, which has its own reporting channel) should go to the project's listed contact method (Contact page, per SITE_STRUCTURE.md) with a clear "security" marker in the subject.
- This section should be revisited once the platform has a dedicated security contact address and, eventually, a formal disclosure policy — appropriate for the Community/Platform phases of ROADMAP.md rather than the current Foundation phase.


---

# 15. Pre-Launch Security Checklist

Before any production launch or major public push:

- [ ] Administrator accounts limited to the minimum necessary people, each with a strong, unique password.
- [ ] Royal MCP "Allow AI to write WordPress options" and "Allow AI to modify theme appearance" toggles confirmed off, unless a specific reviewed workflow requires them.
- [ ] Static API key issued only to trusted automation, not shared as a general-purpose curator credential.
- [ ] OAuth is the default connection method for curator-level AI access.
- [ ] Automated backups configured and one recovery test performed.
- [ ] All plugins and the theme updated to current versions.
- [ ] `wp-config.php` and all secrets confirmed absent from version control.
- [ ] Privacy Policy page reflects actual data collection (contact form, newsletter) rather than placeholder text.
- [ ] Royal MCP Activity Log confirmed to be capturing tool calls as expected.


---

# 16. Ongoing Security Review Cadence

- Review Royal MCP's changelog on every update for capability or authentication-relevant changes (Section 10).
- Periodically sample the Activity Log (Section 12), not only after an incident is suspected.
- Revisit this document whenever a new phase of ROADMAP.md is entered — each phase (Community accounts, Platform submissions) introduces new data and new trust boundaries that this document does not yet cover.


---

# 17. Future Security Considerations

Mirrors the phased approach used throughout the project's documentation:

- **User accounts (Phase 3)**: authentication policy, password storage, session security, and account-data retention need to be added here before launch of that phase.
- **Organization submission workflow (Phase 4)**: needs abuse-prevention policy (rate limiting on submissions, spam/fraud detection) layered on top of the Draft-only rule already established in ARCHITECTURE.md Section 19.
- **API integrations (Phase 4)**: if FursaZetu exposes its own public API, this document will need an API access and rate-limiting policy of its own, separate from Royal MCP's inbound MCP surface.
- **Formal vulnerability disclosure program**: appropriate once the platform has meaningful traffic and a dedicated security contact (Section 14).

None of these should be implemented ahead of the ROADMAP.md phase they belong to, but this document should be amended at the start of each relevant phase, not after a problem surfaces.


---

# 18. Guiding Principle

Security exists to protect the same thing EDITORIAL_POLICY.md protects: the user's trust that what they find on FursaZetu is real, accurate, and safe to act on.

Every policy in this document should make it harder for that trust to be broken by accident, by a compromised credential, or by an AI-assisted workflow operating outside its intended boundary — never harder for a trusted curator to do their job well.
