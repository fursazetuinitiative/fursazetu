# OPPORTUNITY_MODEL.md

Version: 1.0

Status: Active

Project: FursaZetu

Website: https://fursazetu.gt.tc


# 1. Purpose

This document defines the complete data model for the "Opportunity" — the single most important content object on the FursaZetu platform.

It exists to give every other technical and editorial decision a shared foundation. The WordPress Custom Post Type, ACF field groups, taxonomies, SEO schema, filtering logic, and Royal MCP automation workflows should all be built as direct implementations of what is defined here — not as independent decisions made file by file.

If a proposed feature, field, or automation does not map back to something defined in this document, it should be added here first, deliberately, before it is built.

This document sits above ARCHITECTURE.md, SEO.md, and DEVELOPMENT_STANDARDS.md in the documentation hierarchy defined in PROMPTS.md. Those documents describe *how* the platform is built; this document describes *what* is being built.


---

# 2. Platform Philosophy

FursaZetu is a curated opportunity intelligence platform, not a blog and not an application portal.

Every opportunity published on the platform is a structured record, not a freeform article. Structure is what allows the platform to:

- Filter and search opportunities reliably.
- Detect and archive expired content automatically.
- Present consistent, scannable opportunity pages regardless of who wrote them.
- Let AI tools assist with drafting and formatting without compromising accuracy.
- Support future features (alerts, recommendations, organization profiles) without re-architecting the content model.

Per EDITORIAL_POLICY.md, every opportunity must be traceable to a credible, verifiable source. The data model reflects this by treating source and verification as first-class fields, not afterthoughts.


---

# 3. Opportunity Lifecycle

Every opportunity record moves through a defined lifecycle. Status is a system-managed field, not a manual label the curator invents freely.

## Draft

Created by a curator, or drafted with AI assistance per EDITORIAL_POLICY.md's AI-Assisted Curation clause. Not visible to site visitors. Missing required fields are expected at this stage.

## Pending Verification

All required fields are present, but the source has not yet been independently confirmed against an official channel (see EDITORIAL_POLICY.md, Verification).

## Published — Active

Verified, live, and within its application window. This is the default public-facing state.

## Published — Closing Soon

Automatically applied when the deadline falls inside a defined warning window (see Section 13, Deadline Engine). No editorial action required; this is a computed state, not a curator decision.

## Expired

The deadline has passed. The opportunity is no longer eligible for new applications. Per EDITORIAL_POLICY.md's Archive Strategy, expired opportunities are not deleted — they are relabeled and preserved for reference and SEO value.

## Archived

A long-term expired opportunity, or one manually archived by a curator (duplicate, source retracted, fraud discovered, etc.). Archived opportunities remain on the platform but are clearly marked as closed and excluded from active browse/filter results by default.

## Reopened (exception state)

Some opportunities (recurring fellowships, rolling admissions) reopen on a cycle. A reopened opportunity either becomes a new record referencing the prior one (preferred, protects historical accuracy) or has its deadline and status reset by a curator — never by automation.

State transitions:

Draft → Pending Verification → Published (Active) → Published (Closing Soon) → Expired → Archived

Only a human curator can move a record out of Draft into Pending Verification or Published. Automation (see Section 14) is only permitted to move a record forward along the Active → Closing Soon → Expired → Archived path.


---

# 4. Opportunity Structure

Every opportunity record is composed of eight field groups:

1. **Core Identity** — what the opportunity is called and what it is.
2. **Classification** — where it belongs in the site's taxonomy.
3. **Timing** — when it opens, closes, and starts.
4. **Eligibility & Requirements** — who it is for.
5. **Benefits** — what the applicant receives.
6. **Application** — how to apply and where.
7. **Source & Verification** — where the information came from and how trustworthy it is.
8. **SEO & Metadata** — how the opportunity is discovered and displayed.

Sections 5 and 6 define exactly which fields belong to each group and whether they are mandatory.


---

# 5. Required Fields

An opportunity cannot leave Draft status without every field below. This list is the enforceable minimum — it corresponds directly to the Opportunity Acceptance Policy in EDITORIAL_POLICY.md.

| Field | Type | Notes |
|---|---|---|
| Title | Text | Opportunity name, organization implied or included per Writing Standards (Section 7) |
| Category | Taxonomy (single) | One of the 9 categories defined in SITE_STRUCTURE.md |
| Opportunity Type | Taxonomy (single, nested under Category) | e.g. "Internship" under Careers — see Section 9 |
| Organization | Taxonomy or relationship | The provider of the opportunity |
| Deadline Date | Date | Drives the Deadline Engine (Section 13); required unless Rolling/Ongoing is explicitly selected |
| Deadline Type | Select | Fixed Date / Rolling / Ongoing — see Section 13 |
| Official Application Link | URL | Must resolve to the provider's own domain or a recognized official application portal |
| Source | Text + URL | Name of the source and the URL it was verified against |
| Verification Status | Select | Verified / Pending Verification — internal field, not necessarily shown publicly |
| Short Description | Text (1–2 sentences) | Used in listing cards, search results, and meta description fallback |

