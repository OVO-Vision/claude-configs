---
description: Generate comprehensive documentation for implemented features
allowed-tools: ["Task"]
---

# Document Feature

Generate documentation for implemented AL features.

## Usage

```
/document
```

**Prerequisite:** Must have implemented code and planning documents.

## What It Does

Spawns docs-writer agent to:
1. Detect documentation location (wiki/ or docs/)
2. Read implementation artifacts
3. Generate feature documentation
4. Create/update API reference (if public codeunits)
5. Update CHANGELOG.md
6. Maintain documentation folder structure

## Documentation Generated

### Feature Documentation
- Clear user-focused explanation
- Technical implementation details
- Configuration steps
- Validation rules and error messages
- Testing guidance
- Troubleshooting tips

### API Documentation (If Applicable)
- Public procedure signatures
- Parameter descriptions
- Usage examples
- Error conditions
- Performance notes

### CHANGELOG Update
- Version entry with changes
- Links to detailed feature docs

## Folder Structure Created

```
docs/ (or wiki/)
├── Features/
│   └── [feature-name].md
├── API/
│   └── [codeunit-name].md
├── Setup/
├── Architecture/
├── CHANGELOG.md
└── README.md
```

## When to Use

- Feature implementation complete
- Before deployment
- Want user-facing documentation
- Need API reference for other developers
- Maintaining project documentation

## Output

- Documentation files in `docs/` or `wiki/`
- Updated CHANGELOG.md
- Session log entry

## Example

```
User: /document

Claude: Spawning docs-writer...
[Agent runs]

Claude: Documentation complete → docs/

Generated:
- Features/customer-credit-limit.md (245 lines)
- API/credit-limit-management.md (180 lines)
- Updated CHANGELOG.md

Documentation structure maintained.
```

---

**Creates comprehensive documentation from implementation artifacts.**
