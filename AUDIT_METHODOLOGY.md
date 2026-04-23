# Archonics Audit Methodology v1.0

**Purpose:** Define a rigorous, repeatable process for auditing production agent systems across prompt, tool, context, and evaluation dimensions. This document is the specification that drives all Archonics audits — free scans, instant audits, and full audits all apply the same framework at different depths.

**Audience:** Internal use by the Archonics audit engine (Claude API calls) and human reviewers. Excerpts appear publicly on archonics.ai as a trust signal.

**Version:** 1.0 — April 22, 2026

---

## Core thesis

Most production agent failures are not model failures. They are **context engineering failures**. The model received ambiguous instructions, poorly described tools, bloated context, or no feedback loop that would have caught the problem before production. The audit methodology is designed to surface these failures systematically rather than anecdotally.

An audit examines four dimensions, scored independently and synthesized into a prioritized fix list.

---

## The four dimensions

### Dimension 1: System prompt analysis

A system prompt is a specification document written in natural language. Like any specification, it can be evaluated for clarity, consistency, completeness, and fitness for purpose.

**What we examine:**

1. **Role clarity.** Does the prompt establish a clear operating identity, or does it hedge across multiple roles that create behavioral ambiguity?
2. **Instruction conflicts.** Are there directives that contradict each other? (e.g., "be concise" and "always explain your reasoning in detail.") Agents resolve conflicts unpredictably, producing inconsistent behavior.
3. **Negative space.** What does the prompt *not* say? Missing guidance on error handling, edge cases, refusals, and tool-use priority is a frequent failure source.
4. **Priority structure.** When instructions conflict at runtime, which wins? Well-engineered prompts establish explicit priority; most don't.
5. **Token efficiency.** What fraction of the prompt is load-bearing? Dead weight in the system prompt increases cost on every turn and can dilute attention to the instructions that matter.
6. **Format specification.** Is output structure specified with enough precision that downstream parsing is reliable?
7. **Failure-mode coverage.** Does the prompt specify what to do when the agent cannot complete the task, lacks information, or encounters ambiguous input?

**Output:** Findings list with severity (critical / high / medium / low) and evidence (specific quoted passages or identified gaps).

### Dimension 2: Tool definition review

Tool descriptions are prompts in disguise. The model reads each tool's description and parameter schema to decide when to call it and with what arguments. Weak tool definitions cause tool-call hallucinations, parameter errors, and missed opportunities.

**What we examine:**

1. **Description quality.** Does the tool description start with a clear action verb and communicate when this tool should (and should not) be used? Descriptions that only describe *what* the tool does without describing *when to use it* produce reliable underuse or misuse.
2. **Parameter schema precision.** Are parameter types tight? (e.g., enum vs. free string, specific format vs. "any text.") Loose schemas invite invalid calls.
3. **Parameter description coverage.** Every parameter should have a description that communicates intent, acceptable values, and edge cases.
4. **Error response design.** What does the tool return when it fails? Models handle structured errors with actionable guidance far better than raw stack traces or generic "Error occurred."
5. **Tool set coherence.** Do multiple tools have overlapping purposes? Models split calls unpredictably when two tools could plausibly handle the same request.
6. **Tool set minimalism.** Is every tool earning its place? Each additional tool increases context cost and decision complexity.
7. **Discoverability.** If a tool should be used in a specific scenario, does its description explicitly name that scenario?

**Output:** Per-tool findings plus a tool-set-level assessment.

### Dimension 3: Context packing analysis

Context is the most expensive resource an agent has. Waste is ubiquitous. The audit examines what goes into the context window, when, and why.

**What we examine:**

1. **Content audit.** What is actually in context on a typical turn? System prompt, tool definitions, conversation history, retrieved documents, memory, reminders. We quantify each.
2. **Redundancy detection.** Is information repeated across system prompt, tool descriptions, and retrieved context? Redundancy produces attention dilution and wasted tokens.
3. **Freshness logic.** For retrieved or injected context (memory, RAG results, prior turns), what determines inclusion? Is inclusion logic tuned, or does it default to "include everything relevant"?
4. **Ordering.** Models weight recent and salient context more heavily. Is high-priority information positioned to survive attention competition?
5. **Truncation risk.** What happens as conversations grow long? Does the agent have a strategy for context overflow, or does it silently drop content?
6. **Cost per turn.** Dollar cost of a representative interaction, broken down by context category. Surfaces the highest-ROI reduction targets.
7. **Cache utilization.** For providers with prompt caching, is the static portion of the prompt positioned to maximize cache hits?

