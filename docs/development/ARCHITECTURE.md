# ARCHITECTURE.md

Version: 1.0

Status: Active

Project: FursaZetu

Website: https://fursazetu.gt.tc


# 1. Purpose

This document defines the technical architecture of the FursaZetu WordPress build.

It exists to translate OPPORTUNITY_MODEL.md into concrete implementation decisions: the Custom Post Type, taxonomies, field storage mechanism, URL structure, file organization, automation, and how Royal MCP tools map onto opportunity workflows.

Where OPPORTUNITY_MODEL.md defines *what* an opportunity is, this document defines *how* WordPress stores, serves, and automates it. Every decision below should trace back to a field, rule, or workflow already defined in OPPORTUNITY_MODEL.md — this document does not introduce new content requirements of its own.

Per PROMPTS.md's documentation-priority order, ARCHITECTURE.md sits directly beneath PROJECT.md, BRAND.md, and above SITE_STRUCTURE.md, DESIGN_SYSTEM.md, and OPPORTUNITY_MODEL.md in terms of *when it's consulted*, but its content is downstream of OPPORTUNITY_MODEL.md specifically. Any conflict between this document and OPPORTUNITY_MODEL.md is resolved in favor of OPPORTUNITY_MODEL.md, and this document should be corrected.


---

# 2. Architecture Philosophy

Consistent with DESIGN_SYSTEM.md's "adaptation over replacement" principle, the technical build follows the same discipline:

- Use WordPress core and the BlogHash/Blogsy theme's native capabilities first.
- Extend via a **child theme** and, where logic doesn't belong in a theme, a small number of purpose-built **must-use or standard plugins** — never by editing theme core files.
- Prefer configuration over custom code, and custom code over a new plugin dependency, per PROJECT.md's AI Development Principles ("avoid unnecessary plugins," "prefer WordPress-native solutions").
- Keep the Opportunity data model itself decoupled from presentation — the CPT and its fields should not assume any particular theme; the Blogsy child theme consumes the model, it does not define it.


---

# 3. Technology Stack Overview

| Layer | Technology | Notes |
|---|---|---|
| CMS | WordPress | Core content engine |
| Theme | BlogHash / Blogsy (child theme) | Per DESIGN_SYSTEM.md — never edit the parent theme directly |
| Custom fields | Advanced Custom Fields (ACF) | See Section 6 for rationale |
| AI / automation interface | Royal MCP plugin | Bridges Claude/other MCP clients to WordPress; see Section 13 |
| Local development | XAMPP | Per README.md |
| Version control | Git / GitHub | Per README.md |
| SEO | Yoast SEO or Rank Math (either, auto-detected) | Royal MCP's `wp_get_seo_meta` / `wp_update_seo_meta` tools already auto-route between the two |

No additional plugins should be added to this stack without checking PROJECT.md's Decision Framework first (does it improve trust, usability, accessibility, discoverability, or alignment with the mission?).


---

# 4. Custom Post Type: `opportunity`

## Registration summary

| Property | Value |
|---|---|
| Post type key | `opportunity` |
| Label | Opportunities |
| Public | true |
| Has archive | true (used as fallback; see Section 8 for the primary category-based URL structure) |
| Supports | title, editor, excerpt, thumbnail, revisions, author |
| Show in REST | true (required for Royal MCP tool compatibility, e.g. `wp_get_posts`, `wp_create_post` with `post_type=opportunity`) |
| Hierarchical | false |

## Mapping to OPPORTUNITY_MODEL.md

| Opportunity Model concept | WordPress implementation |
|---|---|
| Title | Native `post_title` |
| Full Description | Native `post_content` |
| Short Description | Native `post_excerpt` |
| Featured Image | Native featured image (thumbnail) |
| Status (lifecycle) | Combination of native `post_status` (draft/publish) and an ACF Select field `opp_lifecycle_status` for Active / Closing Soon / Expired / Archived — see Section 6 for why lifecycle status is not collapsed entirely into `post_status` |
| Internal Notes | Native post meta, protected (`_` prefix), never exposed publicly |

`post_status = publish` corresponds to the "Published" branch of the OPPORTUNITY_MODEL.md lifecycle (Active / Closing Soon / Expired all remain `publish`); `post_status = draft` corresponds to Draft. Pending Verification is implemented as `post_status = pending`, which is a native WordPress status and requires no custom status registration. Archived opportunities remain `publish` with `opp_lifecycle_status = archived`, rather than moving to `draft`, so they stay reachable by direct URL per EDITORIAL_POLICY.md's Archive Strategy.


