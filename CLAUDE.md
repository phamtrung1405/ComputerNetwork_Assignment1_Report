# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Not a code project: a Vietnamese-language LaTeX report (HCMUT Computer Networks, Assignment 2 – "Network Design for a Company") documenting a multi-site hospital network simulated in Cisco Packet Tracer 9.0.1. The 29-network-device topology (names follow the reference diagram `h.png`, copied to `graphics/topology.png`): Main Site (HCMC), DMZ/Internet edge (MAIN-FW-01/MAIN-FW-02, MAIN-VPN, ISP1-R), WAN backbone (Carrier-R leased line, ISP2-R broadband backup), Branch 1 (DBP) and Branch 2 (BHTQ). The assignment spec is `spec.pdf`.

## Build

No build script. Compile with a LaTeX toolchain (vntex is used, so pdfLaTeX/latexmk with Vietnamese support):

```
latexmk -pdf main.tex      # or: pdflatex main.tex; bibtex main; pdflatex main.tex (x2)
```

`main.pdf` is committed (`.gitignore` ignores only aux/log files, not PDFs). Rebuild it when content changes.

## Structure

- `main.tex` – entry point: cover (`cover-page.sty`), `Content/score.tex`, TOC, `Content/cisco.tex`, then bibliography from `references.bib` (`\nocite{*}`, IEEEtran style).
- `Config/` – preamble split into `packages.tex`, `layout.tex`, `code_style.tex` (listings style for device configs).
- `Content/cisco.tex` – report body entry: intro, the `\tcfig` figure macro, then `\input`s `step1.tex`–`step6.tex` (one per assignment step, written from the `step-*.md` notes) and finally `Content/security.tex` (talented-engineer security section). Security is not listed in `main.tex`.
- `graphics/tc_pic/` – test-case evidence screenshots, named by case ID (`LAN_*`, `WAN_*`, `HA_*`, `SEC_*`, `DMZ_01`, `INT_01`, `VPN_*`). They are referenced from `cisco.tex` (LAN/WAN/HA) and `security.tex` (SEC/DMZ/INT/VPN).

## Markdown companion docs (not compiled into the PDF)

- `config.md` – full CLI configs for all 25 devices (source of truth for the configs quoted in the report).
- `simulate.md` – demo/simulation scenarios and IP/VLAN plan.
- `test.md` – real test-case log (TC-01..TC-07: FAIL→PASS causes and fixes) plus backup/failover scenarios.
- `step-1…7-*.md` – per-step working notes that the `.tex` steps are written from.
- `recapture_images.md` – screenshots to retake/add against the reference diagram.
- `maintest.txt` – the mandatory test-case checklist derived from the assignment (LAN-/WAN-/etc. IDs).

## Conventions to preserve

- The report is deliberately conservative about claims: test evidence is scoped to what screenshots actually show (e.g. `SEC_01.png` shows Guest reaching DMZ `10.10.50.10`, so guest isolation is recorded as FAIL; `SEC_02.png` only proves one blocked ping; no evidence exists for SSH-from-Jump-Host or Internet→DMZ NAT; only 15/25 configs were exported). Do not upgrade results to PASS or generalize beyond the evidence.
- Keep `config.md`, `simulate.md`, `test.md` and the `.tex` content consistent (IPs, VLAN IDs, device names, HSRP priorities, OSPF/floating-static AD 150 design, native VLAN handling).
- Credentials are redacted (`[REDACTED]`, `[ADMIN_USER]`); never write real passwords into docs.
- Text is Vietnamese; keep new content in Vietnamese.
- `h.png` (repo root) is the reference topology and the source of truth for device names; `graphics/topology.png` is its copy used in the report.
