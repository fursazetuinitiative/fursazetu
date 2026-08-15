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

**Status as of this update:** the base Blogsy configuration pass (layout, header, menu, footer widgets, permalinks, Hester Core Theme Options) is complete and confirmed hands-on — see Section 11. Color palette remains intentionally deferred, not yet audited. Opportunity-specific templates and CPT/taxonomy structure remain unbuilt, per ARCHITECTURE.md.


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

Confirmed against Blogsy's official WordPress.org listing (v1.0.19, by peregrinethemes) on the date this section was last updated. Rows still marked "Pending Theme Audit" require a hands-on check inside wp-admin, not just the public feature list, since real customizer behavior can differ from marketing copy.

| Feature | Status | Notes |
|---|---|---|
| Header styles | Confirmed available | Blogsy ships multiple header style options natively — exact style choice for FursaZetu is a Section 10 audit task once inside the Customizer |
| Footer widget areas | Confirmed available | Native footer-widgets support confirmed. SITE_STRUCTURE.md's three footer columns (Navigation, Resources, Connect) should map to Blogsy's native footer widget areas rather than a custom footer template |
| Custom Logo / Custom Menu | Confirmed available | Standard WordPress Custom Logo + Custom Menu support, exposed through Blogsy's Customizer |
| Custom Colors | Confirmed available | Native color customization exists. This is also where BRAND.md's still-open color palette decision gets resolved — see Section 10's audit notes |
| Editor Style | Confirmed available | Block editor content styling matches the front end — relevant to how opportunity Full Description content (OPPORTUNITY_MODEL.md Section 8) will actually render |
| Layout options: One column / Left sidebar / Right sidebar / Full width template | Confirmed available | Blogsy supports sidebar-free (one column) and full-width layouts natively — a strong candidate for the opportunity single template and category archives, favoring DESIGN_SYSTEM.md's Simplicity principle over a widget-heavy sidebar |
| Grid layout (native blog archive) | Not used for Opportunities, but relevant as a reference pattern | Blogsy's native grid/masonry blog layout is not used directly for opportunity archives (Section 5), but its visual pattern is a legitimate starting reference for the opportunity card grid in COMPONENT_LIBRARY.md once populated |
| Post Formats, Sticky Post, Threaded Comments | Confirmed available, not used | These are blog/editorial features not relevant to the Opportunity content type; left at theme defaults, unused |
| "Theme Options" panel | Confirmed to exist as a feature tag | Contents not yet audited in wp-admin — see Section 11. Likely surfaces through the Hester Core companion plugin (Section 7) rather than the theme alone |
| Companion plugin dependency (Hester Core) | Confirmed required for full functionality | See Section 7 — several of the features above (extra widgets, some customizer options, one-click demo import) are documented as requiring Hester Core, not the theme in isolation |
| Post/page templates (non-opportunity content) | In use, unmodified | About, Contact, and other static pages (SITE_STRUCTURE.md) use Blogsy's native page templates — no reason to override these |
| Blogsy's native blog/archive templates, applied to Opportunities | Not used for Opportunities | Opportunity archives use custom taxonomy templates (ARCHITECTURE.md Section 9), not Blogsy's default post archive — see Section 5 |

Rows still marked "Pending Theme Audit" elsewhere in this document should be updated the same way: confirmed only after a hands-on check, never from marketing copy alone.


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
| Hester Core (Peregrine Themes) | Official companion plugin for Blogsy — unlocks additional widgets, customization options, Elementor widgets, and one-click demo import | Required for Blogsy's full feature set, confirmed via the theme's own plugin ecosystem | THEME_GUIDE.md Section 4 |
| Advanced Custom Fields (ACF) | Opportunity field storage | Required | ARCHITECTURE.md Section 6 |
| Royal MCP | AI-assisted curation interface | Required for the AI-driven workflow; the site functions without it for manual curation | ARCHITECTURE.md Section 13, SECURITY.md Section 5 |
| Yoast SEO or Rank Math | SEO meta management | Required (either one) | ARCHITECTURE.md Section 12 |
| SiteVault (Royal MCP integration) | Backups | Recommended | SECURITY.md Section 9 |
| ForgeCache (Royal MCP integration) | Page caching | Optional | ARCHITECTURE.md Section 15 |
| WPML or Polylang | Multi-language | Not yet — Phase 3/4 | ARCHITECTURE.md Section 19 |

Note: Blogsy is built by Peregrine Themes, the same studio behind BlogHash — should FursaZetu ever revisit theme choice (Section 12), both are governed by this same plugin-dependency pattern.

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

Recorded honestly rather than guessed at. Resolved as of this update, confirmed hands-on in wp-admin (not from marketing copy):

- **Layout**: one-column/full-width confirmed selected, sidebar removed — matches DESIGN_SYSTEM.md's Simplicity principle, applied site-wide rather than per-template for now.
- **Header style**: a clean, simple header confirmed selected. Specific style name not yet recorded — worth a one-line addition here once convenient, purely for future reference.
- **Menu**: Home / Browse Opportunities / About / Contact confirmed created and assigned, with minor adjustments made from the original suggestion as needed on-site.
- **Footer widgets**: populated and arranged per the recommended Navigation / Resources / Connect structure from SITE_STRUCTURE.md.
- **Permalinks**: confirmed set to "Post name" — the required baseline for ARCHITECTURE.md Section 8's category-based rewrite.
- **Hester Core Theme Options**: opened and reviewed; relevant sections configured. Full settings inventory not itemized here — acceptable, since the practical outcome (a working, reviewed configuration) is what matters, not an exhaustive settings transcript.

Still open:

- **Color palette**: deliberately deferred, on the project's own decision — the current theme default palette is left unchanged intentionally, to be revisited once site structure and content are further along rather than locked in prematurely. This is a considered sequencing choice, not an oversight, and should be revisited explicitly before public launch (see SECURITY.md's Pre-Launch Checklist pattern — a similar pre-launch branding checklist item should exist once COMPONENT_LIBRARY.md is active).
- Whether Opportunity CPT/taxonomy registration should live in the child theme's `inc/` folder or in a small standalone MU-plugin (Section 3) — unrelated to the configuration pass above, still unresolved.
- Whether Blogsy ships any native structured-data (schema.org) output that could conflict with ARCHITECTURE.md Section 12's custom schema output — not yet checked.
- Whether Blogsy's default search template needs overriding for `opportunity`-scoped search (ARCHITECTURE.md Section 11) — not yet determined.

These items should be resolved via the audit process (Section 10) before the corresponding child theme files are written, not discovered mid-implementation.


---

# 12. Future Considerations

- If FursaZetu ever needs a feature Blogsy fundamentally cannot support well (not just "needs custom code," but architecturally incompatible), that would be a PROJECT.md Decision Framework conversation about the theme choice itself — not something this document should quietly work around indefinitely.
- As COMPONENT_LIBRARY.md is populated, its components should be cross-referenced here where they originate from a Blogsy-native pattern versus a fully custom one, so the two documents stay consistent with each other.


---

# 13. Guiding Principle

Every line of custom theme code should exist because Blogsy genuinely couldn't do the job — not because no one checked whether it already could.

This document is what makes that check verifiable after the fact. If a future contributor can't find the entry explaining why something was overridden, that's a gap in this document, not a license to assume the override was necessary.