---

# 5. Taxonomies

| Taxonomy key | Hierarchical | Maps to |
|---|---|---|
| `opportunity_category` | Yes | Category (Section 9, OPPORTUNITY_MODEL.md) — the 9 fixed top-level terms from SITE_STRUCTURE.md |
| `opportunity_type` | Yes (nested under Category as parent/child terms) | Opportunity Type |
| `organization` | No | Organization |
| `opportunity_location` | Yes (Country → Region, where useful) | Location |
| `opportunity_tag` | No | Tags |

`opportunity_type` terms are stored as children of their corresponding `opportunity_category` term (e.g. "Internships" is a child term of "Careers"). This lets the Publishing Checklist rule "Category and Opportunity Type must be consistent" be partially enforced by taxonomy structure itself, rather than relying entirely on editorial discipline — a curator or an ACF conditional field can restrict the Opportunity Type choices to children of whichever Category is selected.

`opportunity_category` terms are fixed per SITE_STRUCTURE.md and OPPORTUNITY_MODEL.md; new top-level terms are not added without updating both of those documents first.


---

# 6. Field Storage Decision: ACF

Custom fields are implemented as **ACF field groups**, not raw post meta, for two concrete reasons:

1. **Royal MCP has dedicated `acf_*` tools** (`acf_get_field`, `acf_get_fields`, `acf_update_field`, `acf_get_field_groups`) that return and accept type-aware, formatted values — hydrated image arrays, parsed repeater rows, and post-object relationships — instead of the raw serialized values the generic `wp_get_post_meta` / `wp_update_post_meta` tools return. Given how much of FursaZetu's content workflow is expected to run through AI-assisted curation, this materially reduces the chance of a malformed field write.
2. ACF's Return Format settings let the *same* field (e.g. Related Opportunities) come back as either an ID or a fully hydrated object depending on context, without extra template logic.

All field keys use the `opp_` prefix defined in OPPORTUNITY_MODEL.md Section 9, applied as the ACF field name (not just the label), so both `acf_*` tools and the generic post-meta tools resolve to the same underlying key if either path is used.

## Field group: "Opportunity Details"

| ACF Field Name | ACF Field Type | OPPORTUNITY_MODEL.md Field |
|---|---|---|
| `opp_deadline_date` | Date Picker | Deadline Date |
| `opp_deadline_type` | Select (Fixed / Rolling / Ongoing) | Deadline Type |
| `opp_lifecycle_status` | Select (Active / Closing Soon / Expired / Archived) | Status (computed, Section 4) |
| `opp_official_link` | URL | Official Application Link |
| `opp_source_name` | Text | Source (name) |
| `opp_source_url` | URL | Source (url) |
| `opp_verification_status` | Select (Pending / Verified) | Verification Status |
| `opp_eligibility` | Repeater or WYSIWYG | Eligibility Criteria |
| `opp_benefits` | Repeater or WYSIWYG | Benefits |
| `opp_application_process` | WYSIWYG | Application Process |
| `opp_publication_date` | Date Picker | Publication Date |
| `opp_cost_note` | Text | Cost to Applicant |
| `opp_related_opportunities` | Relationship (post_type: `opportunity`) | Related Opportunities |
| `opp_internal_notes` | Textarea | Internal Notes |

Field group location rule: shown only when Post Type = `opportunity`.


---

# 7. Snapshot Query Pattern

OPPORTUNITY_MODEL.md's Section 10 (Opportunity Snapshot) is not a stored object — it is a **query pattern**: any listing context (homepage, category archive, search results, related-opportunities block) should request only the Snapshot fields, not the full ACF field group.

Implementation guidance:

- Snapshot queries should request `title`, featured image, `opp_deadline_date`, `opp_lifecycle_status`, `opportunity_category`/`opportunity_type` terms, `organization` term, and `post_excerpt` only.
- Snapshot queries should never trigger a load of `opp_eligibility`, `opp_benefits`, or `opp_application_process` — those are Full Record fields, loaded only on the single opportunity template.
- This directly implements DESIGN_SYSTEM.md's Performance First principle at the data layer, not just the asset layer.


---

# 8. URL Structure

