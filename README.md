<!--
  Everything below is verifiable from the linked repositories.
  No static status badges: a badge that asserts "CI passing" without reading the
  build is decoration, and decoration is what makes a profile look generated.
  Every badge here is a live GitHub Actions endpoint.
-->

### Security tooling for AI agents

Agents now hold tools, memory and delegated credentials. That makes the scaffold
around the model — system prompt, tool schemas, permission gates, memory store,
inter-agent messages — an attack surface in its own right, and the part most
teams own but test least.

That is the problem I work on. The pattern I keep coming back to:

> **A detector that cannot be shown to stop detecting is not a detector.**
> So every project here ships with an adversarial counterpart, or a benchmark
> gate that fails the build.

---

#### What I'm building

**[LogAgent-SOC](https://github.com/saadcyber21/ai-soc-detection-engine)** —
autonomous threat hunting with explainable-AI forensics
![CI](https://github.com/saadcyber21/ai-soc-detection-engine/actions/workflows/ci-cd.yml/badge.svg)

Multi-agent pipeline over local open-weight models: zero data egress, no cloud
API calls. Zero-shot heterogeneous log parsing with verbatim source-span
validation, a bounded ReAct loop grounded in a closed MITRE ATT&CK set, and
forensic reports with calibrated confidence. Ships a CyberMetric benchmark —
hallucination rate, precision/recall, p95 latency, Brier calibration — that
**gates CI**, so a regression in accuracy fails the build rather than the demo.

**[zero-trust-ai-gateway](https://github.com/saadcyber21/zero-trust-ai-gateway)** —
RBAC and payload-inspection proxy for agent tool calls
![CI](https://github.com/saadcyber21/zero-trust-ai-gateway/actions/workflows/ci.yml/badge.svg)

Sits between an agent and the systems it can reach. Validates agent identity
against role policy and inspects payloads for destructive operations —
`DROP TABLE`, command chaining, reverse shells — before they execute.

**[secret-sweep](https://github.com/saadcyber21/secret-sweep)** — credential and
secret detection across repositories
![CI](https://github.com/saadcyber21/secret-sweep/actions/workflows/ci.yml/badge.svg)

**[Phish-Guard](https://github.com/saadcyber21/Phish-Guard)** — phishing detection
![CI](https://github.com/saadcyber21/Phish-Guard/actions/workflows/ci.yml/badge.svg)

**[plan-guard](https://github.com/saadcyber21/plan-guard)** — guardrails for agent execution plans
![CI](https://github.com/saadcyber21/plan-guard/actions/workflows/ci.yml/badge.svg)

**[kql-genie](https://github.com/saadcyber21/kql-genie)** — natural language to KQL for threat hunting
![CI](https://github.com/saadcyber21/kql-genie/actions/workflows/ci.yml/badge.svg)

**[nist-ai-compliance-mapper](https://github.com/saadcyber21/nist-ai-compliance-mapper)** —
maps AI system controls to NIST AI RMF

---

#### In progress

**quillback** — offline red-team harness for AI agent scaffolds. 18 deterministic
attacks mapped to the OWASP Top 10 for Agentic Applications (ASI 2026), SARIF
export for GitHub code scanning, zero runtime dependencies, no API keys. On the
bundled deliberately-vulnerable reference agent it reports 19 findings; on the
hardened variant, 0. That contrast is the point — it is what separates a scanner
from a noise generator.

---

#### Honest limitations

I would rather state these than have you find them.

- **LogAgent-SOC** needs Ollama or vLLM for real model inference. CI exercises
  the deterministic heuristic backend, so a green build proves the pipeline,
  parser, grounding and benchmark gate — not model quality on your data.
- **quillback** covers 6 of the 10 ASI categories. ASI05, ASI08, ASI09 and
  ASI10 have no attacks yet, and its report says so explicitly rather than
  implying full coverage.
- Several older repositories here are stale experiments. They are kept for
  history, not as current work.

---

#### How I work

- Tests must prove both halves: that a flaw is detected, **and** that detection
  stops once the flaw is fixed. One half alone is easy to fake.
- Refusals have to be observable. A control that fails silently is
  indistinguishable from no control, and I have shipped that bug twice.
- Nothing gets a badge asserting a state it does not read.

Security issue in one of these? Please open a **private** report rather than a
public issue.
