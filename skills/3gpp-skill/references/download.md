# Downloading 3GPP Specifications

## Base URL

```
ftp://ftp.3gpp.org/Specs/archive/
```

## Directory Structure

```
Specs/archive/
├── 21_series/         # Requirements
├── 23_series/         # Architecture (e.g. TS 23.501)
├── 24_series/         # UE-network signaling
├── 29_series/         # Core network protocols
├── 36_series/         # LTE/E-UTRAN
├── 37_series/         # Multi-RAT
└── 38_series/         # NR (5G)
    ├── 38.300/        # NR Overall description
    ├── 38.211/        # NR Physical channels
    ├── 38.331/        # NR RRC
    └── ...
```

Each spec has its own directory containing versioned zip files.

## Workflow: List → Pick → Download → Read

### Step 1: List available versions

```bash
curl -s "ftp://ftp.3gpp.org/Specs/archive/38_series/38.300/"
```

Returns a listing like:

```
38300-ia0.zip    (6.3MB)  # Rel-18
38300-j20.zip    (7.9MB)  # Rel-19
38300-j30.zip    (7.7MB)  # Rel-19, newer
```

### Step 2: Pick the latest version

Version code letters sort alphabetically: `a < b < ... < f < g < h < i < j < k`

So `j30` > `j20` > `ia0`. Pick the one with the highest letter and number.

### Step 3: Download

```bash
curl -LO "ftp://ftp.3gpp.org/Specs/archive/38_series/38.300/38300-j30.zip"
```

The URL pattern is:
```
ftp://ftp.3gpp.org/Specs/archive/{series}_series/{spec_no_dots}/{filename}
```

Where `{series}` is the numeric series (38, 23, 29, etc.), `{spec_no_dots}` is the spec number without dots (38300), and `{filename}` is the zip filename from the listing.

### Step 4: Extract and read

```bash
unzip -o 38300-j30.zip -d ./38300-j30
```

Then use `read_file` on the `.docx` — it handles Word documents automatically.

## Version Code Reference

| Letter | Release | Example |
|--------|---------|---------|
| f | Rel-15 | `f00` = v15.0.0, `f10` = v15.1.0 |
| g | Rel-16 | `g00` = v16.0.0 |
| h | Rel-17 | `h00` = v17.0.0 |
| i | Rel-18 | `i00` = v18.0.0, `ia0` = v18.10.0 |
| j | Rel-19 | `j00` = v19.0.0 |
| k | Rel-20 | `k00` = v20.0.0 |

## ZIP Contents

Most spec zips contain:

| File | Format | Description |
|------|--------|-------------|
| `{spec}-{version}.docx` | Word | Formal specification text |
| `TS{spec}_{Service}.yaml` | YAML/OpenAPI | API definitions (for 29-series) |

## Tips

- Always pick the **latest version** (highest letter + number combo) from the directory listing
- Previous versions remain in the directory for reference
- `read_file` extracts .docx text automatically — no extra conversion needed
- OpenAPI YAML files (29-series) contain exact REST API definitions
