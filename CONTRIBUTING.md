# Contributing to OM AI Tools

Thank you for helping make AI tooling safer for everyone.

## What we accept

- **Tool assessments** — security reviews of npm, pip, or brew packages used in AI workflows
- **Safe usage guides** — practical hardening steps for tools already assessed
- **Corrections** — if a finding is factually wrong or a tool version changed materially

We do not accept:
- Promotional content for any tool (including omopen.ai products)
- Assessments that are speculative without supporting evidence from the source
- Assessments of tools you are affiliated with or financially benefit from

## Assessment template

Copy `tools/_template/README.md` and fill in each section. The minimum required sections are:

1. **What is it** — what the tool does, who maintains it, where it lives
2. **Assessment methodology** — the exact commands you ran to inspect it (no install)
3. **Findings** — numbered, each with a severity and a "what to do"
4. **What was NOT found** — explicitly state the concerns you checked and did not observe
5. **Risk summary table**
6. **Safe usage guide** — actionable steps for the cautious user
7. **Alternatives** — at least two

## Severity levels

| Level | Meaning |
|-------|---------|
| 🔴 High | Active risk under plausible default usage |
| 🟡 Medium | Risk under a realistic configuration the user might not notice |
| 🟢 Low | Theoretical or edge-case risk |
| ℹ️ Informational | Not a risk — just worth knowing |

## Rules for findings

- **No speculation.** If you can't show the code line, the finding doesn't stand.
- **Distinguish design from defect.** A MITM proxy that proxies traffic is doing its job. Note it as informational, not as a vulnerability.
- **Be specific about versions.** All findings must state which version was reviewed.
- **Show your grep.** Include the actual command you ran and the output snippet that triggered the finding.

## Submitting

1. Fork this repo
2. Create a branch: `assess/<tool-name>-<version>` (e.g. `assess/litellm-1.40.0`)
3. Add your assessment under `tools/<tool-name>/README.md`
4. Open a PR — the template will ask for a brief summary and your inspection method

## Code of conduct

Assessments must be factual, evidence-based, and respectful. We will not merge assessments that read as attacks on maintainers rather than reviews of code.
