# PROMPTS.md

Version: 1.0

Status: Active

Project: FursaZetu

Website: https://fursazetu.gt.tc

---

# Purpose

This document defines the standard prompts, workflows, and interaction patterns used when collaborating with AI assistants during the design, development, maintenance, and growth of the FursaZetu platform.

Rather than serving as a collection of isolated prompts, this document establishes a consistent methodology for using AI as a collaborative partner throughout the project lifecycle.

Its purpose is to improve consistency, reduce ambiguity, preserve architectural integrity, and ensure that every AI-generated output aligns with the principles and standards of FursaZetu.

---

# Philosophy

Artificial Intelligence is a collaborator, not an autonomous decision-maker.

AI should assist by accelerating research, generating ideas, improving documentation, writing code, and solving problems.

Final decisions remain the responsibility of the project team.

Every AI response should strengthen—not replace—human judgment.

---

# AI Responsibilities

AI may assist with:

- Research
- Planning
- Documentation
- UI/UX Design
- WordPress Development
- PHP Development
- CSS
- JavaScript
- SEO
- Accessibility
- Performance Optimization
- Content Drafting
- Opportunity Curation
- Code Reviews
- Testing
- Debugging

AI should always explain significant architectural decisions before implementation.

---

# Project Context

Before responding to any project-related request, AI should understand that:

- FursaZetu is an Opportunity Intelligence Platform.
- WordPress is the Content Management System.
- The Blogsy theme is the design foundation.
- Opportunities are the primary content object.
- Mobile-first design is mandatory.
- Performance is a product feature.
- Accessibility is a core requirement.
- SEO should improve discoverability without compromising usability.
- Existing functionality should be extended before introducing custom development.

---

# Documentation Priority

When generating recommendations or code, AI should follow project documentation in the following order:

1. PROJECT.md
2. BRAND.md
3. ARCHITECTURE.md
4. SITE_STRUCTURE.md
5. DESIGN_SYSTEM.md
6. OPPORTUNITY_MODEL.md
7. EDITORIAL_POLICY.md
8. SEO.md
9. DEVELOPMENT_STANDARDS.md
10. COMPONENT_LIBRARY.md
11. PERFORMANCE.md
12. ACCESSIBILITY.md
13. SECURITY.md

Project documentation always takes precedence over assumptions.

# Prompt Writing Principles

Every prompt should:

- Be specific.
- Provide sufficient context.
- Define the desired outcome.
- Identify constraints.
- Reference relevant project documentation.
- Encourage explanation rather than assumptions.

Well-structured prompts produce better results.

# Standard Prompt Structure

Every substantial request should follow this pattern:

## Context

Describe the current situation.

## Objective

Clearly state the desired outcome.

## Constraints

List limitations or requirements.

## References

Mention relevant project documents.

## Deliverable

Describe the expected output.

# Example Prompt

## Context

We are redesigning the Opportunity Detail page.

## Objective

Improve readability and mobile usability.

## Constraints

- Use Blogsy theme capabilities first.
- Preserve existing functionality.
- Follow DESIGN_SYSTEM.md.
- Follow SEO.md.
- Follow COMPONENT_LIBRARY.md.

## Deliverable

Produce a detailed implementation plan before generating code.

# Development Prompts

Before requesting code generation, AI should be instructed to:

- Explain the problem.
- Identify alternative solutions.
- Recommend the simplest maintainable approach.
- Consider performance.
- Consider accessibility.
- Consider SEO.
- Preserve compatibility with WordPress updates.
- Avoid modifying theme core files.

Implementation should follow explanation.


# Content Prompts

When creating opportunity content:

- Follow OPPORTUNITY_MODEL.md.
- Follow EDITORIAL_POLICY.md.
- Verify information where possible.
- Maintain neutral language.
- Avoid sensationalism.
- Structure content consistently.
- Produce SEO-friendly but human-readable content.


# Design Prompts

When improving the interface:

- Prioritize usability.
- Prioritize mobile devices.
- Minimize visual clutter.
- Maintain visual consistency.
- Respect Blogsy's design language.
- Avoid unnecessary animations.

Design should improve user understanding rather than decoration.


# Problem-Solving Prompts

When debugging:

AI should:

- Explain the issue.
- Identify the root cause.
- Suggest multiple solutions.
- Compare trade-offs.
- Recommend the preferred solution.
- Explain implementation risks.

Avoid guessing.

# Code Generation Prompts

Before generating code, AI should:

- Explain the approach.
- Describe affected files.
- Identify dependencies.
- Highlight potential side effects.
- Produce clean, documented code.
- Follow WordPress Coding Standards.

Generated code should prioritize readability over cleverness.

# Documentation Prompts

When writing documentation:

- Explain the purpose.
- Maintain consistency with existing documentation.
- Avoid duplication.
- Write for both humans and AI.
- Preserve long-term maintainability.

Documentation should teach principles rather than merely describe features.

# Review Prompts

Before approving work, AI should evaluate:
- Correctness
- Maintainability
- Performance
- Accessibility
- Security
- SEO
- User Experience
- Consistency with project standards

Constructive critique is preferred over simple approval.

# Prompt Templates

This section may grow over time with reusable prompts for:

- UI Design
- Component Design
- Feature Planning
- Opportunity Creation
- SEO Audits
- Performance Reviews
- Accessibility Reviews
- Security Reviews
- Documentation
- Testing
- Bug Fixing
- Release Preparation

# Guiding Principle

Every prompt should move the FursaZetu platform closer to its mission of becoming a trusted, accessible, and high-quality destination for curated opportunities.

AI should not simply generate outputs.

It should contribute thoughtfully, consistently, and responsibly to the long-term success of the platform.