No opportunity is published without all ten of these fields populated and internally consistent (e.g. a Deadline Date in the past cannot accompany a Published — Active status).


---

# 6. Recommended Fields

These fields are not required to publish but are strongly encouraged, and several are required in practice for specific Opportunity Types (e.g. a Scholarship should always carry Eligibility and Benefits).

| Field | Type | Notes |
|---|---|---|
| Full Description | Rich text | The main body content — see Section 8 |
| Eligibility Criteria | Rich text or repeater | Who can apply |
| Benefits | Rich text or repeater | What is provided (funding amount, stipend, mentorship, etc.) |
| Location | Text or taxonomy | Country/region/city, or "Remote" / "Global" |
| Application Process | Rich text or steps | How to apply, beyond the link itself |
| Publication Date | Date | When FursaZetu published the listing (distinct from Deadline) |
| Cost to Applicant | Text | Explicitly state "Free to apply" where true — protects trust |
| Tags | Taxonomy (multiple) | Free-form discovery terms (e.g. "AI", "climate", "undergraduate") |
| Featured Image | Media | Organization logo or opportunity banner |
| Related Opportunities | Relationship | See Section 11 |
| Internal Notes | Text (private) | Curator-only notes, never shown publicly |


---

# 7. Writing Standards

Writing standards inherit directly from EDITORIAL_POLICY.md and BRAND.md. In the context of the data model specifically:

- **Title field** should read as a real name, not a keyword-stuffed string: "Mastercard Foundation Scholars Program 2027," not "Scholarship Kenya Africa Funding Mastercard 2027."
- **Short Description** must summarize what the opportunity is, who it is for, and why it matters — never promotional language ("Amazing opportunity!" is not acceptable; see EDITORIAL_POLICY.md, Neutrality).
- **Full Description** should be simplified for a general audience without altering the meaning of the original announcement, per EDITORIAL_POLICY.md's Accessibility principle.
- Every claim of fact in the record (deadline, amount, eligibility) must be traceable to the Source field. Nothing should be invented or estimated to fill a gap.


---

# 8. Content Structure

The Full Description body should follow a consistent internal structure so that opportunity pages remain scannable and predictable across the whole site, regardless of category:

1. **Overview** — one short paragraph: what it is, who runs it, why it exists.
2. **Eligibility** — who qualifies.
3. **Benefits** — what is provided.
4. **Application Process** — how to apply, what's required (documents, essays, interviews).
5. **Deadline** — restated in plain language even though it also exists as a structured field.
6. **Official Source** — a clear, visible link out, matching EDITORIAL_POLICY.md's Transparency principle.

This structure is what COMPONENT_LIBRARY.md's opportunity-detail template should be built against once that document is populated.


---

# 9. Metadata Standards

## Taxonomy: Category (single-select, required)

Fixed to the nine categories defined in SITE_STRUCTURE.md. This taxonomy should not be extended casually — new top-level categories are a SITE_STRUCTURE.md decision, not a per-opportunity one.

`careers` · `education` · `funding` · `learning` · `leadership-development` · `events` · `competitions-challenges` · `volunteering` · `research`

## Taxonomy: Opportunity Type (single-select, required, scoped under Category)

Maps to the sub-types already enumerated per category in SITE_STRUCTURE.md — for example, under Careers: Jobs, Graduate Programmes, Internships, Attachments, Apprenticeships, Industrial Training, Traineeships. Each category has its own type list; a curator should never be able to select a type that belongs to a different category.

## Taxonomy: Organization

Represents the opportunity provider. May start as a flat taxonomy and evolve into a full profile object in Phase 3 (Community) of ROADMAP.md ("Organization profiles"). Kept as a taxonomy now, not a Custom Post Type, to avoid premature complexity per PROJECT.md's Decision Framework.

## Taxonomy: Location

Country/region-based. Supports future country-specific browsing mentioned in SITE_STRUCTURE.md's filtering plans.

## Taxonomy: Tags (multi-select, optional)

Free-form. Used for cross-cutting discovery, not for primary classification.

## Field naming convention

