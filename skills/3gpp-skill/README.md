# 3GPP Expert Skill

A comprehensive 3GPP telecommunications skill — covering everything from GSM (1992) through 6G (Release 21). Forked from [lugasia/3gpp-skill](https://github.com/lugasia/3gpp-skill).

## What It Does

This skill provides deep, standards-grounded expertise across the full 3GPP ecosystem:

- **All generations**: 2G/GSM, 3G/UMTS, 4G/LTE, 5G NR, 5G-Advanced, 6G
- **All releases**: Phase 1 through Release 21, with detailed feature breakdowns
- **Protocol stacks**: PHY, MAC, RLC, PDCP, SDAP, RRC, NAS — with LTE vs NR differences
- **Core network**: EPC to 5GC/SBA evolution, all network functions (AMF, SMF, UPF, etc.)
- **Deployment**: Network planning, spectrum strategy, migration paths (NSA/SA options), O-RAN
- **Security**: Authentication (EPS-AKA, 5G-AKA), SUPI/SUCI, IMSI catcher analysis
- **Practical consulting**: Link budgets, cell planning, troubleshooting, interoperability

## Spec Download & Reading

For detailed protocol/procedure questions, the skill can:

1. **Download** the actual 3GPP specification from `ftp://ftp.3gpp.org/Specs/archive/`
2. **Read** the specification document directly — scan headings, identify relevant sections, and extract table definitions with presence rules (M/C/O)
3. **Cite** exact section numbers and spec versions in answers

See `references/download.md` and `references/reading-docx.md` for details.

## What's Inside

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill instructions — 7 knowledge domains, response patterns |
| `references/releases.md` | Release-by-release reference (Phase 1 → Rel-21) with spec series table |
| `references/phy-layer.md` | PHY layer deep-dive: synchronization signals, physical channels, RACH |
| `references/working-groups.md` | RAN/SA/CT Working Group structure with owned specs |
| `references/download.md` | How to download specs from ftp.3gpp.org |
| `references/reading-docx.md` | Methodology for reading long spec .docx files |

## Coverage

### Releases

| Era | Releases | Key Technologies |
|-----|----------|-----------------|
| 2G | Phase 1 – Rel-98 | GSM, GPRS, EDGE |
| 3G | Rel-99 – Rel-7 | UMTS, WCDMA, HSPA, HSPA+ |
| 4G | Rel-8 – Rel-14 | LTE, LTE-A, LTE-A Pro, NB-IoT, C-V2X |
| 5G | Rel-15 – Rel-17 | NR, 5GC/SBA, NTN, RedCap, Sidelink |
| 5G-Adv | Rel-18 – Rel-19 | AI/ML, XR, Ambient IoT, MIMO evolution |
| 6G | Rel-20 – Rel-21 | Sub-THz, ISAC, AI-native, digital twins |

### Specification Series

Covers TS 21–38 series with go-to specs for architecture (TS 23.501), NR radio (TS 38.xxx), NAS (TS 24.501), security (TS 33.501), and more.

## License

MIT License — see [LICENSE](LICENSE) for details.

Built by [@lugasia](https://github.com/lugasia).
