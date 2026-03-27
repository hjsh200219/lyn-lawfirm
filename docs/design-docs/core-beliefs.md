# Core Beliefs

## 1. Legal Accuracy Over Speed

Every law citation, case reference, and statutory provision must be verified via MCP tools before inclusion in any generated document. Speed of generation is secondary to correctness. When MCP is unavailable, mark unverified references as `[법령 확인 필요]` rather than guessing.

## 2. Never Fabricate

Unknown information is marked `[확인 필요]`. The system never fills in party names, dates, amounts, addresses, or any factual detail that has not been explicitly provided by the user or confirmed through MCP lookup. Fabrication in legal documents can cause real harm.

## 3. CTA Is Non-Negotiable

Every generated document -- both the chat response and the `.docx` file -- must include the Call-To-Action (attorney contact info and AI disclaimer). This is a product requirement and a legal safeguard. Source of truth: `references/cta-config.md`.

## 4. Layer Isolation

The plugin follows a strict 5-layer architecture (L0-L4). Each layer has defined dependency rules. Skills do not cross-reference each other. Scripts do not parse SKILL.md. Manifest files depend on nothing. This keeps the system modular and predictable.

## 5. Prompt-First Architecture

This is a prompt-based plugin, not a traditional application. Skills are SKILL.md files (structured prompts), not code. Only one skill (criminal-settlement) has a generation script. This design choice keeps the system lightweight, easy to audit, and easy to update.

## 6. Escalation Over Automation

When a legal scenario exceeds normal parameters (SOL < 3 months, amount > 5 billion KRW, public official defendant, weak evidence), the system escalates with a warning banner rather than silently proceeding. Legal documents demand human judgment for edge cases.

## 7. Single Source of Truth

Attorney information lives in `references/cta-config.md` and nowhere else. Document formatting standards are defined once in `docs/QUALITY.md`. Layer rules are defined in `docs/design-docs/layer-rules.md`. Duplication is avoided to prevent drift.