Custom fields should use a consistent `opp_` prefix (e.g. `opp_deadline_date`, `opp_official_link`, `opp_verification_status`) regardless of whether they are implemented as ACF fields or native post meta, so that Royal MCP tool calls (`wp_get_post_meta`, `wp_update_post_meta`, or the `acf_*` tools) operate against predictable, self-documenting keys. Final field-storage mechanism (ACF vs. native meta) is an ARCHITECTURE.md decision; this document defines the field names and types that decision must satisfy.


---

# 10. Opportunity Snapshot

The Opportunity Snapshot is the compact, denormalized summary used everywhere a full record isn't needed: homepage featured/latest sections, category browse grids, search results, and related-opportunity blocks.

| Snapshot Field | Source |
|---|---|
| Title | Core Identity |
| Organization | Classification |
| Category + Type | Classification |
| Deadline (formatted) | Timing |
| Status badge (Active / Closing Soon / Expired) | Computed — Deadline Engine |
| Location | Recommended field |
| Short Description | Core Identity |
| Featured Image | Recommended field |

The Snapshot should never require loading the Full Description or Eligibility/Benefits fields — it exists specifically so listing pages stay fast, per DESIGN_SYSTEM.md's Performance First principle.


---

# 11. Data Relationships

- **Opportunity → Category** — many-to-one (one category per opportunity).
- **Opportunity → Opportunity Type** — many-to-one, constrained to the selected Category.
- **Opportunity → Organization** — many-to-one (one provider per opportunity; the same organization can have many opportunities).
- **Opportunity → Tags** — many-to-many.
- **Opportunity → Related Opportunities** — many-to-many, curator-assigned or auto-suggested by shared Category/Organization/Tags. Powers SEO.md's Internal Linking Strategy.
- **Opportunity → Source** — one-to-one per record; a source is recorded per opportunity rather than as a shared taxonomy, since the same organization may publish opportunities through different official channels over time.


---

# 12. Validation Rules

An opportunity record fails validation — and cannot move to Published — if any of the following are true:

- Any Required Field (Section 5) is empty.
- Deadline Date is in the past, unless Deadline Type is explicitly Rolling or Ongoing.
- Official Application Link does not resolve, is not HTTPS, or does not point to a plausible official domain (not a generic aggregator, not a shortened link without a disclosed destination).
- Source is missing a name, a URL, or both.
- Verification Status is not "Verified."
- Category and Opportunity Type do not match (a type from one category selected under a different category).
- Title duplicates an existing Active opportunity from the same Organization with the same Deadline Date (duplicate-detection check, prevents accidental double-publishing).

Validation is a precondition for the Draft → Pending Verification → Published transition. It is not itself a substitute for the human verification step described in EDITORIAL_POLICY.md — a record can pass every validation rule and still be factually wrong; only a curator's check against the official source satisfies Verification Status.


---

# 13. Deadline Engine

The Deadline Engine is the automated process that keeps opportunity status synchronized with real-world deadlines, so curators are not manually re-labeling hundreds of records.

## Inputs

- `opp_deadline_date`
- `opp_deadline_type` (Fixed / Rolling / Ongoing)
- Current server date

## Behavior

| Condition | Resulting Status |
|---|---|
| Deadline Type = Fixed, deadline more than 7 days out | Published — Active |
| Deadline Type = Fixed, deadline within 7 days | Published — Closing Soon |
| Deadline Type = Fixed, deadline has passed | Expired |
| Deadline Type = Rolling or Ongoing | Published — Active indefinitely (excluded from automatic expiry) |
| Expired for longer than 90 days | Archived |

The 7-day and 90-day windows are configurable defaults, not fixed constants — they should live in a single settings location in the eventual implementation, not be hardcoded per-template.

## Implementation note (for ARCHITECTURE.md / Royal MCP workflow)

This is a natural candidate for a scheduled WordPress cron task that queries opportunities by status and deadline field, then updates status via `wp_update_post` (or `wp_update_post_meta` if status is meta-driven rather than post-status-driven). It should run independently of any AI/MCP session — deadline transitions must happen reliably even when no one is actively curating that day.


---

# 14. Automation Rules

This section defines what AI assistance (including Royal MCP-driven workflows) is permitted to do to an opportunity record, and what always requires a human curator — extending EDITORIAL_POLICY.md's AI-Assisted Curation clause into concrete field-level permissions.

## AI/MCP may do without prior human review

- Draft Full Description, Short Description, Eligibility, and Benefits text from a supplied official source, left in Draft status.
- Suggest Category, Opportunity Type, and Tags for curator confirmation.
- Generate SEO title/description candidates (Section 15) for curator confirmation.
- Perform automated Deadline Engine status transitions (Section 13) — these are mechanical, not editorial, judgments.
- Flag likely-duplicate records for curator review.

