# {Tool Name} — Independent Security Assessment

**Assessed by:** OM Open AI  
**Package:** [`{package-name}`]({registry-url}) on {npm/pip/brew}  
**Version reviewed:** `{x.y.z}`  
**Assessment date:** {YYYY-MM-DD}  
**Method:** Tarball inspection — package downloaded and extracted, **never installed or executed**  
**Verdict:** {✅ Safe to use / ⚠️ Usable with care / 🔴 Do not install} — {one-line reason}

---

## What is {Tool Name}?

{2–4 sentence description of what the tool does, who maintains it, and what problem it solves.}

---

## Assessment Methodology

```bash
# Show exact commands used — no install, tarball only
curl -sL {tarball-url} -o tool.tgz
tar tzf tool.tgz | head -40
tar xzf tool.tgz {key files to extract}
grep -rE "{pattern}" {extracted-path}/
rm -rf {extracted-path} tool.tgz
```

---

## Findings

### Finding 1 — {Title}

**Severity:** {🔴 High / 🟡 Medium / 🟢 Low / ℹ️ Informational}

{Description of what was found, why it matters, and the evidence (code snippet or grep output).}

**What to do:** {Specific, actionable step the user can take.}

---

### What was NOT found

| Concern | Result |
|---------|--------|
| External telemetry / analytics | ✅ Not found |
| Phone-home on startup | ✅ Not found |
| API key exfiltration | ✅ Not found |
| Crypto mining code | ✅ Not found |
| {Add rows for what you checked} | |

---

## Risk Summary

| Risk | Level | Condition |
|------|-------|-----------|
| {Risk description} | 🔴/🟡/🟢 | {When this applies} |

---

## Safe Usage Guide

{Step-by-step instructions for the cautious user.}

---

## Alternatives

| Tool | Type | Notes |
|------|------|-------|
| {Name} | {Type} | {Notes} |

---

## Disclaimer

This assessment reflects `{package}@{version}` as reviewed on {date}. Provided for informational purposes only. Not affiliated with or endorsed by the tool's authors.

---

*Assessment by [OM Open AI](https://omopen.ai) · Licensed [CC BY 4.0](../../LICENSE)*
