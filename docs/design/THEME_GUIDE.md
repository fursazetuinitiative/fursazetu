# THEME_GUIDE.md

Version: 1.0

Status: Active — Living Document

Project: FursaZetu

Website: https://fursazetu.gt.tc


# 1. Purpose

This document records every decision about how FursaZetu uses the BlogHash/Blogsy theme: which native features are used as-is, which are deliberately avoided or overridden, what requires custom code instead, the child theme strategy, and the plugin dependencies the theme layer relies on.

It exists so that no customization is ever made — or later questioned — without a documented reason attached to it. A future contributor (human or AI) should be able to read this document and understand *why* something works the way it does, without having to reverse-engineer it from the child theme's code.

This document implements DESIGN_SYSTEM.md's "adaptation over replacement" principle at a decision-by-decision level. DESIGN_SYSTEM.md states the philosophy; this document is the running log of that philosophy applied to a real theme.


---

# 2. Status of This Document

**This is a living document, not a one-time spec.** Unlike OPPORTUNITY_MODEL.md or ARCHITECTURE.md, which define a target state, THEME_GUIDE.md is meant to grow entry by entry as the Blogsy theme is actually audited and worked with.

At the time of writing, the specific feature set of the installed Blogsy theme has not yet been audited file-by-file. Rather than guess at what Blogsy does or doesn't include, this document is structured so that:

- Decisions already established by other documents (child theme strategy, what clearly requires custom code) are recorded now, as final.
- Blogsy-specific feature rows are entered as **Pending Theme Audit** until someone has actually opened the theme's `functions.php`, template files, and customizer options and confirmed what's there.

Section 10 defines the audit process used to convert a "Pending" row into a confirmed one. No row should be marked "In Use" or "Avoided" based on assumption.


---

# 3. Child Theme Strategy

**Decision: a child theme is used, named `blogsy-child`, per ARCHITECTURE.md Section 17.**

Rationale:

- DESIGN_SYSTEM.md's Maintainability principle explicitly requires that customizations remain compatible with future theme updates, and explicitly prohibits modifying theme core files.
- A child theme is the WordPress-native mechanism for exactly this — Blogsy updates can be applied without wiping out FursaZetu-specific templates, styles, or logic.
- All opportunity-specific templates (`single-opportunity.php`, taxonomy archives, the opportunity card template part) live in the child theme, not the parent, per ARCHITECTURE.md Section 9.

## What lives in the child theme

- `functions.php` — enqueues parent + child styles, hooks that don't belong in a standalone plugin.
- Template overrides — only files that genuinely need to differ from the parent's version, following WordPress's template hierarchy so unmodified Blogsy templates continue to serve everything else.
- `inc/` — CPT/taxonomy registration, the Deadline Engine cron job, schema.org output (as established in ARCHITECTURE.md Section 17).
- Child-theme-specific CSS, scoped to additive styling rather than wholesale overrides where possible (Section 6).

## What does not live in the child theme

- Anything that is genuinely site-wide infrastructure rather than presentation (e.g. the Opportunity CPT/taxonomy registration is arguably plugin-territory, not theme-territory) is an open question — see Section 11, Open Items. Keeping CPT registration in a small MU-plugin instead of the child theme's `inc/` folder would decouple the data model from the theme entirely, which may be preferable long-term. This decision is not yet finalized and should be revisited before the CPT is actually registered in code.


---

# 4. Blogsy Features In Use

| Feature | Status | Notes |
|---|---|---|
| Header / navigation | Pending Theme Audit | Assumed used as-is per DESIGN_SYSTEM.md's "Theme First" principle unless a specific conflict is found |
| Footer widget areas | Pending Theme Audit | SITE_STRUCTURE.md's footer content (Navigation, Resources, Connect) should be checked against Blogsy's native footer widget areas before any custom footer template is considered |
| Customizer (site identity, colors, typography) | Pending Theme Audit | Preferred over hard-coded theme-mod overrides wherever Blogsy's customizer already exposes the needed control |
| Post/page templates (non-opportunity content) | In use, unmodified | About, Contact, and other static pages (SITE_STRUCTURE.md) use Blogsy's native page templates — no reason to override these |
| Widgets / sidebar | Pending Theme Audit | To be confirmed against actual homepage/archive layout needs from SITE_STRUCTURE.md |
| Blogsy's native blog/archive templates | Not used for Opportunities | Opportunity archives use custom taxonomy templates (ARCHITECTURE.md Section 9), not Blogsy's default post archive — see Section 5 |