**Output:** Context inventory with cost breakdown, redundancy map, and a prioritized reduction plan.

### Dimension 4: Evaluation gap analysis

The final dimension examines what the team *knows* about their agent's behavior. An agent without evals is an agent whose quality is a rumor.

**What we examine:**

1. **Eval coverage.** What behaviors are tested? What behaviors are shipped but not tested?
2. **Regression protection.** When a prompt changes, what catches the downstream breakage? Most teams we audit have zero automated regression coverage on prompt changes.
3. **Tool-call accuracy.** Is there a test that the agent calls the right tool with the right arguments for a given scenario?
4. **Behavioral guardrails.** Are refusals, safety behaviors, and edge-case handling tested, or are they assumed to work?
5. **Production observability.** What is logged? Can the team reconstruct why a specific production call produced a specific output?
6. **Failure-case library.** Does the team collect the specific failures users have reported, and are those cases codified into tests?
7. **Eval-development feedback loop.** When a new failure is observed in production, how long until there's a test preventing its recurrence? For most teams, the answer is "never."

**Output:** Gap analysis mapping shipped behaviors against test coverage, with recommended high-ROI eval additions.

---

## Severity scoring

Every finding is assigned one of four severity levels:

- **Critical** — Active cause of production failures, or a failure mode one bad input away from firing. Fix immediately.
- **High** — Reliably produces degraded quality under normal operation. Fix this sprint.
- **Medium** — Measurable quality impact but not user-visible on typical traffic. Fix this quarter.
- **Low** — Efficiency or polish issue. Fix when convenient.

Severity is assigned based on expected *user-visible impact*, not on how intellectually interesting the issue is. A sloppy system prompt that nevertheless produces reliable outputs gets a lower severity than a clean prompt with a subtle instruction conflict that fires on 2% of real traffic.

---

## Prioritization framework

Every audit concludes with a ranked fix list. Ranking is a function of:

- **Severity** (above)
- **Effort to fix** (trivial / modest / significant / major)
- **Blast radius** (does the fix improve one behavior, or does it propagate across the system?)
- **Reversibility** (can we ship this fix and roll back cleanly if it regresses?)

The top items on the fix list are always high-severity, low-effort, high-blast-radius, reversible changes. Teams get disproportionate value from shipping these first.

---

## Deliverable structure

Every audit produces a written report with this structure:

1. **Executive summary** (1 page, non-technical leadership can read this)
2. **Context** (what system we audited, what we had access to, what we didn't)
3. **Findings by dimension** (prompt / tools / context / eval)
4. **Prioritized fix list** (top 10, ordered)
5. **Recommended eval additions** (if any tests would have caught the findings)
6. **Open questions** (what we need to know to deepen the analysis)

Report length scales with tier:

- **Free Scan** — 3 findings, single page, no executive summary
- **Instant Audit ($49)** — Full methodology applied programmatically, ~5-10 page PDF
- **Full Audit ($750)** — Human-reviewed, 15-25 page PDF, tuned to the team's context

---

## Privacy posture

Archonics audits process prompts, tool definitions, and sample interactions. This content may contain proprietary intellectual property, customer data, or security-sensitive information. Our handling posture:

- No prospect content is retained after audit delivery unless the customer explicitly requests retention for a follow-up engagement.
- No prospect content is used to train Archonics models or improve the methodology against that specific customer's systems.
- Anonymized patterns (e.g., "we observed 18/20 audited systems lack tool-call regression tests") may inform methodology evolution; specific content never does.
- Customers requesting higher assurance (NDAs, data processing agreements) are accommodated at the Full Audit tier.

This posture is stated publicly on archonics.ai and in the MCP server's tool descriptions, because customers handing over prompts need to know how we handle them before they do.

---

## Versioning

This methodology is versioned. Every audit report references the methodology version it was produced under. Improvements based on audit experience are tracked in a changelog appended to this document.
