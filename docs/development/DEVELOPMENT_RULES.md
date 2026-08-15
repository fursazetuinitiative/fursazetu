# DEVELOPMENT_RULES.md

Version: 1.1

Status: Active

Project: FursaZetu

Website: https://fursazetu.gt.tc

# Current Development Phase

The FursaZetu project has completed its planning and architecture phase.

The current phase is IMPLEMENTATION.

All core documentation (Brand, Design System, Site Structure, Opportunity Model, Editorial Policy, Architecture, Theme Guide) should be treated as the project's source of truth.

Do not create new documentation unless:

- an existing document requires correction;
- a new feature introduces a completely new concept;
- documentation is explicitly requested.

Implementation should always take priority over creating additional markdown files.

# Development Priorities

When working on this project, always prioritize:

1. Building and improving the WordPress website.
2. Using existing Blogsy and Hester Core capabilities.
3. Maximizing Gutenberg blocks and patterns.
4. Improving user experience.
5. Accessibility.
6. Performance.
7. Responsive design.

Only recommend custom code when native WordPress capabilities cannot achieve the required result.


# Decision Making

When multiple approaches exist:

Prefer

Native WordPress
↓

Blogsy features
↓

Hester Core
↓

Gutenberg patterns
↓

Royal MCP tools
↓

Custom PHP/CSS/JS

Always choose the simplest maintainable solution.

# Royal MCP Usage

Royal MCP should be used to inspect and modify the WordPress website whenever possible.

Before concluding that a task cannot be completed:

1. Inspect available MCP tools.
2. Inspect the active theme.
3. Inspect installed plugins.
4. Inspect reusable block patterns.
5. Inspect WordPress settings.

Only declare a limitation after confirming that no native WordPress, Blogsy, Hester Core, Gutenberg, or Royal MCP capability can accomplish the task


# 1. Purpose

This document defines the coding standards, naming conventions, file organization, and workflow rules for all custom development on FursaZetu.

It formalizes the file/folder structure already sketched in ARCHITECTURE.md Section 17 and the child-theme conventions established in THEME_GUIDE.md, turning both into enforceable rules rather than one-off examples. Every custom file written for FursaZetu — theme, plugin, or configuration — should be able to be checked against this document.

This document is referenced directly by PROMPTS.md's documentation-priority list (as "DEVELOPMENT_STANDARDS.md") and governs the Development Prompts and Code Generation Prompts sections of that document.


---

# 2. Relationship to Other Documents

This document does not introduce new architectural decisions. It formalizes decisions already made elsewhere:

- **What** gets built and its data shape: OPPORTUNITY_MODEL.md.
- **Where** it lives structurally (CPT, taxonomies, templates, file layout): ARCHITECTURE.md.
- **Which theme features are used vs. overridden**: THEME_GUIDE.md.
- **How it's built, named, and reviewed**: this document.

If a rule here appears to conflict with ARCHITECTURE.md or THEME_GUIDE.md, those documents take precedence and this document should be corrected.


---

# 3. Coding Philosophy