Rows marked "Pending Theme Audit" should be updated to "In use, unmodified," "In use, customized (see Section 6)," or "Not applicable" as the audit (Section 10) proceeds.


---

# 5. Blogsy Features We're Avoiding or Overriding

| Feature | Decision | Rationale |
|---|---|---|
| Default `single.php` / blog post template | Not used for Opportunities | Opportunities are a distinct content type with a fixed internal structure (OPPORTUNITY_MODEL.md Section 8) that a generic blog-post template cannot express — `single-opportunity.php` is a deliberate override, not a missed opportunity to reuse the theme |
| Default archive/category templates, as applied to Opportunities | Not used for Opportunities | Opportunity browsing is taxonomy-driven (`opportunity_category`, `opportunity_type`) with a Snapshot-based card layout (ARCHITECTURE.md Section 7), which needs its own template rather than Blogsy's generic post-excerpt archive loop |
| Any theme-native "related posts" feature | Avoided for Opportunities | Related content on an opportunity page is driven by the `opp_related_opportunities` ACF relationship field and shared Category/Organization/Tags (OPPORTUNITY_MODEL.md Section 11), not by whatever generic similarity logic a theme feature might use — a theme-native related-posts widget would not understand opportunity-specific relationships |
| Theme-native SEO fields (title/meta tags), if present | Avoided | SEO meta is handled by the dedicated SEO plugin (Yoast or Rank Math) per ARCHITECTURE.md Section 12, to stay compatible with Royal MCP's `wp_update_seo_meta` tool, which auto-routes to those plugins specifically |

Any additional avoidance decision discovered during the theme audit should be added to this table with its rationale, not just implemented silently in code.


---

# 6. What Requires Custom Code

The following are already established, by ARCHITECTURE.md, as things Blogsy cannot be expected to provide natively — they require custom code in the child theme or a small companion plugin, not theme configuration:

- Opportunity Custom Post Type and taxonomy registration (ARCHITECTURE.md Section 4–5).
- Category-based permalink rewrite (`/{category}/{opportunity-slug}/`) (ARCHITECTURE.md Section 8).
- Opportunity-specific templates: `single-opportunity.php`, `taxonomy-opportunity_category.php`, `taxonomy-opportunity_type.php`, `taxonomy-organization.php`, and the `opportunity-card.php` template part (ARCHITECTURE.md Section 9).
- The Deadline Engine cron job (ARCHITECTURE.md Section 10).
- Snapshot-scoped queries for listing contexts (ARCHITECTURE.md Section 7).
- Schema.org structured data output per Opportunity Type (ARCHITECTURE.md Section 12).
- Any filtering/search UI beyond what a generic theme search provides, per ARCHITECTURE.md Section 11.

None of the above should be attempted as a Blogsy customizer setting or a theme option — they are content-model logic, not visual configuration, and belong in code regardless of what the audit in Section 10 finds.


---

# 7. Plugin Dependencies

| Plugin | Purpose | Required / Optional | Reference |
|---|---|---|---|
| Advanced Custom Fields (ACF) | Opportunity field storage | Required | ARCHITECTURE.md Section 6 |
| Royal MCP | AI-assisted curation interface | Required for the AI-driven workflow; the site functions without it for manual curation | ARCHITECTURE.md Section 13, SECURITY.md Section 5 |
| Yoast SEO or Rank Math | SEO meta management | Required (either one) | ARCHITECTURE.md Section 12 |
| SiteVault (Royal MCP integration) | Backups | Recommended | SECURITY.md Section 9 |
| ForgeCache (Royal MCP integration) | Page caching | Optional | ARCHITECTURE.md Section 15 |
| WPML or Polylang | Multi-language | Not yet — Phase 3/4 | ARCHITECTURE.md Section 19 |

