# Downloading 3GPP Specifications

## Base URL

All 3GPP specifications are available at:

```
https://www.3gpp.org/ftp/Specs/archive/
```

## Directory Structure

```
Specs/archive/
├── 21_series/         # Requirements
│   └── 21.xxx/
│       └── 21xxx-vv.zip
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

## Download Command

```bash
# Using curl
curl -LO "https://www.3gpp.org/ftp/Specs/archive/29_series/29.518/29518-k00.zip"

# Using wget
wget "https://www.3gpp.org/ftp/Specs/archive/29_series/29.518/29518-k00.zip"
```

## Version Naming

File format: `{spec_no_dots}-{version_code}.zip`

Version codes:
- `fXX` → Rel-15 (e.g. `f10` = v15.1.0)
- `gXX` → Rel-16
- `hXX` → Rel-17
- `iXX` → Rel-18
- `jXX` → Rel-19
- `kXX` → Rel-20

The highest version in a spec directory is usually the latest.

## ZIP Contents

Most spec zips contain:

| File | Format | Description |
|------|--------|-------------|
| `{spec}-{version}.docx` | Word | Formal specification text |
| `TS{spec}_{Service}.yaml` | YAML/OpenAPI | API definitions (for 29-series) |

## Reading the Spec

```bash
# Unzip
unzip -o 29518-k00.zip -d ./29518-k00

# Read with standard tools (read_file handles .docx automatically)
# Inside the agent, just use read_file on the .docx directly
```

## Tips

- Always download the **latest version** (highest letter) for the most up-to-date specification
- Previous versions are kept in the same directory for reference
- The `read_file` tool can extract text from .docx files directly — no need for extra conversion
- OpenAPI YAML files (for 29-series core network specs) contain the exact REST API definitions
