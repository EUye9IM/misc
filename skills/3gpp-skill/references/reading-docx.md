# Reading 3GPP Specification Documents

`.docx` files extracted from 3GPP spec ZIPs are long (often 30K+ lines). This document describes the recommended reading methodology.

## Workflow: Scan Headings → Read Tables → Check Notes

### Step 1: Open the document

```python
from docx import Document
doc = Document("29518-k00.docx")
```

### Step 2: Scan headings to find relevant sections

```python
for i, para in enumerate(doc.paragraphs):
    if para.style.name.startswith('Heading'):
        print(f"[{i}] {para.text.strip()}")
```

This prints all headings, ordered by document position. The number in brackets `[i]` is the paragraph index — you can jump to that position to read the content below it.

Heading depth convention (3GPP specs use `toc N` styles):
- `toc 2`: Top-level (1. Introduction)
- `toc 3`: Section (2.1)
- `toc 4–5`: Subsection (2.2.1, 6.2.6.2)
- `toc 6`: Deepest level (6.2.6.2.5 Type: AmfEventReport)

A spec typically has 1500+ headings. **Scan first, never read fully.**

### Step 3: Identify candidate sections

From the heading list, pick **1–3 sections** that likely contain the answer.

Typical patterns:
- A type definition section (e.g. "Type: AmfEventReport") has a table listing all its fields
- An operation section (e.g. "CreateUEContext") has request/response parameter tables
- An enumeration section lists all allowed values

Take note of the **paragraph index** of each selected heading.

### Step 4: Read the definition table

After identifying a heading, find the table that belongs to that section. Tables are ordered sequentially in the document, and each table's heading paragraph (e.g. "Table 6.2.6.2.5-1: Definition of type XXX") appears right after the section heading.

```python
# Approach 1: Find tables by section heading text
target_section = "Definition of type AmfEventReport"

for i, para in enumerate(doc.paragraphs):
    if target_section in para.text:
        print(f"Found at paragraph {i}")

# Then find the next table using Attribute name header
for table in doc.tables:
    header = ' '.join([c.text.strip() for c in table.rows[0].cells])
    if 'Attribute name' in header:
        # This is a definition table — check first few rows to confirm it's the right one
        field_names = [row.cells[0].text.strip() for row in table.rows[1:4]]
        print(fields_names)
```

### Step 5: Read table rows

Once you've identified the right table, read all its rows:

```python
for row in table.rows:
    cells = [c.text.strip().replace('\n',' ') for c in row.cells]
    print(' | '.join(cells))
```

### Step 6: Understand the table columns

| Column | Meaning |
|--------|---------|
| Attribute name | IE identifier |
| Data type | Reference to another type or primitive |
| **P** | **Presence: M=Mandatory, C=Conditional, O=Optional** |
| Cardinality | `1`=exactly one, `0..1`=optional, `1..N`=one or more |
| Description | Semantics + conditions for `C` fields |
| Applicability | Feature flag that gates this IE (e.g. `ENABLE`, `AIML_CN`) |

### Step 7: Read the NOTE(s)

After the table, one or more **NOTE** paragraphs explain the conditions for `C` (Conditional) fields. These are critical — they contain the actual rules.

```python
# Find NOTEs near the definition heading
for para in doc.paragraphs[heading_idx:heading_idx + 50]:
    if para.text.strip().startswith('NOTE'):
        print(para.text.strip())
        break
```

### Step 8: Follow data type references

If a field's Data Type is a named type (e.g. `AmfEventState`), find **its** definition section from the heading list and repeat the process. Types may be objects (with their own table) or enums.

## Tips

- **Do NOT scan the entire document** — find the right heading first, then read only 1–3 tables and their NOTEs
- Use the **paragraph index** from Step 2 to jump directly to relevant content
- Actually read table rows by iterating — do not try to remember the table structure from your training data
- The **P column** is only in the Word .docx, not in the YAML files — always read from the docx for presence rules
- You can also install python-docx via `uv pip install python-docx` for structured access
- If python-docx is not available, you can use `unzip` + `read_file` on the extracted XML instead, but tables will lose their structure
