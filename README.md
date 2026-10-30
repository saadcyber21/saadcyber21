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

**[quillback](https://github.com/saadcyber21/quillback)** —
offline red-team harness for AI agent scaffolds
[![CI](https://github.com/saadcyber21/quillback/actions/workflows/ci.yml/badge.svg)](https://github.com/saadcyber21/quillback/actions)

No API keys, no tokens, deterministic — so it can gate a pull request. Every
tool in this space (garak, PyRIT, promptfoo, deepteam) attacks a *live model*;
quillback attacks the **scaffold** you actually own: system prompt, tool schemas,
permission gates, memory store, inter-agent messaging. Those artefacts are
static, so no inference is needed and the same input always gives the same
findings.

18 attacks across OWASP ASI01/02/03/04/06/07, SARIF export, zero runtime
dependencies. On the bundled deliberately-vulnerable reference agent it reports
**19 findings**; on the hardened variant, **0**. CI asserts both directions on
every commit, because an attack that cannot be shown to stop detecting is not a
detector. It scans itself: those 19 findings are live in the repo's Security tab.

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

**[secret-sweep](https://github.com/saadcyber21/secret-sweep)** —
credential detection with a *measured* error rate
![CI](https://github.com/saadcyber21/secret-sweep/actions/workflows/validate.yml/badge.svg)

One stdlib-only file, no install step. Fifteen pattern rules for prefixed
credentials, plus an entropy detector for the unprefixed random strings that
rules cannot describe.

Entropy detection is a heuristic, so I measured it rather than describing it.
Against the CPython stdlib — 669 files, 304,467 lines — it reports **23 false
positives (0.0076% of lines)** at **99.5% recall** on generated credentials.
`scripts/calibrate.py` reproduces both numbers; CI fails the build if recall
drops below its floor.

Worth reading for the bug it started from: the original detector was
`entropy > 4.5`, which **cannot fire on any token shorter than 24 characters**,
because entropy is bounded by `log2(min(len, alphabet))`. A 20-character AWS
access key ID was undetectable by construction — including the one in the
repo's own sample file, which the README pointed at as the demonstration. It
flagged 610 lines on the same corpus. Fixing it meant separating randomness
from *compressed English*: `MAX_RETRY_ATTEMPTS_CONFIG` has a flat character
distribution and passes any pure entropy test, so detection now combines
entropy ratio, alphabet coverage, and case-transition rate with structural
guards for sequential constants, identifier syntax, and reference forms like
`actions/checkout@v4`.

**[Phish-Guard](https://github.com/saadcyber21/Phish-Guard)** — phishing detection
![CI](https://github.com/saadcyber21/Phish-Guard/actions/workflows/validate.yml/badge.svg)

**[plan-guard](https://github.com/saadcyber21/plan-guard)** — guardrails for agent execution plans
![CI](https://github.com/saadcyber21/plan-guard/actions/workflows/validate.yml/badge.svg)

**[kql-genie](https://github.com/saadcyber21/kql-genie)** — natural language to KQL for threat hunting
![CI](https://github.com/saadcyber21/kql-genie/actions/workflows/validate.yml/badge.svg)

**[nist-ai-compliance-mapper](https://github.com/saadcyber21/nist-ai-compliance-mapper)** —
maps AI system controls to NIST AI RMF

---

#### Honest limitations

I would rather state these than have you find them.

- **LogAgent-SOC** needs Ollama or vLLM for real model inference. CI exercises
  the deterministic heuristic backend, so a green build proves the pipeline,
  parser, grounding and benchmark gate — not model quality on your data.
- **quillback** is v0.1.0 and covers 6 of the 10 ASI categories. ASI05, ASI08,
  ASI09 and ASI10 have no attacks yet. Its report lists those as blind spots on
  every run rather than implying full coverage, and it has no framework
  adapters — you write a `Scaffold` subclass by hand.
- **secret-sweep** scans the working tree only — no git history, so it will not
  find a secret that was committed and later deleted. It does not verify
  credentials against provider APIs, so it cannot distinguish a live key from a
  rotated one. Its 23 residual false positives on CPython are CamelCase Win32
  bindings and zipfile struct constants; entropy findings need human review, and
  the recall corpus is generated rather than harvested, so 99.5% honestly means
  "99.5% of the shapes I modelled". Use gitleaks or trufflehog for history and
  verification.
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
