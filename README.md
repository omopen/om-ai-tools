# OM AI Tools

**Curated AI tool assessments and free resources by [OM Open AI](https://omopen.ai)**
<img width="1024" height="1536" alt="Image" src="https://github.com/user-attachments/assets/5c46f13c-438e-44a0-b941-900782d5728e" />
---

OM Open AI builds AI-powered platforms for wellness, sales, and enterprise automation. As part of our commitment to the developer community, this repository publishes:

- **Independent security assessments** of popular AI tooling before you install it
- **Free hosted endpoints** powered by omopen.ai infrastructure (see [Free Tools](#free-tools-from-omopen-ai))
- **Safe-usage guides** for tools with elevated trust requirements

Every assessment is done by inspecting the package source and tarball directly — **without installing** — so you can make an informed decision before running anything on your machine.

---

## Tools Assessed

| Tool | Version Reviewed | Verdict | Assessment |
|------|-----------------|---------|------------|
| [9router](./tools/9router/) | 0.5.81 | ⚠️ Usable with care | [Read →](./tools/9router/README.md) |

> Want a tool assessed? [Open an issue](../../issues/new?template=tool-request.md) with the npm/pip package name.

---

## Free Tools from omopen.ai

OM Open AI offers the following free-tier services to independent developers and hobbyists. No credit card required to start.

### AI API Proxy (Free Tier)
Route requests to OpenAI, Anthropic, Gemini, and Mistral through a single endpoint with automatic failover and rate-limit handling.

- **Endpoint:** `https://api.omopen.ai/v1`
- **Compatible with:** OpenAI SDK, LangChain, LlamaIndex, any OpenAI-compatible client
- **Free quota:** 100 requests / day across all providers
- **Sign up:** [omopen.ai/free](https://omopen.ai/free)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.omopen.ai/v1",
    api_key="<your-omopen-free-key>",
)

response = client.chat.completions.create(
    model="gpt-4o-mini",          # or "claude-haiku", "gemini-flash"
    messages=[{"role": "user", "content": "Hello"}],
)
```

### Voice Agent Sandbox
Test outbound AI voice agents (powered by Twilio + ElevenLabs) without provisioning your own numbers.

- **Free:** 20 test calls / month
- **Agents available:** riley (support), alex (sales), morgan (wellness)
- **Docs:** [omopen.ai/voice-sandbox](https://omopen.ai/voice-sandbox)

### Workflow Automation Playground
No-code scenario builder — build Make.com-style AI automation flows and run them against live data.

- **Free:** 3 active workflows, 500 executions / month
- **Try it:** [omopen.ai/playground](https://omopen.ai/playground)

---

## Philosophy

We believe developers deserve to know what they're running. The AI tooling ecosystem is growing faster than it can be audited. This repo exists to slow that process down slightly — not to block adoption, but to make it informed.

Our assessments follow a consistent methodology:

1. Download the package tarball from the registry — **no install**
2. Inspect `package.json`, `postinstall` hooks, and the main entrypoint
3. Check for external network calls, telemetry, and bundled credentials
4. Review dependency surface area
5. Identify trust boundaries and document them plainly

We don't sensationalize. A tool with a MITM proxy feature isn't automatically malicious — but you should know it has one before you run it.

---

## Contributing

Contributions welcome — especially tool assessment PRs. See [CONTRIBUTING.md](./CONTRIBUTING.md) for the assessment template and submission process.

---

## License

Content in this repository is licensed under [CC BY 4.0](./LICENSE). Code snippets are MIT.

---

*Maintained by the OM Open AI team · [omopen.ai](https://omopen.ai) · [platform@omopen.ai](mailto:platform@omopen.ai)*