- **Reuse before building.** Check whether WordPress core, the Blogsy parent theme (per THEME_GUIDE.md's audit process), or an already-installed plugin (ACF, Royal MCP) already does the job before writing new code.
- **Readability over cleverness**, per PROJECT.md's AI Development Principles. A junior contributor or a future AI session should be able to read a file and understand its purpose without needing the original author present.
- **Small, single-purpose files** over one large file doing many things — already modeled in ARCHITECTURE.md Section 17's `inc/` breakdown (`cpt-opportunity.php`, `deadline-engine.php`, `seo-schema.php` as separate files, not one combined file).
- **WordPress Coding Standards (WPCS)** govern all PHP, consistent with Royal MCP's own codebase style (which this project has direct visibility into) and with PROJECT.md's explicit instruction to "follow WordPress Coding Standards."


---

# 4. PHP Standards

- Follow WPCS formatting: tab indentation, Yoda conditions, space-before-parenthesis on control structures, brace-on-same-line for functions — matching the style already visible throughout the Royal MCP plugin's codebase, which serves as a working reference example available in this project.
- Every user-supplied or external value is sanitized on input and escaped on output — `sanitize_text_field()`, `esc_html()`, `esc_url()`, `esc_attr()` as appropriate, never raw `echo` of a dynamic value. This is a security rule as much as a style rule; see Section 9.
- Every database interaction goes through WordPress APIs (`WP_Query`, `get_posts()`, `$wpdb->prepare()` when raw SQL is genuinely necessary) — no unprepared, hand-built SQL.
- Capability checks (`current_user_can()`) guard any code path that writes data, mirroring the per-tool capability-check pattern already established in Royal MCP's own code and required by SECURITY.md Section 4's role model.
- No inline styles or inline scripts where `wp_enqueue_style()` / `wp_enqueue_script()` can be used instead.


---

# 5. Naming Conventions

## Function and hook prefix

All custom functions, hooks, and filters use the `fz_` prefix (short for FursaZetu), distinct from Royal MCP's own `royal_mcp_` prefix to avoid any collision between the two codebases operating on the same site.

Example: `fz_register_opportunity_post_type()`, `fz_deadline_engine_run()`, `fz_get_opportunity_snapshot()`.

## Custom Post Type & taxonomies

Fixed by ARCHITECTURE.md and not to be renamed casually:

- Post type: `opportunity`
- Taxonomies: `opportunity_category`, `opportunity_type`, `organization`, `opportunity_location`, `opportunity_tag`

## ACF field names

Fixed by OPPORTUNITY_MODEL.md Section 9 and ARCHITECTURE.md Section 6: the `opp_` prefix (e.g. `opp_deadline_date`, `opp_official_link`). No custom field should be added without this prefix, and no field should be added to the Opportunity field group that isn't first defined in OPPORTUNITY_MODEL.md.

## CSS

- Class naming follows a BEM-style convention (`opportunity-card`, `opportunity-card__title`, `opportunity-card--closing-soon`) so component boundaries stay legible in markup, consistent with DESIGN_SYSTEM.md's Consistency principle.
- Component-specific styles are scoped to their component file/selector, not layered as global overrides, so a Blogsy theme update is less likely to produce an unexpected visual conflict (THEME_GUIDE.md Section 9).

## JavaScript

- Minimal by default, per DESIGN_SYSTEM.md's Performance First principle — JavaScript is added only where it delivers real interaction value (e.g. filter UI), not by default on static content.
- Any custom script is enqueued with a `fz-` handle prefix and declared dependencies (no implicit jQuery reliance unless the dependency is declared).


---

# 6. File & Folder Organization

Formalizes ARCHITECTURE.md Section 17 as a binding structure:

```
wp-content/
  themes/
    blogsy/                     (parent — never modified)
    blogsy-child/
      functions.php             (enqueues, hook wiring only — no business logic)
      single-opportunity.php
      archive-opportunity.php
      taxonomy-opportunity_category.php
      taxonomy-opportunity_type.php
      taxonomy-organization.php
      template-parts/
        opportunity-card.php
      inc/
        cpt-opportunity.php
        deadline-engine.php
        seo-schema.php
      assets/
        css/
        js/
```

Rules:

- `functions.php` wires things together (enqueues, `require_once` for `inc/` files) — it does not contain the actual logic for CPT registration, cron handling, or schema output. Those live in their own `inc/` files.
- Every file in `inc/` has a single, named responsibility matching its filename. A file named `deadline-engine.php` does not also register a taxonomy.
- The open question from THEME_GUIDE.md Section 11 (whether CPT/taxonomy registration should instead live in a standalone MU-plugin rather than the child theme) must be resolved before `cpt-opportunity.php` is written — this document does not resolve it, it only fixes the structure for whichever answer is chosen.


---

# 7. Version Control & Git Workflow

- **Branching:** `main` is always deployable. Feature work happens on descriptively named branches (`feature/opportunity-cpt`, `fix/deadline-engine-timezone`), merged via pull request rather than committed directly to `main`.
- **Commit messages:** short imperative summary line, optionally followed by a blank line and rationale — matching the "what changed and why" discipline CHANGELOG.md already requires at the project level. A commit that changes behavior worth noting to a user or curator should have a corresponding CHANGELOG.md entry.
- **No secrets in commits:** `wp-config.php`, API keys, and OAuth credentials are never committed, per SECURITY.md Section 6 and the existing `.gitignore` rules — this is a hard rule, not a style preference, and a commit found to contain a secret should be treated as a credential-rotation event under SECURITY.md Section 11.
- **ACF field groups are exported and committed** (PHP or JSON export), per ARCHITECTURE.md Section 18, so the schema is reproducible across environments rather than living only in one database.


---

# 8. Documentation in Code

- Every function has a WordPress-standard docblock: purpose, `@param`, `@return`.
- Comments explain *why*, not *what* — the code already says what it does; a comment earns its place by explaining a non-obvious decision (a pattern already modeled throughout Royal MCP's codebase, e.g. its inline notes on why certain security checks are ordered the way they are).
- Any deliberate deviation from a rule in this document (an exception that had to be made) is commented at the point of deviation with a one-line reason, so it reads as a documented decision rather than an oversight.


---

# 9. Security Practices in Code

Extends SECURITY.md into code-level rules:

- Sanitize on input, escape on output, always — no exceptions for "trusted" internal data, since a field populated via Royal MCP today could be populated via a public submission form tomorrow (ARCHITECTURE.md Section 19).
- Any code path that writes to `opp_verification_status` or moves an opportunity to Published is checked against the current user's actual capability, not assumed safe because it's only called from an admin screen.
- No feature is built that requires disabling a WordPress or Royal MCP security default (e.g. widening the `royal_mcp_writable_options` allowlist) without that decision being recorded in SECURITY.md first, not discovered as a side effect of a feature branch.


---

# 10. Performance Practices in Code

Extends ARCHITECTURE.md Section 14 into code-level rules:

- Listing/archive contexts query only Snapshot fields (ARCHITECTURE.md Section 7) — a `WP_Query` for a card grid should never request full post content or the full ACF field group.
- Expensive operations (the Deadline Engine's full-catalog scan) run on cron, never on a front-end page load.
- Database queries are counted during development (via `Query Monitor` or equivalent) before a template is considered done — a template that N+1-queries ACF fields per card in a loop is not acceptable.


---

# 11. Testing Before Merge

Before a branch is merged to `main`:

- [ ] Tested locally on XAMPP, not only "looks right" in the editor.
- [ ] Checked at mobile width first, then desktop, per DESIGN_SYSTEM.md's Mobile-First principle.
- [ ] Checked against WCAG-level basics: keyboard navigation, heading hierarchy, color contrast, per DESIGN_SYSTEM.md's Accessibility principle.
- [ ] No PHP notices/warnings in the debug log (`WP_DEBUG` enabled locally).
- [ ] Any new ACF field or taxonomy term matches OPPORTUNITY_MODEL.md exactly — no ad hoc fields added outside that document.
- [ ] Any new Royal MCP-facing behavior (a field an AI workflow would touch) is checked against the Automation Rules in OPPORTUNITY_MODEL.md Section 14.


---

# 12. Working With AI-Generated Code

Since a meaningful share of this project's code is expected to be AI-assisted (per PROMPTS.md), the following rules apply specifically to that workflow:

- AI-generated code is reviewed with the same rigor as human-written code against this entire document — it does not get a pass on WPCS, security, or naming conventions because of how it was produced.
- Per PROMPTS.md's Development Prompts guidance, an explanation of the approach and affected files is expected *before* code is generated, not reconstructed afterward to justify what was already written.
- An AI session should not introduce a new naming convention, file location, or field name that isn't already established in this document or OPPORTUNITY_MODEL.md — if a genuinely new pattern is needed, it should be proposed as a documentation change first, per PROMPTS.md's Documentation Priority order, not introduced silently through a single code change.


---

# 13. Deployment Checklist

- [ ] All Section 11 testing steps pass locally.
- [ ] SECURITY.md's Pre-Launch Security Checklist reviewed if this is a launch-relevant change.
- [ ] CHANGELOG.md updated for any user- or curator-facing change.
- [ ] No debug output, `var_dump()`, or temporary logging left in code.
- [ ] Deployed to production only after the equivalent local (XAMPP) test, per THEME_GUIDE.md Section 9 and SECURITY.md Section 10.


---

# 14. Future Considerations

- As the codebase grows, this document should gain a formal automated-linting setup (PHPCS with the WPCS ruleset run in CI) rather than relying on manual review alone — appropriate once there is more than one regular contributor.
- If the project ever adopts a build step for assets (bundling/minification), this document should be updated with the relevant tooling conventions rather than left silent on it.


---

# 15. Guiding Principle

Consistency is what lets this project be maintained by more than one mind — human or AI — without every change requiring an archaeology project first.

A rule in this document exists to make the next contributor's job easier, not to slow the current one down. Where a rule genuinely gets in the way of a better solution, the right response is to propose changing the rule here, not to quietly ignore it.
