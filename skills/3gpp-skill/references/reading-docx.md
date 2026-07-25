# Reading 3GPP Specification Documents

`.docx` files extracted from 3GPP spec ZIPs are long (often 30K+ lines). This document describes the recommended reading methodology.

## Workflow: Read Headings → Select Sections → Read Tables

### Step 1: Open the document

```python
from docx import Document
doc = Document("29518-k00.docx")
```

### Step 2: Read headings to find relevant sections

```python
for i, para in enumerate(doc.paragraphs):
    if para.style.name.startswith('Heading'):
        print(f"[{i}] {para.style.name}: {para.text.strip()}")
```

Headings use `toc 1–9` styles internally, where `toc 2` ≈ H1, `toc 6` ≈ H5 (deepest detail). A spec typically has **1500+ headings** — always scan first, never read fully.

### Step 3: Identify candidate sections

From the heading list, pick **1–3 sections** that likely contain the answer (e.g. the Type definition + the referenced sub-type). Read only those.

### Step 4: Read the definition table

Navigate to the target section and read its table:

```python
for para in doc.paragraphs:
    if 'Definition of type AmfEventReport' in para.text:
        break

for table in doc.tables:
    header = ' '.join([c.text for c in table.rows[0].cells])
    if 'Attribute name' in header and 'supi' in str(table.rows[1].cells[0].text).lower():
        for row in table.rows:
            cells = [c.text.strip().replace('\n',' ') for c in row.cells]
            print(' | '.join(cells))
```

### Step 5: Understand the table columns

| Column | Meaning |
|--------|---------|
| Attribute name | IE identifier |
| Data type | Reference to another type or primitive |
| **P** | **Presence: M=Mandatory, C=Conditional, O=Optional** |
| Cardinality | `1`=exactly one, `0..1`=optional, `1..N`=one or more |
| Description | Semantics + conditions for `C` fields |
| Applicability | Feature flag that gates this IE (e.g. `ENABLE`, `AIML_CN`) |

### Step 6: Read the NOTE(s)

After the table, **NOTEs** explain the conditions for `C` (Conditional) fields. Always read them — they contain the actual rules.

### Step 7: Follow data type references

If a field's Data Type is another type (e.g. `AmfEventState`), find **its** definition section and read its table/YAML schema. Types may be defined as simple objects (with properties) or as enums.

### Tips

- **Do NOT scan the entire document** — find the right heading first, then read only 1–3 tables
- For types defined in the same spec, use the TOC to locate the definition section
- For types defined in other specs (e.g. `TS29571_CommonData.yaml`), download and read that spec too
- OpenAPI YAML files within the ZIP contain the schema definitions but lack the `P` (Presence) column — use the Word docx for presence rules
