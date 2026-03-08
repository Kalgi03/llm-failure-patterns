# LLM Failure Patterns

![MIT License](https://img.shields.io/badge/license-MIT-green)
![Version](https://img.shields.io/badge/version-0.4.2-blue)
![Patterns](https://img.shields.io/badge/patterns-52-teal)

An open-source taxonomy of how AI integrations fail in production, with a live classifier to diagnose failure types instantly.

**Live demo:** https://kalgi03.github.io/llm-failure-patterns/llm-failure-patterns.html

---

## The Problem

Every team integrating an LLM into production hits the same wall: the AI fails, and nobody has a clear name for what just happened.

Is it a rate limit? A context overflow? A prompt injection? A hallucination cascade? These failures are real, costly, and deeply unintuitive to people coming from traditional software backgrounds. The error messages are often cryptic. The fixes are scattered across docs, forums, and GitHub issues.

I spent time as a support engineer watching these failures repeat across customers, companies, and use cases. The patterns were obvious once you had seen enough of them. But there was no structured, searchable reference. This project is that reference.

---

## How I Designed the Taxonomy

I started by listing raw failure modes with no grouping: rate limits, hallucinations, context overflow, prompt injection, JSON parse errors, latency spikes. Around 60 in total.

From there, I grouped them by **where in the stack the failure originates**:

| Category | Description | Count |
|---|---|---|
| API and Infrastructure | Provider-side: rate limits, auth, timeouts, quotas | 12 |
| Model Behavior | Model-side: hallucinations, refusals, instruction drift | 9 |
| Context Failures | Token math: overflow, truncation, context poisoning | 8 |
| Security Failures | Attack surface: prompt injection, jailbreaks, data leakage | 7 |
| Integration Failures | App-side: JSON parse errors, tool call failures, streaming drops | 10 |
| Performance Failures | Cost and speed: latency spikes, token inefficiency, cold starts | 6 |

Each pattern has a consistent structure:
- **Name** (short, searchable)
- **Category** (one of the six above)
- **Severity** (S1 Critical / S2 Major / S3 Minor, borrowed from incident management)
- **HTTP code** where applicable
- **Plain-English description** of what is happening
- **Real error message** from a real API response
- **Searchable tags**
- **Immediate fix** (what to do right now)
- **Prevention strategy** (what to build so it does not happen again)

The severity system deserves a note. I borrowed S1/S2/S3 from incident response rather than inventing a new scale. Anyone with ops experience already understands S1 means "production is down, page someone." That shared language matters when a developer is debugging at 2am.

---

## How the Classifier Works

The classifier tab takes a user's error message, log output, or plain-English description and identifies which failure pattern it matches.

**Flow:**
1. User pastes their error or describes the symptom in the input panel
2. The input is sent to Claude with a structured prompt that references the full taxonomy
3. Claude identifies the closest matching pattern and returns: pattern name, category, severity, confidence score, what is happening, immediate fix, and prevention strategy
4. The result panel updates in place with the diagnosis

Claude is used as the classification brain because the patterns are defined in natural language, not as rigid rule sets. A user might paste `Error 429` or they might write "my app slows down after a burst of requests and then recovers." Both describe rate limiting. A rule-based classifier would need two separate rules. Claude handles both with the same prompt.

**Current status:** The UI is fully built. The API connection is Phase 2. The classifier currently shows a hardcoded example result (Rate Limit Exceeded at 94% confidence) so reviewers can see the full intended experience.

---

## Hard Tradeoffs

**Static HTML vs. a full-stack app**

I shipped this as a single HTML file with no build step, no framework, no server. This was a deliberate choice for v1. It means: zero dependencies to break, instant load, anyone can fork and read the full source in one file, and deployment is just pushing to GitHub Pages. The tradeoff is that adding patterns requires editing HTML, which does not scale past 50 or so entries.

**Hardcoded data vs. a database**

Patterns are currently written directly into the HTML. The benefit: every pattern is reviewable in a single pull request diff. A contributor does not need to understand a database schema or an API to add a pattern. The cost: no dynamic filtering, no search that actually queries data, no easy way to add 200+ patterns without the file becoming unmaintainable.

**Breadth vs. depth**

I chose 52 patterns across 6 categories rather than 10 very detailed patterns in one category. The goal was a useful reference across the full surface area of LLM failures. The risk is that some patterns are not documented as deeply as they could be. I would trade breadth for depth in v2.

**Mock classifier vs. live API**

I built the UI and the full result experience before connecting a real API. This let me validate the design and user flow without managing API keys, CORS, or a backend in v1. The tradeoff is that the classifier is not functional yet, which reduces the "wow" factor for someone who finds the repo before v2 ships.

---

## What I Would Improve Next

1. **Connect the real Claude API.** The system prompt is already designed. This is the single highest-value next step.

2. **Move patterns to structured JSON or Markdown files.** One file per pattern, stored in `/patterns/`. Makes community contribution trivial and removes the HTML file size problem.

3. **Make search and filters actually work.** Right now the search bar and filter chips are UI only. Wire them to the data.

4. **Deploy to Vercel with a real domain.** GitHub Pages works but Vercel gives edge caching, analytics, and a cleaner URL.

5. **Add a real changelog.** The version pill currently says v0.4.2. That should link to an actual changelog showing what changed between versions.

---

## Contributing

Patterns are added by submitting a pull request. Each pattern needs: name, category, severity, a real error message, description, immediate fix, and prevention strategy. Open an issue first if you are unsure whether a pattern belongs.

---

## License

MIT. Use freely, credit appreciated.
