---
title: Finding broker IDs from integration/importer names
when: When a user refers to a system name (like 'Quantix', 'TMW', etc.) that is actually an importer/integration rather than a broker name, and you need to find the actual broker ID
---

## Problem
Users may refer to branches or operations by their integration/importer name (e.g., 'Quantix') rather than the broker entity name. These names appear in the system as importer keys but are not directly queryable as broker names.

## Discovery Process

### 1. Check if it's a direct broker match
```bash
xbe view brokers list --q <search-term> --json
```

If this returns empty results, the term might be an integration/importer name rather than a broker name.

### 2. Search the knowledge base
```bash
xbe knowledge search <search-term>
```

Look for references in:
- `flag_desc --importer`: These descriptions often show importer keys (e.g., `quantix_tmw`)
- Related commands that use the term

### 3. Try searching project offices
```bash
xbe view project-offices list --q <search-term> --json
```

This can help identify if the term is associated with a specific office or division.

### 4. Search brokers more broadly
If you identified an importer pattern (e.g., `quantix_tmw`), try searching for related terms:
```bash
xbe view brokers list --json | jq '.[] | select(."company-name" | test("<pattern>"; "i"))'
```

### 5. Once you have the broker ID
Query the resources you need:
```bash
xbe view transport-orders list --broker <broker-id> --start-on <date> --end-on <date> --is-managed true --json
```

## Example Flow
User asks about "Quantix branch":
1. Search brokers for "Quantix" → no results
2. Search project offices for "Quantix" → no results
3. Search knowledge base → find references to `quantix_tmw` importer
4. Realize this is an integration name, not a broker name
5. Search brokers more carefully → find broker ID 297 is associated with Quantix operations
6. Use broker ID 297 for queries

## Key Insight
Integration/importer names (like `quantix_tmw`) are technical identifiers for data import sources, not the primary broker entity names. You need to map from the integration name to the actual broker ID through knowledge base search or by understanding the organizational structure.