Every plugin added to this list should have passed PROJECT.md's Decision Framework first. A plugin should not appear in a live site's plugin list without a corresponding row here.


---

# 8. Customizer & Theme Mod Policy

- Theme customization (colors, typography, header/logo settings) should go through Blogsy's native Customizer wherever it already exposes the needed control, per DESIGN_SYSTEM.md's "Theme First" principle.
- Per SECURITY.md Section 5, Royal MCP's "Allow AI to modify theme appearance" toggle remains off by default. If a specific, reviewed workflow ever needs AI-driven theme-mod writes, the target theme mod must also be added to Royal MCP's `royal_mcp_writable_theme_mods` allowlist filter — a theme mod is never made AI-writable by turning on the toggle alone.
- Any theme mod written to by code (rather than a human through the Customizer UI) should be documented here with its purpose, so a future audit doesn't find an unexplained value and assume it's a leftover.


---

# 9. Theme Update Policy

- Blogsy (parent theme) updates should be applied only after confirming, via this document, that no override in the child theme depends on a Blogsy template structure that the update might change (e.g. a template part filename or hook that the child theme relies on).
- Per SECURITY.md Section 10, updates are tested locally (XAMPP) before being applied to production.
- If a Blogsy update changes something this document assumed to be stable, the relevant row in Section 4 or 5 should be corrected immediately, not left stale.


---

# 10. Theme Audit Process

This is the repeatable process for converting a "Pending Theme Audit" row into a confirmed decision, and for adding new rows as they're discovered:

1. Identify the specific Blogsy feature, template, or customizer option in question.
2. Determine whether it already satisfies a need defined in SITE_STRUCTURE.md, DESIGN_SYSTEM.md, or OPPORTUNITY_MODEL.md as-is.
3. If yes → record it in Section 4 as "In use, unmodified," with a one-line note on what it's used for.
4. If it needs light adjustment (styling, minor markup change) → record it in Section 4 as "In use, customized," with a pointer to where the override lives in the child theme.
5. If it conflicts with a requirement (as with the Opportunity-specific cases in Section 5) → record it in Section 5 with the rationale for overriding or avoiding it.
6. If nothing in Blogsy addresses the need at all → record it in Section 6 as requiring custom code.

Every audit entry should answer the question DESIGN_SYSTEM.md poses directly: *"Can the BlogHash/Blogsy theme already achieve this?"* — and record the answer, not just the outcome.


---

# 11. Open Items / Pending Audit

Recorded honestly rather than guessed at:

- Full Blogsy feature inventory (header, footer, widgets, customizer options, post formats, any built-in related-content or SEO features) has not yet been audited against actual theme files.
- Whether Opportunity CPT/taxonomy registration should live in the child theme's `inc/` folder (as ARCHITECTURE.md Section 17 currently shows) or in a small standalone MU-plugin, decoupling the data model from the theme layer entirely, is not yet decided (Section 3).
- Whether Blogsy ships any native structured-data (schema.org) output that could conflict with or duplicate the custom schema output planned in ARCHITECTURE.md Section 12 has not been checked.
- Whether Blogsy's default search template needs overriding to satisfy the `opportunity`-scoped search behavior in ARCHITECTURE.md Section 11, or whether a query-var filter is sufficient without a template override, is not yet determined.

These items should be resolved via the audit process (Section 10) before the corresponding child theme files are written, not discovered mid-implementation.


---

# 12. Future Considerations

- If FursaZetu ever needs a feature Blogsy fundamentally cannot support well (not just "needs custom code," but architecturally incompatible), that would be a PROJECT.md Decision Framework conversation about the theme choice itself — not something this document should quietly work around indefinitely.
- As COMPONENT_LIBRARY.md is populated, its components should be cross-referenced here where they originate from a Blogsy-native pattern versus a fully custom one, so the two documents stay consistent with each other.


---

# 13. Guiding Principle

Every line of custom theme code should exist because Blogsy genuinely couldn't do the job — not because no one checked whether it already could.

This document is what makes that check verifiable after the fact. If a future contributor can't find the entry explaining why something was overridden, that's a gap in this document, not a license to assume the override was necessary.
