# Layer Dependency Rules

## Layer Definitions

```
L0  Plugin Manifest    .claude-plugin/, .mcp.json
L1  Shared References  references/
L2  Skills             skills/*/SKILL.md
L3  Scripts            skills/*/scripts/
L4  Documentation      docs/, README.md
```

## Dependency Rules

```
L0 -> (nothing)          Manifest depends on nothing
L1 -> (nothing)          References are self-contained
L2 -> L1                 Skills MAY reference shared resources (../../references/)
L2 -> L0 (MCP only)      Skills use MCP tools defined in .mcp.json
L3 -> L1                 Scripts MAY import CTA config from references
L3 -> (npm: docx)        Scripts depend on npm docx package
L4 -> L0, L1, L2, L3     Documentation may reference any layer
```

## Prohibited Dependencies

- L0 must NOT depend on L2 or L3 (manifest must not reference skill internals)
- L1 must NOT depend on L2 (references must not import from skills)
- L2 must NOT depend on other L2 (skills must not cross-reference each other's SKILL.md at runtime; they may mention other skills in "참고" sections for user guidance only)
- L3 must NOT depend on L2 (scripts must not parse SKILL.md)

## File Ownership

| Layer | Owner | Change Frequency |
|-------|-------|-----------------|
| L0 | Plugin admin | Rare (version bumps, marketplace updates) |
| L1 | Content admin | Low (attorney info changes) |
| L2 | Legal domain expert | Medium (template refinements, new crime types) |
| L3 | Developer | Low (docx generation logic) |
| L4 | Any contributor | Medium (documentation updates) |

## Adding a New Skill

1. Create `skills/<skill-name>/SKILL.md` with frontmatter (name, description, triggers)
2. Reference `../../references/cta-config.md` for CTA (do not duplicate attorney info)
3. If the skill needs a generation script, place it in `skills/<skill-name>/scripts/`
4. If the skill has reference material, place it in `skills/<skill-name>/references/`
5. Update `README.md` with the new skill description
6. Update `docs/product-specs/index.md`