## AI/MCP must never do without human confirmation

- Set Verification Status to "Verified."
- Move a record from Draft or Pending Verification into Published.
- Invent, estimate, or infer factual details (deadlines, amounts, eligibility) not present in the supplied source.
- Modify the Official Application Link or Source fields on an already-Published record.
- Permanently delete an opportunity (archiving is acceptable per the lifecycle; deletion is not the default path — see EDITORIAL_POLICY.md's Content Removal Policy).

Any Royal MCP tool call that would cross from the first list into the second must stop and return control to the curator rather than proceeding.


---

# 15. SEO Standards

This section operationalizes SEO.md specifically for the opportunity object.

- **URL structure:** `/{category-slug}/{opportunity-slug}` per SEO.md's example (`/funding/mastercard-foundation-scholars-program-2027`). Opportunity Type is not part of the URL; Category is.
- **Meta Title formula:** `{Opportunity Title} | FursaZetu`
- **Meta Description:** derived from Short Description if not explicitly overridden.
- **Structured data (schema.org):** type selection depends on Opportunity Type — e.g. `JobPosting` for Careers-type opportunities with a hiring organization, `EducationalOccupationalProgram` or `Course` for Learning/Education types, `Event` for Events and Competitions. This mapping should be finalized in ARCHITECTURE.md but the underlying field values (organization, dates, location) are already captured in this model and require no additional fields to support it.
- **Canonical URL:** one per opportunity; Archived records keep their original canonical rather than redirecting, per EDITORIAL_POLICY.md's Archive Strategy.
- **Indexing:** Draft and Pending Verification records are never indexed. Archived records generally remain indexed unless the underlying opportunity was found to be fraudulent (see EDITORIAL_POLICY.md, Content Removal Policy), in which case they are deindexed and removed.


---

# 16. Internal Linking

Every opportunity page should link outward to:

- Other Active opportunities in the same Category (hub-style navigation).
- Other opportunities from the same Organization.
- Manually or automatically suggested Related Opportunities (Section 11).

This satisfies SEO.md's Internal Linking Strategy and also serves the platform's core UX goal from SITE_STRUCTURE.md — helping a user who finds one opportunity discover adjacent ones without a fresh search.


---

# 17. Publishing Checklist

Before a curator moves a record from Pending Verification to Published:

- [ ] All Required Fields (Section 5) are complete.
- [ ] Source has been checked directly, not copied from a reposting site (EDITORIAL_POLICY.md, Verification).
- [ ] Deadline Date and Deadline Type are correct and internally consistent.
- [ ] Official Application Link works and points to the real provider.
- [ ] Title and Short Description contain no promotional language.
- [ ] Category and Opportunity Type are both correct and consistent with each other.
- [ ] SEO Title and Meta Description are set or confirmed.
- [ ] No duplicate record already exists for this Organization + Deadline combination.


---

# 18. Quality Assurance Checklist

Applied on a sampling basis to already-published opportunities, and always when a correction is reported (see EDITORIAL_POLICY.md, Corrections Policy):

- [ ] Deadline Engine status still matches the actual deadline.
- [ ] Official Application Link still resolves and still points to the correct opportunity (providers sometimes reuse or redirect old URLs).
- [ ] Description still matches the current state of the official announcement.
- [ ] No factual drift has occurred since original publication.


---

# 19. Future Expansion

The following are intentionally out of scope for the current model but anticipated by ROADMAP.md and should extend this document, not bypass it, when they are built:

- **Multi-language records** (translated Title/Description fields per opportunity) — Phase 3/4.
- **Application tracking** (user-submitted "I applied" status) — Phase 3, requires user accounts.
- **Saved/bookmarked opportunities** — Phase 3, requires user accounts.
- **Organization profile object** — likely graduation of the Organization taxonomy into a full Custom Post Type once organizations need their own dedicated pages (Phase 3).
- **AI-powered recommendations** — Phase 4; depends on the Tags, Category, and user-interaction data already defined here, plus new user-behavior fields not yet modeled.
- **Opportunity submission by external organizations** — Phase 4; will require a new Verification sub-workflow distinct from curator-sourced records, since the source and the submitter would be the same party.

None of these should be implemented ahead of the phase in which ROADMAP.md places them.


---

# 20. Guiding Principle

An opportunity is only as trustworthy as the structure behind it.

Every field in this model exists to answer a question a user, a search engine, or a curator will eventually ask: Is this real? Am I eligible? When is it due? Where do I apply? Where did this come from?

If a proposed field or automation doesn't help answer one of those questions, it does not belong in the model — consistent with PROJECT.md's Guiding Principle that FursaZetu should never add features simply because they are possible.