Per SEO.md's example (`/funding/mastercard-foundation-scholars-program-2027`), the opportunity URL uses the **Category slug**, not the post type slug, as its base — it does not follow WordPress's default `/opportunity/post-name/` pattern.

This is implemented via a custom rewrite rule and a `post_type_link` filter that substitutes the opportunity's assigned `opportunity_category` term slug in place of the post type base:

```
/{opportunity_category-slug}/{post-name}/
```

The native post type archive (`/opportunities/`) is retained as a fallback/technical endpoint (used by feeds, REST discovery, and as a redirect target if an opportunity's category is ever reassigned) but is not the primary user-facing browse URL — Browse Opportunities and category pages (Section 9 of SITE_STRUCTURE.md) are implemented as the taxonomy archive for `opportunity_category`, not the CPT archive.

Because permalinks derive from the Category term, changing an opportunity's Category after publication changes its URL. Per SEO.md ("URLs should not change after publication unless absolutely necessary"), Category reassignment after Published status should be treated as an exceptional edit, not a routine one, and should trigger a redirect if it happens.


---

# 9. Template Hierarchy

Templates live in the Blogsy child theme, following WordPress's standard template hierarchy so the parent theme's supported features (widgets, header/footer, customizer options) continue to work:

| Template file | Purpose |
|---|---|
| `single-opportunity.php` | Full opportunity record — implements the Content Structure from OPPORTUNITY_MODEL.md Section 8 |
| `taxonomy-opportunity_category.php` | Category browse page (SITE_STRUCTURE.md's "Browse Opportunities" filtered view) |
| `taxonomy-opportunity_type.php` | Opportunity Type sub-filter view |
| `taxonomy-organization.php` | Organization archive (precursor to a full Organization profile object per OPPORTUNITY_MODEL.md Section 19) |
| `archive-opportunity.php` | Technical fallback archive (Section 8) |
| `search.php` | Site-wide search, scoped to include `opportunity` post type |
| Template parts: `template-parts/opportunity-card.php` | The Snapshot rendering component — used by homepage, archives, and related-opportunities blocks so the card markup exists in exactly one place |

Template parts should be the enforcement mechanism for consistency called for in DESIGN_SYSTEM.md's Consistency principle — one card template, reused everywhere, rather than similar-but-slightly-different markup per context.


---

# 10. Deadline Engine Implementation

Implements OPPORTUNITY_MODEL.md Section 13 as a WordPress cron job, intentionally decoupled from any AI/MCP session.

## Mechanism

- A daily `wp-cron` event (`fursazetu_deadline_engine`) queries all `opportunity` posts with `post_status = publish`.
- For each, it compares `opp_deadline_date` and `opp_deadline_type` against the current date and applies the status table from OPPORTUNITY_MODEL.md Section 13, writing the result to `opp_lifecycle_status`.
- The 7-day "Closing Soon" window and 90-day "Archived" window are stored as options (`fursazetu_closing_soon_days`, `fursazetu_archive_after_days`) rather than hardcoded, so they can be adjusted in one place.
- The cron job never modifies `post_status`, `opp_verification_status`, or any editorial field — it writes to `opp_lifecycle_status` only, consistent with OPPORTUNITY_MODEL.md Section 14's automation boundary.

## Why not rely on Royal MCP for this

Royal MCP tool calls only run inside an active AI session. The Deadline Engine must run every day regardless of whether anyone is curating that day, so it is implemented as native WP-Cron, not as an MCP-driven workflow. Royal MCP may be used to *inspect* the results (e.g. "list opportunities closing soon") via `wc_get_orders`-style read tools adapted to `wp_get_posts`, but it does not drive the transition itself.


---

# 11. Search & Filtering

- Category, Opportunity Type, Location, and Deadline are the primary filter axes, matching SITE_STRUCTURE.md's stated filtering plan.
- Filtering is implemented as `WP_Query` taxonomy + meta_query combinations against the Snapshot fields (Section 7) — never against the full ACF field group, to keep archive and filter pages fast.
- Full-text search (`wp_search` in Royal MCP terms, or native `s=` queries on the frontend) should be scoped to `post_type=opportunity` plus the small set of static pages, not the entire site index, so opportunity search results aren't diluted by unrelated content.
- Sort options (soonest deadline, most recent, alphabetical) map directly to `opp_deadline_date` and `post_date` — no additional fields are required to support sorting.


---

# 12. SEO Integration

Implements OPPORTUNITY_MODEL.md Section 15 technically:

- **Meta title/description**: written via the active SEO plugin's own meta fields (Yoast or Rank Math), which Royal MCP's `wp_update_seo_meta` tool already auto-detects and routes to — no custom SEO field group is needed in ACF.
- **Slug**: the WordPress-native `post_name`, also reachable via `wp_update_seo_meta`'s `slug` parameter per Royal MCP's own documentation, kept in sync with the Category-based permalink structure in Section 8.
- **Structured data (schema.org)**: emitted by the single-opportunity template based on `opportunity_type`, per the mapping sketched in OPPORTUNITY_MODEL.md Section 15 (`JobPosting`, `EducationalOccupationalProgram`/`Course`, `Event`). This is theme-level output, not a stored field — the underlying values (organization, dates, location) already exist in the ACF field group and taxonomies.
- **Canonical handling for Archived records**: the SEO plugin's default canonical (self-referencing) is left as-is; Archived opportunities are not redirected, per EDITORIAL_POLICY.md's Archive Strategy.


---

# 13. Royal MCP Integration Mapping

This section is the concrete answer to the open question from the initial project review: which Royal MCP tools implement which parts of the opportunity workflow.

| Workflow step (OPPORTUNITY_MODEL.md) | Royal MCP tool(s) |
|---|---|
| Draft a new opportunity from a supplied source | `wp_create_post` (`post_type=opportunity`, `status=draft`) |
| Set Category / Type / Organization / Tags | `wp_add_post_terms` against `opportunity_category`, `opportunity_type`, `organization`, `opportunity_tag` |
| Populate structured fields (deadline, link, source, eligibility, benefits) | `acf_update_field` per field, or `acf_get_fields` first to confirm current state |
| Set featured image from a supplied URL | `wp_upload_media_from_url` + `wp_set_featured_image` |
| Generate/confirm SEO title, description, slug | `wp_update_seo_meta` |
| Move Draft → Pending Verification | `wp_update_post` (`status=pending`) — human-confirmed action only, per OPPORTUNITY_MODEL.md Section 14 |
| Move Pending Verification → Published | `wp_update_post` (`status=publish`) plus `acf_update_field` on `opp_verification_status=Verified` — human-confirmed action only |
| List opportunities closing soon / expired for review | `wp_get_posts` with `post_type=opportunity`, filtered client-side or via a future dedicated meta query, since Royal MCP's generic `wp_get_posts` does not currently support ACF meta filtering directly |
| Check for likely duplicates before creating a new record | `wp_search` scoped to title/organization, prior to `wp_create_post` |
| Read full ACF field group for review | `acf_get_fields` |

Consistent with OPPORTUNITY_MODEL.md Section 14, any tool call in the "human-confirmed action only" rows must not be issued autonomously by an AI agent — it represents the curator's own explicit instruction, not an inferred next step.


---

# 14. Performance Considerations

- Snapshot queries (Section 7) are the default for every listing context; full-record loads are reserved for the single template.
- Featured images should be served through WordPress's native responsive image sizes; no separate image-handling plugin is introduced without a PROJECT.md Decision Framework justification.
- Category and Opportunity Type archives are natural caching candidates (they change only when opportunities are published/updated, not on every page view) — see Section 15.
- The Deadline Engine (Section 10) runs once daily via cron, not on-request, specifically so that status computation never adds latency to a page load.


---

# 15. Caching Strategy

- Standard page caching applies to all public opportunity pages; the MCP endpoint itself is explicitly excluded from page caching by Royal MCP's own `Cache-Control: no-store` headers on its REST routes, so no conflict exists between AI-driven writes and a caching layer.
- Any caching plugin introduced must have its cache invalidated on `save_post` for the `opportunity` post type and on term changes to the taxonomies in Section 5, so a curator's or AI's update is reflected promptly rather than serving a stale Snapshot.
- If Royal MCP's ForgeCache integration (`fc_purge_url`, `fc_clear_cache`) is the caching layer in use, opportunity publish/update workflows should call `fc_purge_url` for the affected opportunity's URL as a matter of course — this is a candidate for a `save_post` hook rather than a manual MCP step.


---

# 16. Security Considerations

Distinct from Royal MCP's own internal plugin security (covered in its own readme/changelog), this section covers FursaZetu-side policy for how that access is used:

- The WordPress API key / OAuth credentials that authorize Royal MCP access are treated as admin-equivalent credentials (per Royal MCP's own documentation, API-key auth runs as an administrator) and should be issued only to whichever AI tooling is actively doing curation work — not shared broadly.
- `allow_option_writes` and `allow_theme_writes` (Royal MCP admin toggles) should remain **off** for FursaZetu's day-to-day content workflow — nothing in the opportunity workflow (Section 13) requires site option or theme-appearance writes, so there is no reason to enable that surface.
- Destructive actions (`wp_delete_post` on an opportunity) are not part of the standard workflow; the model's answer to "this opportunity should no longer be public" is Archived status (Section 4), not deletion, per EDITORIAL_POLICY.md's Content Removal Policy.
- Royal MCP's Activity Log should be treated as the audit trail for AI-assisted curation — periodic review of `tools/call:*` entries is the practical enforcement mechanism behind OPPORTUNITY_MODEL.md Section 14's automation boundary, catching any tool call that shouldn't have happened rather than only preventing it in advance.
- A dedicated SECURITY.md is still an open gap (see PROJECT.md's documentation-priority list); this section is a placeholder for opportunity-workflow-specific security policy until that document exists.


---

# 17. File & Folder Structure

```
wp-content/
  themes/
    blogsy/                  (parent theme — never modified directly)
    blogsy-child/             (child theme — all customization lives here)
      functions.php           (CPT + taxonomy registration, rewrite rules, cron hook)
      single-opportunity.php
      archive-opportunity.php
      taxonomy-opportunity_category.php
      taxonomy-opportunity_type.php
      taxonomy-organization.php
      template-parts/
        opportunity-card.php
      inc/
        cpt-opportunity.php      (post type + taxonomy registration, isolated from functions.php)
        deadline-engine.php      (cron job logic)
        seo-schema.php           (structured data output)
  plugins/
    royal-mcp/                 (as reviewed — third-party, not modified)
    advanced-custom-fields/    (or ACF Pro, per Section 6)
```

Registration and automation logic is split into small, single-purpose files under `inc/` rather than one large `functions.php`, per PROJECT.md's "keep code clean and maintainable" principle. This structure should be finalized alongside DEVELOPMENT_RULES.md once that document exists.


---

# 18. Environment & Deployment

- **Local development**: XAMPP, per README.md. The Opportunity CPT, taxonomies, and ACF field groups should be defined in code (Section 17), not created manually through wp-admin, so the schema is version-controlled and reproducible across environments — ACF field groups should be exported to PHP or JSON and committed to the repository, not left as database-only state.
- **Production**: the live site at fursazetu.gt.tc. Deployment should carry the same child theme and `inc/` files; no environment-specific schema drift is permitted between local and production.
- **Royal MCP configuration** (API keys, OAuth client state) is environment-specific and is never committed to the repository — this is already reflected in `.gitignore`'s exclusion of `wp-config.php` and local-only paths.


---

# 19. Future Technical Considerations

Mirrors OPPORTUNITY_MODEL.md Section 19, translated into technical terms:

- **Multi-language records**: likely WPML or Polylang, per Royal MCP's own documented compatibility (translated posts as separate post objects with the same `opportunity` post type).
- **Application tracking / saved opportunities**: requires user accounts and a new relationship table or user-meta-based bookmark system — out of scope until Phase 3 per ROADMAP.md.
- **Organization profile object**: graduation of the `organization` taxonomy into a dedicated `organization` Custom Post Type once profile pages are needed — the taxonomy-to-CPT migration path should preserve existing term-to-opportunity relationships.
- **AI-powered recommendations**: would consume the existing Category/Type/Tag data plus new interaction-tracking fields not yet modeled; no premature schema should be added for this now.
- **External organization submissions**: would require a front-end submission form (likely Gravity Forms/WPForms or a custom form) that creates opportunities in Draft status only, feeding into the same Publishing Checklist — never a direct-to-Published path.

None of these should be implemented ahead of the ROADMAP.md phase they belong to.


---

# 20. Guiding Principle

Architecture exists to serve the content model, not the other way around.

Every technical decision in this document should make it easier to publish a trustworthy opportunity quickly, and harder to publish an untrustworthy one by accident — consistent with FursaZetu's mission of becoming Africa's most trusted platform for verified opportunities.

Where a technical shortcut would make development faster but would weaken verification, structure, or trust, OPPORTUNITY_MODEL.md and EDITORIAL_POLICY.md take precedence over convenience.
