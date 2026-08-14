# Some learnings for Claude Architect 

## Aug 14, 2026

# From Claude Call to Bounded Agent Architecture

## 1. Level

**Foundation — Four-week consolidation checkpoint**

This checkpoint covers the architectural judgment built across the first four weeks. There is **no new pattern today**. The goal is to decide which mechanism solves which problem, especially when several plausible Claude features appear in the same scenario.

---

## 2. Today’s concept

The first four weeks can be compressed into one architectural sequence:

> **Define the boundary → define the contract → execute safely → interpret the result → add orchestration only when the problem demands it.**

You should now be able to distinguish these layers:

| Problem you are solving                          | First architectural place to look                 |
| ------------------------------------------------ | ------------------------------------------------- |
| Claude sends invalid tool parameters             | Tool description / `input_schema`                 |
| Claude requests a client tool                    | Application executes it and returns `tool_result` |
| Application stops after a tool request           | Orchestration / `stop_reason` handling            |
| External capabilities need a standard connection | MCP                                               |
| Information versus action                        | Resource versus tool                              |
| Local capability versus shared remote service    | `stdio` versus Streamable HTTP                    |
| Sensitive capability needed only occasionally    | Least privilege / step-up access                  |
| Request belongs to one known category            | Routing                                           |
| Required subtasks are unknown beforehand         | Orchestrator–workers                              |
| First result needs criterion-driven refinement   | Evaluator–optimizer                               |
| Business path is predetermined                   | Workflow                                          |
| Path must adapt to environmental feedback        | Agent                                             |

Claude’s current client-tool lifecycle still follows the contract you have learned: Claude sees the tool definition and schema, returns structured `tool_use` blocks with `stop_reason: "tool_use"`, the application executes those client tools, and the results return in `tool_result` blocks. ([Claude Platform][1])

Anthropic’s agent guidance similarly emphasises choosing the **simplest architecture sufficient for the task** rather than treating increasing autonomy as inherently better. ([Anthropic][2])

---

## 3. Why an architect cares

Most production failures are easier to solve once you identify **which layer actually changed**.

Suppose quality falls after a deployment. Possible causes include a modified tool schema, new permissions, a routing error, an incorrect tool result, or an agent being given more autonomy. Immediately changing the system prompt or model ignores those distinctions.

The architect’s job is therefore to diagnose before redesigning.

A useful mental sequence is:

**Constraint → failure layer → simplest sufficient control → verify outcome**

This applies equally to exam scenarios. Attractive distractors frequently improve something—speed, sophistication, autonomy, model capability—while failing to address the actual constraint.

---

## 4. Architect’s lens

For today’s checkpoint, use three questions repeatedly:

1. **What exactly changed or failed?**
   Separate model reasoning from interface, integration, authorization, orchestration, and workflow problems.

2. **Who should control the decision?**
   Decide whether it belongs to deterministic application logic, Claude, the MCP host/server boundary, or a human reviewer.

3. **What is the least complex architecture that satisfies every hard constraint?**
   Do not introduce an agent, additional model call, remote service, or broad permission merely because it is technically possible.

---

# 5. Real-life integrated scenario

A retailer is building a Claude-based **customer-resolution assistant**.

It must handle:

* ordinary product questions;
* delivery investigations;
* refund requests;
* complicated complaints spanning several systems.

The company has these systems:

* product-policy documents;
* order-management APIs;
* shipping APIs;
* refund APIs;
* customer-support records.

Most requests fit one known category. However, unusual complaints may require investigation across several systems depending on what Claude discovers.

Refunds are financially consequential and require explicit customer-service authorization.

### A sensible architecture

The application first **routes** ordinary requests into predefined support paths because the categories are known. Anthropic identifies routing as a good fit when distinct categories can be classified accurately and benefit from specialised handling. ([Anthropic][2])

Product policies can be supplied as contextual **resources**, while live order queries and refund operations are **tools**. MCP’s current architecture continues to distinguish tools as executable functions, resources as context data, and prompts as reusable interaction templates. ([Model Context Protocol][3])

A centrally operated order-management MCP server used by many assistants naturally fits remote connectivity, while a genuinely workstation-local capability would more naturally fit `stdio`; MCP’s current architecture documents both local `stdio` and remote Streamable HTTP deployment patterns. ([Model Context Protocol][3])

For an ordinary delivery question:

```text
Route: delivery
      ↓
Claude calls shipment lookup
      ↓
Application executes client tool
      ↓
tool_result returned
      ↓
Claude interprets evidence
      ↓
Answer
```

For a refund, the architecture adds an authorization boundary before the financial action executes.

For a complex complaint such as:

> “My replacement was never delivered, the original order was charged twice, and my previous support case says I was promised a refund.”

the required investigation may not be predictable. Only that exceptional route might justify dynamic orchestration across order, shipping, payment, and support-history workers. Anthropic’s orchestrator–worker pattern is specifically intended for cases where the necessary subtasks depend on the particular input rather than being known beforehand. ([Anthropic][2])

Notice what the design does **not** do: it does not turn every product question into a multiagent investigation.

---

# 6. Checkpoint questions

These are **practice-derived questions**, not authentic Anthropic certification questions.

### Question 1

Claude correctly returns:

```text
stop_reason = "tool_use"
```

with a valid `lookup_shipment` request.

The application displays “Claude is checking your shipment” and ends the interaction without executing anything.

What is the **best first fix**?

**A.** Strengthen the tool description.
**B.** Increase `max_tokens`.
**C.** Execute the requested client tool, return the corresponding `tool_result`, and continue the conversation.
**D.** Replace the workflow with an autonomous agent.

### Spot the clue

> **“Valid `lookup_shipment` request”**

Claude selected and formatted the tool correctly. The failure occurred **after tool selection**.

### Answer reasoning

**Correct: C.**

For client tools, `stop_reason: "tool_use"` means Claude is waiting for the application to execute the requested operation and return its result. ([Claude Platform][1])

**Why A is tempting but weaker:** poor descriptions certainly can cause wrong tool selection or malformed requests. Neither happened here.

**What could change the answer:** if Claude repeatedly selected the wrong shipping tool or supplied incorrect parameters, then the tool definition and schema would become the first diagnostic target.

---

### Question 2 — **Select TWO**

The refund tool is used in fewer than 2% of support conversations. The security team wants to reduce the impact of compromised credentials and preserve clear user intent.

Which TWO choices are strongest?

**A.** Give every support session broad refund-write access at login.

**B.** Keep normal sessions read-oriented and obtain narrowly scoped authorization when a refund is actually requested.

**C.** Tell Claude in the system prompt that refund access is sensitive and rely on that instruction instead of server-side authorization.

**D.** Require an appropriate approval/control before the consequential refund executes.

**E.** Replace the refund tool with a resource.

### Spot the clue

> **“Fewer than 2%”**, **“compromised credentials”**, and **“clear user intent.”**

The privileged capability is uncommon and consequential.

### Answer reasoning

**Correct: B and D.**

Access to a capability and approval of a particular action are different controls. Granting narrow permissions only when necessary reduces exposure, while a consequential financial operation can still require explicit approval.

**Why A is tempting but weaker:** it improves convenience but widens the privilege boundary for the other 98% of interactions.

**What could change the answer:** if every user were a dedicated refund specialist whose principal workflow required refunds throughout a short controlled session, a narrowly scoped refund permission at session start might be reasonable. It still would not justify unrelated financial permissions.

---

### Question 3

The retailer has four stable ordinary support categories, but about 3% of complaints require unpredictable investigation across orders, shipping, payments, and previous support interactions.

Which architecture best follows the principles learned so far?

**A.** Use orchestrator–workers for every request because multiagent systems are more flexible.

**B.** Use one deterministic workflow containing every possible investigation for every customer.

**C.** Route ordinary requests to specialised workflows and allow the exceptional complex route to invoke bounded dynamic orchestration when the required investigation cannot be predetermined.

**D.** Generate several answers in parallel and choose the longest.

### Spot the clue

Two different problem structures coexist:

> **“Four stable ordinary support categories”**

but:

> **“unpredictable investigation”** for a small minority.

Do not force one orchestration pattern onto both.

### Answer reasoning

**Correct: C.**

Routing solves the predictable classification problem. Dynamic orchestration is reserved for the cases where decomposition itself requires reasoning. Anthropic explicitly differentiates routing from orchestrator–workers on this basis and recommends adding multi-step complexity only when simpler approaches fall short. ([Anthropic][2])

**Why A is the strongest distractor:** it would technically handle both simple and complex cases, but it pays the coordination, cost, latency, and operational complexity of dynamic delegation even where that flexibility provides no value.

**What could change the answer:** if nearly every support interaction became cross-system, unpredictable, and iterative, the balance could move toward agentic orchestration as the default rather than the exception.

---

## 7. Diagnostic pattern

If you remember only one diagnostic chain from the first month, use this:

> **Is the problem the input contract, execution, returned evidence, control flow, system boundary, permission boundary, task classification, dynamic decomposition, or output quality?**

Only after identifying that should you decide whether to change:

**schema → application logic → MCP design → authorization → routing/workflow → agent architecture → evaluation**

This prevents the common mistake of treating every AI-system problem as a prompting problem.

---

## 8. One-line architect rule

> **Put intelligence where uncertainty exists, deterministic controls where the rules are known, and additional autonomy only where it earns its cost and risk.**

## Aug 13, 2026

# Workflow or Agent? Set the Right Autonomy Boundary

## 1. Level

**Foundation**

Today’s lesson is the Thursday architectural trade-off for Week 4. The Foundation-level skill is choosing **how much decision-making Claude should own**. Professional depth later adds enterprise controls around that autonomy: policy, observability, financial limits, segregation of duties, incident handling, and lifecycle governance.

---

## 2. Today’s concept

Anthropic makes a useful architectural distinction between **workflows** and **agents**:

* A **workflow** uses predefined code paths to orchestrate Claude and tools.
* An **agent** lets Claude dynamically decide how to proceed and which tools or steps to use. ([Anthropic][1])

Think of the difference as **where the control flow lives**.

### Workflow

```text
Receive document
      ↓
Extract fields
      ↓
Validate fields
      ↓
Check database
      ↓
Generate summary
      ↓
Human approval
```

Your application determines the path.

Claude may perform intelligent work inside individual stages, but it does not decide that the entire process should suddenly branch into five new investigations.

### Agent

```text
Goal
 │
 ▼
Claude assesses situation
 │
 ├─ use tool
 ├─ inspect result
 ├─ revise plan
 ├─ use another tool
 ├─ recover from failure
 └─ stop when goal reached
```

Claude controls more of the path.

Anthropic recommends increasing this kind of complexity only when the problem actually requires it. Workflows offer predictability for well-defined tasks; agents are better suited to open-ended problems where the required sequence of steps cannot reasonably be hard-coded. Agentic designs typically trade additional cost and latency for greater flexibility. ([Anthropic][1])

The architect’s question is therefore not:

> **“Can Claude act autonomously?”**

It is:

> **“Which decisions genuinely need Claude’s judgment, and which should remain deterministic?”**

---

## 3. Why an architect cares

Autonomy creates value when the environment is uncertain.

It also creates additional failure modes.

As Claude gains control over more steps, you potentially gain:

* adaptation to unexpected conditions;
* recovery from tool failures;
* flexible investigation;
* ability to pursue long or variable tasks.

But you also increase:

* cost variability;
* latency variability;
* number of possible execution paths;
* difficulty of exhaustive testing;
* risk of compounding errors;
* importance of stopping conditions and guardrails.

Anthropic specifically recommends that agents obtain ground-truth feedback from their environment after actions, and notes that practical agent loops commonly include stopping conditions such as iteration limits. It also warns that autonomous systems have higher costs and greater potential for compounding errors. ([Anthropic][1])

This gives architects a useful principle:

> **Autonomy should be earned by uncertainty.**

If the business already knows the correct process, encode that process.

Do not make Claude rediscover it on every execution.

---

## 4. Architect’s lens

When deciding between a workflow and an agent, ask:

### 1. **Can the valid path be specified before execution?**

If the business process is:

```text
receive → validate → enrich → approve → submit
```

and those stages are mandatory, a workflow is usually preferable.

If instead the task is:

> “Determine why this deployment failed and fix the application.”

the required number and type of actions may depend entirely on what Claude discovers.

That is much more agent-shaped.

### 2. **What is the consequence of an unnecessary or incorrect action?**

Greater autonomy is easier to justify for reversible exploration:

* searching documentation;
* inspecting logs;
* running tests in a sandbox.

It deserves stronger constraints for consequential actions:

* deleting production records;
* approving payments;
* changing employee compensation;
* sending legally binding communications.

You can also combine approaches: allow agentic investigation but place consequential actions behind deterministic controls or human approval.

### 3. **Can success be observed from the environment?**

Agents work best when they can act and obtain useful feedback.

For example:

```text
modify code
    ↓
run test
    ↓
test fails
    ↓
inspect failure
    ↓
modify code again
```

Anthropic highlights coding as particularly suitable for agents because automated tests provide concrete environmental feedback and measurable outcomes. ([Anthropic][1])

If Claude cannot tell whether its actions improved the situation, increased autonomy may simply create a longer chain of unverified guesses.

---

## 5. Real-life example

A company wants Claude to help resolve failed enterprise integrations.

An integration may fail because of:

* expired credentials;
* schema changes;
* API throttling;
* malformed source data;
* an unavailable downstream system;
* mapping logic;
* a deployment regression.

The team considers two architectures.

### Option A — Fixed workflow

```text
Check credentials
      ↓
Check API status
      ↓
Check schema
      ↓
Check mappings
      ↓
Check logs
      ↓
Produce diagnosis
```

This is highly predictable.

But every incident performs every step, and new failure modes require modifying the workflow.

### Option B — Fully autonomous remediation agent

Claude receives:

> “Find and fix the integration.”

It may inspect logs, query APIs, modify mappings, alter configuration, retry transactions and deploy changes.

That is flexible—but far more autonomy than the business may initially need.

### Better architecture: bounded autonomy

The architect separates **diagnosis** from **consequential remediation**.

```text
Failure detected
      ↓
Agentic investigation
      │
      ├─ inspect logs
      ├─ query API health
      ├─ inspect schemas
      ├─ compare recent changes
      └─ test hypotheses
      ↓
Diagnosis + proposed remediation
      ↓
Deterministic policy checks
      ↓
Human approval for risky change
      ↓
Controlled execution
```

Claude has autonomy where dynamic reasoning creates value.

The surrounding system retains deterministic control where the cost of a mistake is higher.

This is often the more useful interpretation of “agent architecture”: **not maximum autonomy, but deliberately placed autonomy**.

Current Claude Platform documentation reflects the growing availability of long-running autonomous execution through Claude Managed Agents, including tool use, persistent sessions, interruption/steering, and sandboxed execution. Managed Agents is currently documented as a beta capability, so its existence should not be confused with an architectural requirement to use autonomous agents for every workload. ([Claude Platform][2])

---

## 6. Exam-style question

**Practice-derived scenario — not an authentic Anthropic certification question.**

A healthcare company uses Claude to process provider-registration applications.

The regulatory process requires exactly these stages:

1. validate mandatory fields;
2. verify the provider licence through an approved service;
3. check sanctions status;
4. generate an evidence summary;
5. send the case to a human credentialing officer for approval.

The sequence is mandatory and auditors require predictable evidence showing that every stage occurred.

Some architects propose replacing the pipeline with an autonomous agent that decides which checks are necessary for each application.

Which architecture is the **best fit**?

**A.** Use an autonomous agent because it can skip checks that appear unnecessary and therefore reduce latency.

**B.** Keep the mandatory stages in a deterministic workflow while using Claude within appropriate stages for tasks such as evidence interpretation and summarisation.

**C.** Use an orchestrator–worker system that dynamically decides which regulatory checks to delegate.

**D.** Ask Claude to perform the entire credentialing decision and periodically audit a sample of its decisions.

---

## 7. Spot the clue

The decisive phrases are:

> **“The sequence is mandatory”**

and

> **“auditors require predictable evidence showing that every stage occurred.”**

There is no architectural benefit in letting Claude decide whether mandatory stages should happen.

The business has already defined the control flow.

Use Claude for the portions requiring intelligence—not for rediscovering the process.

---

## 8. Answer reasoning

### Correct answer: **B**

A workflow fits because the required path is known in advance and every stage must occur. Anthropic recommends workflows where predictability and consistency matter and agents where flexibility and model-directed decision-making are genuinely necessary. ([Anthropic][1])

Claude can still add considerable value inside the workflow:

```text
Deterministic:
Licence lookup must occur

Intelligent:
Interpret the returned licence evidence
```

or:

```text
Deterministic:
Human approval must occur

Intelligent:
Prepare a concise evidence summary for the reviewer
```

This architecture preserves both AI capability and regulatory control.

### Why A is tempting but weaker

Skipping apparently unnecessary checks could reduce latency and cost.

But the scenario does not say:

> “Perform whichever checks appear relevant.”

It says the checks are **mandatory**.

The optimisation therefore conflicts with a harder constraint.

Exam scenarios frequently include attractive answers that optimise one dimension—cost, speed, sophistication—while violating a non-negotiable requirement.

### Why C is weaker

Orchestrator–workers are useful when the required subtasks cannot be predicted beforehand. Here the five stages are explicitly known. Dynamic decomposition adds uncertainty without solving a requirement. Anthropic distinguishes orchestrator–workers from predefined workflows precisely on this point. ([Anthropic][1])

### What additional fact could change the decision?

Suppose the requirement changed:

> Credentialing officers must investigate anomalies, but the necessary investigation varies by application and may involve licensing boards, corporate registries, sanctions sources, professional-history databases, and supporting documents.

Now the **mandatory baseline checks** could remain deterministic while unusual cases trigger an agentic investigation.

The resulting architecture might become:

```text
Mandatory workflow
      ↓
Anomaly?
   /      \
 no       yes
 |         |
continue   bounded investigation agent
             ↓
          return evidence
             ↓
       mandatory human approval
```

Notice that we did not replace the whole workflow with an agent.

We introduced autonomy only where the problem became unpredictable.

That is the architectural judgment being tested.

---

## 9. One-line architect rule

> **Hard-code what the business already knows; give Claude autonomy only where the path genuinely depends on what it discovers.**

---

## 10. Source basis

Anthropic’s official **Building effective agents** guidance defines workflows as predefined orchestration and agents as systems in which the model dynamically controls its process, recommends starting with the simplest sufficient architecture, and highlights the cost, latency, and error-compounding trade-offs of autonomy. ([Anthropic][1])

Current official **Claude Managed Agents** documentation describes Anthropic’s managed autonomous-agent runtime for long-running tasks, tool execution, persistent sessions, sandboxing, and agent steering; it currently remains a beta offering. ([Claude Platform][2])

The healthcare scenario and question are **practice-derived from those architectural principles**, not authentic Anthropic certification questions.

[1]: https://www.anthropic.com/engineering/building-effective-agents "Building Effective AI Agents \ Anthropic"
[2]: https://platform.claude.com/docs/en/managed-agents/overview "Claude Managed Agents overview - Claude Platform Docs"


## Aug 12, 2026

# Routing: Classify First, Then Send Work to the Right Path

## 1. Level

**Foundation**

## 2. Today’s concept

A **routing workflow** first classifies an input, then directs it to a specialised downstream path.

The architecture is simple:

```text
Incoming request
      |
      v
    Router
      |
  +---+---+---+
  |       |   |
  v       v   v
Path A  Path B Path C
```

Anthropic describes routing as useful when a broader task contains **distinct categories that are better handled separately** and the category can be identified accurately. Each route can then use its own prompt, tools, model configuration, workflow, or business logic. ([Anthropic][1])

The key architectural insight is:

> **Routing is appropriate when the destination varies, but the set of possible destinations is already understood.**

That makes routing fundamentally different from the **orchestrator–worker** pattern from Monday.

With routing:

```text
We know the possible paths.
We need to choose one.
```

With orchestrator–workers:

```text
We do not necessarily know what subtasks will be required.
The model must discover and delegate them.
```

And it differs from yesterday’s evaluator–optimizer pattern:

```text
Routing       → Which path should handle this?
Orchestration → What work needs to be created?
Evaluation    → Is this result good enough?
```

These distinctions are useful because they prevent architects from adding unnecessary agent autonomy.

---

## 3. Why an architect cares

Suppose one giant Claude prompt handles:

* invoice questions;
* contract review;
* supplier onboarding;
* security questionnaires;
* general administrative requests.

The prompt must now contain instructions, constraints, examples, and possibly tool descriptions for every category.

That can make the system harder to:

* optimise;
* evaluate;
* secure;
* operate;
* change independently.

A routing layer can separate those concerns.

For example:

```text
                      +-> Invoice workflow
Incoming document --->+-> Contract workflow
                      +-> Tax-document workflow
                      +-> General correspondence
```

Each path can now have only the context and capabilities relevant to its job.

Anthropic specifically notes that separation through routing allows specialised prompts to be optimised independently; trying to optimise one general prompt for very different input types can hurt performance on other types. ([Anthropic][1])

But routing itself has a failure mode:

> **A perfect downstream workflow cannot recover if the request was sent to the wrong route.**

Therefore, architects need to evaluate the router itself—not merely the quality of each downstream process.

---

## 4. Architect’s lens

When considering routing, ask three questions.

### 1. Are the categories genuinely distinct and known?

Routing works well when categories such as:

* invoice;
* purchase order;
* contract;
* tax form;

have meaningfully different downstream handling.

If every input ultimately needs essentially the same reasoning and tools, separate routes may add needless complexity.

### 2. How difficult is classification?

Do not automatically use Claude as the router.

If the request contains a reliable field such as:

```text
documentType = "invoice"
```

ordinary application logic should probably route it.

If instead the system must understand:

> “Please reimburse the duplicate amount charged on last month’s renewal.”

semantic classification may justify an LLM-based router.

Anthropic’s routing guidance explicitly allows classification to be performed either by an LLM or by a traditional classification model or algorithm when that is sufficient. ([Anthropic][1])

### 3. What happens when classification is uncertain?

A production router should not pretend every request fits neatly into a category.

Possible strategies include:

```text
high confidence      -> specialised route
low confidence       -> general workflow
sensitive ambiguity  -> human review
```

The architecture should match the consequences of misrouting.

Sending an ambiguous FAQ to the wrong knowledge prompt is inconvenient.

Sending a potential regulatory complaint into a low-risk general-support workflow may be materially worse.

---

## 5. Real-life example

A global procurement department receives thousands of documents through one mailbox.

Most fall into four categories:

1. supplier invoices;
2. contracts;
3. tax certificates;
4. general supplier correspondence.

The downstream processes differ substantially.

### Invoice route

Extract:

* supplier;
* invoice number;
* amount;
* purchase-order reference.

Then validate against the finance system.

### Contract route

Identify:

* parties;
* term;
* renewal;
* liability;
* governing law.

Then send higher-risk clauses for legal review.

### Tax-document route

Extract jurisdiction-specific tax identifiers and validate mandatory fields.

### Correspondence route

Summarise the message and identify the appropriate procurement owner.

A team proposes giving one autonomous agent every tool and asking:

> “Figure out what this document is and handle it.”

That could work, but it grants the agent more capability and responsibility than the problem requires.

The organisation already knows the four major categories.

A simpler architecture is:

```text
Document
   |
   v
Classifier
   |
   +-- invoice ------> Invoice pipeline
   |
   +-- contract -----> Contract pipeline
   |
   +-- tax ----------> Tax pipeline
   |
   +-- correspondence -> Correspondence pipeline
```

For obvious PDFs with structured metadata, deterministic rules might route the document without an LLM.

For ambiguous uploads, Claude could classify the document semantically.

The architect has separated two questions:

1. **What type of input is this?**
2. **How should that type be processed?**

That makes each part easier to test.

Imagine that contract extraction quality suddenly drops.

Without routing, you might investigate:

* the giant system prompt;
* tool selection;
* model behaviour;
* document parsing;
* contract instructions;
* unrelated invoice instructions.

With routing, diagnosis becomes clearer:

```text
Was it classified as a contract?
      |
     yes
      |
Did the contract workflow fail?
```

That is an operability advantage, not merely a prompting advantage.

---

## 6. Exam-style question

**Practice-derived scenario — not an authentic Anthropic certification question.**

A company operates an internal employee assistant.

Requests fall into four stable categories:

* payroll;
* benefits;
* IT support;
* facilities.

Each category has different approved knowledge sources, tools, access controls, and escalation procedures.

Classification accuracy is high because employees normally describe the issue clearly. The company wants the lowest-complexity architecture that preserves specialised handling.

Which design is the best fit?

**A.** Give one autonomous agent every payroll, HR, IT, and facilities tool and allow it to plan freely.

**B.** Use a routing step to classify the request and send it to the appropriate specialised workflow.

**C.** Launch four specialist agents for every request and ask an evaluator to choose the best answer.

**D.** Use an orchestrator to dynamically invent subtasks for every employee request.

---

## 7. Spot the clue

Two phrases dominate the scenario:

> **“Four stable categories”**

and:

> **“Each category has different … tools, access controls, and escalation procedures.”**

The possible destinations are already known.

The system needs to **choose among predefined paths**, not discover a new decomposition.

That is routing.

---

## 8. Answer reasoning

### Correct answer: **B**

Routing is designed for exactly this situation: classify an input into a known category and direct it to specialised downstream handling. Anthropic recommends the pattern where distinct categories benefit from separate prompts or processes and classification can be performed accurately. ([Anthropic][1])

It also supports the requirement for low architectural complexity. Anthropic’s broader agent guidance recommends starting with the simplest design that satisfies the problem rather than introducing autonomous agents unnecessarily. ([Anthropic][1])

### Why A is tempting but weaker

One capable agent with every tool appears flexible.

It could probably determine:

> “This is a payroll question.”

Then choose the payroll tools.

But now the agent must reason across:

* unrelated tools;
* multiple security domains;
* different escalation policies;
* different instructions.

The requirements already provide a clean separation.

Adding broad autonomy solves a problem the organisation does not have.

### Why D is weaker

An orchestrator–worker architecture becomes attractive when the required work cannot be determined in advance.

For example:

> “Investigate why this employee cannot complete onboarding.”

The investigation might dynamically require identity, HRIS, device-provisioning, background-check, and manager-approval analysis.

But the present scenario says requests belong to four established service categories. The task is classification, not dynamic decomposition.

### What additional fact could change the decision?

Suppose real production data shows that employee requests frequently span categories:

> “My promotion was approved, but my salary did not change, my new software access is missing, and my office location is still incorrect.”

Now one request may require coordinated payroll, HR, IT, and facilities work.

If the necessary combination varies significantly from case to case, a simple single-route classifier may become insufficient.

An orchestrator could then dynamically delegate several investigations and combine their results.

Notice what changed:

**Before:** choose one known destination.

**After:** discover and coordinate several required subtasks.

The architecture should change because the **problem structure changed**, not because multiagent architecture sounds more advanced.

---

## 9. One-line architect rule

> **Use routing when the possible paths are known and the main reasoning problem is choosing the correct one.**

## 10. Source basis

Official Anthropic engineering guidance defines routing as classifying input and directing it to specialised follow-up tasks, recommends it for distinct categories that can be classified reliably, and explicitly notes that the classifier may be an LLM or a traditional algorithm. ([Anthropic][1])

[1]: https://www.anthropic.com/engineering/building-effective-agents?utm_source=chatgpt.com "Building Effective AI Agents \ Anthropic"
[2]: https://www.anthropic.com/news/claude-partner-network?ref=techlaugh&utm_source=chatgpt.com "Anthropic invests $100 million into the Claude Partner Network \ Anthropic"


## Aug 11, 2026

## Evaluator–Optimizer: Improve Through Explicit Feedback Loops

## 1. Level **Foundation**

## 2. Today’s concept

Yesterday’s orchestrator–worker pattern solved a particular problem:

> **The system does not know in advance what subtasks will be needed.**

Today’s pattern solves a different problem:

> **The system can produce an answer, but the first answer may not meet a quality bar that can be clearly evaluated and improved.**

This is the **evaluator–optimizer workflow**.

The basic loop is:

```text
Task
 │
 ▼
Generator
 │
 ▼
Candidate output
 │
 ▼
Evaluator ──── Pass ───► Final output
 │
 Fail + feedback
 │
 ▼
Generator revises
 │
 └──────────────► Evaluate again
```

One model call produces the candidate. A separate evaluation step examines that candidate against explicit criteria and provides actionable feedback. The generator then uses that feedback to revise the result. Anthropic describes this pattern as especially useful when evaluation criteria are clear and iterative refinement measurably improves the result. ([Anthropic][1])

The important distinction is:

> **An evaluator should identify what must change, not merely say that the answer is bad.**

Compare:

```text
Score: 6/10.
```

with:

```text
Revision required:
- Section 3 makes a conclusion unsupported by the supplied evidence.
- The response omits the contractual notice period.
- The recommendation does not address the customer's stated cost constraint.
```

The second result gives the optimizer something useful to act on.

---

## 3. Why an architect cares

A common reaction to inconsistent AI output is:

> “Use a better model.”

Sometimes that is appropriate.

But sometimes the task is inherently iterative.

Humans do not normally produce complex architecture reviews, compliance analyses or executive reports perfectly on their first attempt. They:

1. create a draft;
2. compare it against requirements;
3. identify weaknesses;
4. revise;
5. stop when the quality threshold is met.

An evaluator–optimizer workflow deliberately implements that feedback cycle.

Anthropic distinguishes **workflows**, where model calls and tools follow predefined orchestration paths, from more autonomous agents that dynamically determine how to proceed. Evaluator–optimizer is therefore a workflow pattern: the feedback loop itself is architected in advance even though the critique and revision content are model-generated. ([Anthropic][1])

This can improve quality, but it introduces costs:

* another model invocation;
* additional latency;
* potentially several iterations;
* greater evaluation complexity.

Anthropic’s broader guidance is to start with the simplest solution and add agentic complexity only when the resulting quality improvement justifies the latency and cost trade-off. ([Anthropic][1])

So the architectural decision is not:

> “Would critique make the output nicer?”

It is:

> **“Is the quality problem important enough, measurable enough and improvable enough to justify an iterative loop?”**

---

## 4. Architect’s lens

Before adding evaluator–optimizer, ask three questions.

### 1. Can I describe the quality criteria clearly?

Good criteria might include:

* every recommendation must cite supplied evidence;
* all mandatory requirements must be addressed;
* contradictions between sources must be explicitly identified;
* conclusions must distinguish fact from inference;
* the response must consider cost, latency and security constraints.

If the evaluator itself cannot reliably determine what “good” means, adding another LLM call may simply create another opinion.

### 2. Can feedback actually improve the candidate?

The generator must be able to act on the critique.

A useful evaluator might say:

> “The recommendation assumes EU customer data can leave the region, but the requirements state that data residency is mandatory.”

The generator can fix that.

A vague evaluator saying:

> “Make the architecture more enterprise-ready”

provides little improvement signal.

### 3. Could a deterministic check solve the problem more cheaply?

Do not use an evaluator LLM to check something ordinary code can guarantee.

For example:

```text
Must contain exactly 10 records
```

→ validate with code.

```text
JSON must match this schema
```

→ use schema validation.

```text
All four stakeholder concerns are addressed coherently and the recommendation balances them
```

→ model-based evaluation may be useful.

The evaluator should be reserved for judgments that genuinely require semantic reasoning.

---

## 5. Real-life example

A bank uses Claude to draft investigation summaries for suspicious transaction cases.

The model receives:

* transaction history;
* analyst notes;
* customer information;
* relevant policy excerpts.

The first-generation summary is often readable, but reviewers identify recurring problems:

* an important transaction is occasionally omitted;
* facts and hypotheses are sometimes mixed together;
* conclusions occasionally lack supporting evidence;
* mitigating evidence may receive less attention than suspicious evidence.

Simply asking Claude:

> “Write a better investigation report”

does not clearly address these weaknesses.

The bank therefore defines an evaluation rubric.

### Generator

Produces the investigation summary.

### Evaluator

Checks the draft against four criteria:

1. **Evidence coverage** — are material transactions and analyst findings represented?
2. **Grounding** — is each significant conclusion supported by supplied evidence?
3. **Fact/inference separation** — are hypotheses clearly distinguished from known facts?
4. **Balanced analysis** — does the report include relevant mitigating as well as adverse evidence?

Suppose the evaluator returns:

```text
FAIL

Grounding:
Paragraph 4 states that the customer deliberately structured
payments to avoid monitoring thresholds. The evidence shows
payments below the threshold but does not establish intent.

Coverage:
The draft omits the customer's documented explanation for
the April 14 transfer.

Required revision:
Reframe the intent statement as a hypothesis and include the
customer explanation before presenting the risk conclusion.
```

Claude then revises the draft.

If the second evaluation passes the defined threshold, the document proceeds to the human investigator.

Notice what this architecture **does not** do.

The evaluator does not determine whether the customer committed financial crime.

It evaluates the **quality of the AI-generated analysis** against defined criteria.

The qualified human remains responsible for the consequential decision.

This separation gives the feedback loop a narrow, defensible purpose.

---

## 6. Exam-style question

**Practice-derived scenario — not an authentic Anthropic certification question.**

A company uses Claude to produce architecture recommendations from customer requirements.

The recommendations are usually strong, but reviewers repeatedly find that the first draft:

* overlooks one or two stated constraints;
* occasionally recommends technology without explaining the trade-off;
* sometimes makes assumptions without labelling them.

The requirements differ for every customer, so these issues cannot all be addressed with fixed keyword checks.

The company wants to improve quality without requiring a human architect to rewrite every draft.

Which approach is the best fit?

**A.** Generate three architecture recommendations in parallel and select the longest response.

**B.** Add an evaluator step that checks the draft against explicit criteria for requirement coverage, trade-off reasoning and assumption labelling, then provide targeted feedback for revision.

**C.** Give the generator a much larger `max_tokens` value so it can produce more detail.

**D.** Use an orchestrator to create several specialist workers regardless of the complexity of the request.

---

## 7. Spot the clue

The important phrase is:

> **“Reviewers repeatedly find that the first draft…”**

The system can already perform the task.

The problem is **quality after initial generation**.

Then look at:

> **“requirement coverage, trade-off reasoning and assumptions”**

These are semantic quality criteria that can be articulated and evaluated.

That combination strongly suggests an evaluator–optimizer loop.

---

## 8. Answer reasoning

### Correct answer: **B**

The problem is not primarily task decomposition. It is iterative quality improvement.

Anthropic’s evaluator–optimizer pattern is intended for situations where:

* there are clear evaluation criteria;
* feedback can be articulated;
* the generator can use that feedback to measurably improve its response. ([Anthropic][1])

The evaluator can therefore inspect the architecture recommendation against explicit criteria such as:

```text
✓ Every mandatory constraint addressed
✓ Major trade-offs explained
✓ Assumptions labelled
✓ Recommendation connected to evidence
```

A failed criterion should produce **specific revision guidance**, after which the generator tries again.

### Why A is tempting but weaker

Generating several answers can be useful when diversity or voting improves confidence. Anthropic treats this as a form of parallelisation. ([Anthropic][1])

But the scenario already identifies recurring, known quality defects.

Three drafts may reproduce the same defects.

More importantly, “select the longest” is unrelated to the actual success criteria.

If the requirement instead said:

> “There are several defensible architecture approaches and the company wants genuinely independent alternatives before choosing one,”

parallel generation could be useful.

### Why D is weaker

An orchestrator–worker architecture is valuable when the system must dynamically determine which subtasks need to be delegated. ([Anthropic][1])

Nothing here says decomposition is the problem.

Introducing specialised workers would add coordination cost without directly targeting the identified failure mode.

### What additional fact could change the decision?

Suppose further analysis shows that almost every failure is simply:

> “One mandatory requirement ID is missing from the output.”

If every requirement has an ID such as:

```text
REQ-001
REQ-002
REQ-003
```

and the output must reference every applicable ID, ordinary code could verify coverage deterministically.

Then the better architecture may be:

```text
Generate
   ↓
Programmatic requirement-ID validation
   ↓
Pass / regenerate
```

Using another LLM merely to discover that `REQ-017` is absent would add unnecessary cost and uncertainty.

This leads to an important diagnostic hierarchy:

> **Deterministic validation first; semantic evaluator when judgment is genuinely required.**

---

## 9. One-line architect rule

> **Use evaluator–optimizer when quality can be clearly judged, feedback can drive a better revision, and deterministic validation is not sufficient.**

---

## 10. Source basis

* Official Anthropic engineering guidance, **Building effective agents**, defining evaluator–optimizer as a generator/evaluator feedback loop and recommending it when evaluation criteria are clear and iterative refinement produces measurable improvement. ([Anthropic][1])
* The same official guidance distinguishes predefined workflows from dynamically directed agents and recommends increasing architectural complexity only when the benefit justifies the cost and latency trade-off. ([Anthropic][1])
* Anthropic notes that the surrounding agent tooling landscape has evolved since the original December 2024 publication, but continues to present these composable workflow patterns as architectural guidance while directing readers to newer agent tooling for current implementations. ([Anthropic][1])
* Exam scenario is **practice-derived from official architectural patterns** and is not presented as an authentic certification question.

[1]: https://www.anthropic.com/research/building-effective-agents?_bhlid=137363142c705ed9914261e1aa9fbe9b57c94ed2 "Building Effective AI Agents \ Anthropic"


## Aug 10, 2026

# Delegate Only When the Task Must Be Decomposed Dynamically

## 1. Level - **Foundation**

## 2. Today’s concept

An **orchestrator–worker architecture** uses one central model to decide how a complex task should be divided, delegates the resulting subtasks to worker models or agents, and then combines their results.

The important word is **dynamic**.

The orchestrator does not merely launch a fixed set of predefined jobs. It looks at the specific request and decides:

* what subtasks exist;
* which workers should handle them;
* whether some work can run in parallel;
* when enough evidence has been collected;
* how the separate results should be synthesised.

Anthropic distinguishes this pattern from ordinary parallelisation. In parallelisation, the subtasks are known in advance. With orchestrator–workers, the subtasks are determined from the input because the required decomposition cannot reliably be predicted beforehand.

Consider two research requests.

**Request A**

> Compare these three named contracts for five predefined clauses.

The decomposition is predictable:

```text
Contract 1 analysis
Contract 2 analysis
Contract 3 analysis
        ↓
      combine
```

You probably do **not** need an LLM to decide what work exists.

Now consider:

> Investigate why our European customer churn increased this quarter and identify the most plausible causes.

The required investigation might involve:

```text
             Orchestrator
                  │
        ┌─────────┼──────────┐
        ▼         ▼          ▼
   CRM analysis  Support   Pricing review
                    │
                    ▼
             New hypothesis
                    │
                    ▼
          Product usage analysis
                    │
                    ▼
                Synthesis
```

The subtasks emerge as evidence is discovered.

That is where orchestrator–worker architecture becomes valuable.

The Foundation-level judgment to learn is:

> **Use orchestration when decomposition itself requires reasoning.**

---

### Current Claude platform note

Anthropic now also exposes multiagent orchestration through the **Claude Managed Agents** platform. Its current beta capability supports a coordinator delegating to agents with separate context threads and configurations. This is a product implementation of multiagent patterns; the architectural principle is broader than that particular API.

---

## 3. Why an architect cares

It is easy to assume:

> “If one Claude is useful, several Claudes must be better.”

That is poor architecture.

Every additional worker can introduce:

* extra model calls and cost;
* more latency;
* duplicated work;
* context-transfer overhead;
* inconsistent conclusions;
* more failure paths;
* additional observability and evaluation requirements.

Anthropic’s agent guidance recommends maintaining simplicity and adding complexity only when it creates measurable value. Agents are particularly useful where the model must obtain feedback from the environment, adapt its approach and continue until a meaningful success condition is reached.

Therefore, the architect is not asking:

> “Can we use multiple agents?”

The architect is asking:

> **“Does dynamic delegation solve a problem that a simpler architecture cannot solve sufficiently?”**

This distinction matters in production and in scenario questions.

A single model with good tools may outperform a poorly designed multiagent system simply because it has:

* less coordination overhead;
* one reasoning context;
* fewer points of failure;
* lower cost.

Use the simplest sufficient architecture first.

---

## 4. Architect’s lens

When considering orchestrator–workers, ask three questions:

### 1. Can I determine the subtasks before execution?

If yes, use deterministic sequencing or parallelisation where possible.

If the task itself determines which investigations are needed, an orchestrator may add value.

### 2. Do the workers gain anything from separation?

Useful reasons include:

* different tools;
* specialised instructions;
* isolated context;
* genuinely independent work;
* parallel execution.

If every worker receives the same prompt, same context and same tools, splitting the work may add little value.

### 3. Can the orchestrator verify and synthesise the outputs?

Delegation without an integration step simply produces multiple answers.

The architecture needs a clear mechanism for deciding:

* which evidence matters;
* whether workers disagree;
* whether further investigation is needed;
* what constitutes completion.

---

## 5. Real-life example

A pharmaceutical company uses Claude to assist compliance analysts investigating potential adverse-event reports.

Incoming reports vary enormously.

One message might say:

> “Patient experienced nausea after Product X.”

Another could contain:

* several medications;
* an unclear product name;
* foreign-language correspondence;
* clinical notes;
* a potential hospitalization;
* references to earlier support cases.

A fixed workflow containing ten analysis steps would execute unnecessary work for simple reports.

A single giant prompt could also force one context to contain every possible instruction and capability.

Instead, the company introduces an orchestrator.

```text
Incoming report
      │
      ▼
Classification / planning agent
      │
      ├── Product identification worker
      ├── Medical-event extraction worker
      ├── Translation worker
      ├── Historical-case search worker
      └── Regulatory-rule worker
                 │
                 ▼
          Orchestrator synthesis
                 │
                 ▼
            Human reviewer
```

The orchestrator first examines the case.

For a simple report, it may require only medical-event extraction and product identification.

For a complicated report, it may delegate additional investigations.

The workers have bounded responsibilities and can have different tools or context relevant to their job. Current Anthropic multiagent guidance explicitly identifies **parallelisation**, **specialisation**, and **escalation** as useful delegation patterns for complex work.

The final regulatory decision is still reviewed by a qualified human.

The important architectural benefit is **not merely having multiple agents**.

It is that computational effort expands according to the complexity of the case.

---

## 6. Exam-style question

**Practice-derived scenario — not an authentic Anthropic certification question.**

A software company uses Claude to investigate production incidents.

Every incident is different. Depending on the initial evidence, the investigation may require:

* application-log analysis;
* database analysis;
* source-code inspection;
* recent-deployment review;
* infrastructure diagnostics.

The team cannot know beforehand which investigations will be relevant. Most incidents require only two or three of them.

Which architecture is the best fit?

**A.** Always execute all five diagnostic processes in parallel and combine their outputs.

**B.** Use an orchestrator that examines the incident, dynamically delegates relevant investigations to specialised workers, and synthesises the findings.

**C.** Put all logs, source code, database information and infrastructure data into one prompt so Claude has everything available immediately.

**D.** Create five independent agents and show all five final answers directly to the engineer.

---

## 7. Spot the clue

The decisive phrase is:

> **“The team cannot know beforehand which investigations will be relevant.”**

That tells you the decomposition is **input-dependent**.

Also notice:

> **“Most incidents require only two or three.”**

Running every possible investigation would waste cost and potentially increase latency.

These clues point toward **dynamic orchestration**, not fixed parallelisation.

---

## 8. Answer reasoning

### Correct answer: **B**

An orchestrator–worker design fits because the central agent can inspect the incident, determine which specialised investigations are warranted, delegate those subtasks and then synthesise the evidence.

Anthropic specifically identifies orchestrator–workers as useful when the required subtasks cannot be predicted in advance. This differs from ordinary parallelisation, where the set of subtasks is predefined.

### Why A is tempting

Parallel execution sounds attractive because:

* investigations could finish quickly;
* workers are independent;
* all possible evidence becomes available.

But the scenario explicitly states that many investigations are unnecessary.

Executing all five every time would increase model and tool consumption and could also create irrelevant evidence that the final stage must reconcile.

Parallelisation would be stronger if the problem instead said:

> “Every incident must always undergo the same five mandatory diagnostic checks.”

Then the tasks are known in advance and an LLM orchestrator may add unnecessary reasoning overhead.

### Why C is weaker

A huge single context may appear simpler, but it forces the model to absorb information that may not be relevant to the current incident.

It also couples multiple systems and diagnostic domains into one reasoning step rather than allowing targeted evidence gathering.

A single-agent solution could still be appropriate if that agent can efficiently call the necessary tools itself. Multiagent orchestration should not be introduced merely because many data sources exist.

### What additional fact could change the decision?

Suppose analysis shows that:

* 95% of incidents need only one diagnostic tool;
* one Claude agent chooses that tool reliably;
* worker delegation adds significant latency;
* there is no meaningful benefit from separate context or specialist instructions.

Then the simpler solution would probably be:

```text
Single agent
   ↓
select diagnostic tool
   ↓
observe result
   ↓
continue if necessary
```

The existence of multiple possible capabilities does **not** automatically justify multiple agents.

The additional architecture is justified only when dynamic decomposition or specialist delegation delivers enough quality, latency, context or operational benefit to outweigh the coordination cost.

---

## 9. One-line architect rule

> **Use orchestrator–workers when deciding what work needs to be done is itself part of the reasoning problem.**

---

## 10. Source basis

* Official Anthropic engineering guidance, **Building Effective AI Agents**, including the orchestrator–worker pattern and its distinction from fixed parallelisation.
* Current Claude Platform documentation on **multiagent orchestration**, including coordinator-led delegation, context-isolated agents, parallelisation, specialisation and escalation.
* Current Claude Platform documentation describing reusable Managed Agent configurations; the Managed Agents API remains a beta surface requiring the documented beta header.
* Exam scenario is **practice-derived from official architectural patterns** and is not presented as an authentic certification question.


## Aug 05, 2026

Today’s lesson completes the MCP foundation by connecting **transport choice** to deployment topology, security and operability.

# Week 3, Session 13 — MCP Transport Choice: Local `stdio` or Remote Streamable HTTP?

## 1. Level

**Foundation**

---

## 2. Today’s concept

An MCP transport determines **how an MCP client and server exchange protocol messages**.

It does not change what a tool, resource or prompt means. The same MCP capabilities can theoretically be exposed over different transports; the transport controls matters such as:

* where the server runs;
* how messages are delivered;
* who manages the server process;
* how access is authenticated;
* how failures and cancellation are handled.

The current MCP specification defines two standard transports:

| Transport           | Basic deployment model                                                                                     |
| ------------------- | ---------------------------------------------------------------------------------------------------------- |
| **`stdio`**         | The client launches a local MCP server as a subprocess and communicates through standard input and output. |
| **Streamable HTTP** | The MCP server runs as an independent network service and receives requests at an HTTP endpoint.           |

MCP describes the transport as a **binding**: it carries JSON-RPC messages but does not redefine their meaning. ([Model Context Protocol][1])

### `stdio`: local and client-managed

With `stdio`, the MCP client starts the server process.

The client writes MCP messages to the server’s standard input, and the server writes valid MCP messages to standard output. The server process usually exists for the lifetime of that client connection. ([Model Context Protocol][2])

This is often appropriate when:

* the capability runs on the user’s computer;
* it needs access to local files or development tools;
* only one host needs that process;
* the host should control startup and shutdown;
* opening a network endpoint would add no value.

Typical examples include local source-code tools, filesystem access, desktop automation and developer utilities.

### Streamable HTTP: remote and independently operated

With Streamable HTTP, the MCP server runs independently from the client. It exposes an HTTP endpoint that can receive requests from authorised clients.

Under the current specification, each request is sent using HTTP POST. The response may be a single JSON object or a request-scoped Server-Sent Events stream when incremental messages are needed. ([Model Context Protocol][3])

This is often appropriate when:

* the capability is hosted centrally;
* many users or MCP hosts must share it;
* the service needs independent deployment and scaling;
* access must be managed using network-level identity and authorisation;
* central monitoring, gateways or rate limits are required.

Typical examples include enterprise CRM access, centrally governed knowledge services, production ticketing systems and shared compliance platforms.

The core rule is:

> **Choose the transport from the deployment and trust boundary—not from the tool’s business name.**

---

### Current protocol update

The latest MCP specification is dated **July 28, 2026**.

In this version, the protocol core is stateless and Streamable HTTP no longer uses the older protocol-level session model or the previous GET stream endpoint. The older **HTTP+SSE transport** is deprecated; new implementations should use Streamable HTTP rather than adopting HTTP+SSE. ([Model Context Protocol][3])

Older tutorials may therefore show connection flows that do not match the current protocol.

---

## 3. Why an architect cares

Transport selection affects far more than connectivity.

### Security boundary

A local `stdio` server avoids exposing an MCP endpoint over the network. However, that does not automatically make it safe.

A local server may execute with the same operating-system privileges as its host and could access sensitive files, credentials or commands. Official MCP security guidance recommends sandboxing local servers, granting minimal privileges and making additional access explicit. It specifically identifies `stdio` as a way for a local server to limit access to the launching MCP client. ([Model Context Protocol][4])

A remote HTTP server introduces a different security model. It normally requires:

* transport security;
* authenticated requests;
* access-token validation;
* network controls;
* protection against unauthorised callers.

The MCP authorisation specification applies to HTTP-based transports and uses OAuth-oriented mechanisms when protocol-level authorisation is implemented. `stdio` implementations are instead expected to obtain credentials through their local environment rather than apply the HTTP authorisation flow. ([Model Context Protocol][5])

### Operational ownership

With `stdio`, the host typically owns the process lifecycle:

```text
Start host
   ↓
Launch MCP server
   ↓
Exchange messages
   ↓
Stop server when connection ends
```

With Streamable HTTP, the service operator owns the server lifecycle:

```text
Deploy shared service
   ↓
Monitor and scale it independently
   ↓
Accept authorised client requests
```

This changes who handles upgrades, logging, capacity, availability and incident response.

### Reuse and scale

A local subprocess is simple for one host and one user, but it is usually not the natural choice for a centrally managed service shared by hundreds of clients.

Conversely, deploying a remote service for a small local utility may add unnecessary infrastructure, authentication, network latency and operational overhead.

The architect should choose the **simplest transport that satisfies the real deployment requirement**.

---

## 4. Architect’s lens

Before selecting a transport, ask:

1. **Where must the capability run?**
   If it needs the user’s local filesystem or developer environment, `stdio` may be natural. If it accesses a centrally managed enterprise platform, Streamable HTTP is more likely.

2. **Who should own the server lifecycle?**
   Decide whether the MCP host should start and stop the server or whether an operations team should deploy, monitor and scale it independently.

3. **Who needs to connect, and how will access be controlled?**
   One local host and one local process suggest `stdio`. Multiple distributed hosts, central identity and governed network access suggest Streamable HTTP.

---

## 5. Real-life example

A software company is building a Claude-based engineering assistant.

It needs two MCP integrations.

### Local repository server

The first server can:

* read the developer’s current working tree;
* inspect uncommitted changes;
* run approved local static-analysis commands;
* retrieve local build output.

This capability exists inside each developer’s workstation. The data may not yet have been pushed to a central repository.

A suitable deployment is:

```text
Engineering Assistant
        |
        |— stdio → Local Repository MCP Server
```

The assistant launches the server as a subprocess. The organisation can sandbox it and restrict it to the selected repository directory.

Deploying this capability as a remote HTTP service would be awkward: the remote service would not naturally have access to each developer’s uncommitted local files.

### Shared release-governance server

The second integration can:

* retrieve central release policies;
* check deployment approvals;
* examine organisation-wide change windows;
* submit an authorised release request.

This service is used by developers, release managers and several AI applications.

A suitable deployment is:

```text
Multiple approved MCP hosts
        |
        |— Streamable HTTP → Release Governance MCP Server
```

The service is independently deployed, centrally monitored and protected through enterprise authentication and authorisation.

The two servers may expose equally important MCP tools, but their deployment boundaries are different. Therefore, their appropriate transports are different.

---

## 6. Exam-style question

This is a **practice-derived scenario**, not an authentic certification question.

A development team creates an MCP server that reads uncommitted source-code changes and runs a local compiler.

Requirements include:

* The server must access files on the developer’s workstation.
* The MCP host should start and stop it automatically.
* The server is used only by that local host.
* The team wants to avoid opening an unnecessary network port.

Which transport is the best fit?

**A.** Streamable HTTP behind a public API gateway

**B.** The deprecated HTTP+SSE transport

**C.** `stdio`, with the client launching the server as a restricted local subprocess

**D.** Streamable HTTP with no authentication because the server runs on localhost

---

## 7. Spot the clue

The decisive phrases are:

> **“Access files on the developer’s workstation”**

> **“The MCP host should start and stop it”**

> **“Used only by that local host”**

These describe a **local, client-managed process boundary**.

The requirement to avoid an unnecessary network port reinforces the same conclusion.

---

## 8. Answer reasoning

**Correct answer: C**

`stdio` is designed for a client-launched subprocess communicating through standard input and output. It fits a capability whose lifecycle is tied to the host and whose required data exists locally. It also avoids exposing an HTTP endpoint solely to connect two processes on the same workstation. ([Model Context Protocol][2])

The server should still be treated as executable code with access to the user’s environment. Appropriate protections include restricted filesystem scope, minimal privileges, explicit consent for sensitive commands and sandboxing where practical. ([Model Context Protocol][4])

---

## 9. One-line architect rule

> **Use `stdio` for client-managed local capabilities and Streamable HTTP for independently operated, network-accessible shared services.**

---

## 10. Source basis

* Official MCP transport overview defining transports as bindings and identifying `stdio` and Streamable HTTP. ([Model Context Protocol][1])
* Official MCP `stdio` transport specification. ([Model Context Protocol][2])
* Official MCP Streamable HTTP specification and current compatibility guidance. ([Model Context Protocol][3])
* Official MCP authorisation and security guidance. ([Model Context Protocol][4])
* Practice-derived exam scenario based on official architectural patterns; it is not an authentic certification question.

[1]: https://modelcontextprotocol.io/specification/2026-07-28/basic/transports?utm_source=chatgpt.com "Overview - Model Context Protocol"
[2]: https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/stdio?utm_source=chatgpt.com "stdio - Model Context Protocol"
[3]: https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http?utm_source=chatgpt.com "Streamable HTTP - Model Context Protocol"
[4]: https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices?utm_source=chatgpt.com "Security Best Practices - Model Context Protocol"
[5]: https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization?utm_source=chatgpt.com "Authorization - Model Context Protocol"


## Aug 04, 2026

## 1. Level

**Foundation**

---

## 2. Today’s concept

An MCP server can expose three core building blocks:

* **Tools** — operations the model can request
* **Resources** — information the application can provide as context
* **Prompts** — reusable workflows that the user explicitly selects

The key architectural skill is not memorising the three names. It is deciding **which primitive correctly represents a capability**.

A practical test is:

| Need                                          | MCP primitive | Typical controller |
| --------------------------------------------- | ------------- | ------------------ |
| Claude must **do something**                  | Tool          | Model              |
| Claude needs to **know something**            | Resource      | Application        |
| The user wants to **start a known procedure** | Prompt        | User               |

The current MCP documentation describes tools as model-controlled, resources as application-driven and prompts as user-controlled. These are intended interaction patterns rather than rigid user-interface requirements; an MCP host still decides how each capability appears in its product.

### Tools: perform an operation

A tool represents an executable capability with typed inputs and outputs.

Examples include:

* searching a ticketing system;
* submitting a leave request;
* checking a supplier against a sanctions service;
* updating a calendar;
* modifying a file.

The model can discover available tools and request them when relevant. The host may still require approval before execution, particularly for consequential actions.

### Resources: provide contextual data

A resource represents information that the application can retrieve and place into model context.

Examples include:

* a policy document;
* a database schema;
* API documentation;
* a project file;
* a customer record displayed for review.

Resources have URIs and may contain text or binary data. The host application decides whether to expose them through a picker, retrieve them automatically, search them first or include only selected portions.

### Prompts: start a reusable workflow

An MCP prompt is a server-published, parameterised interaction template.

Examples include:

* “Review this supplier for onboarding”
* “Summarise today’s meetings”
* “Prepare a pull-request review”
* “Investigate this support escalation”

Prompts are intended to be explicitly selected by the user, perhaps through a slash command, command palette or button. A prompt can guide the model in using relevant resources and tools, but it does not itself execute those operations.

A useful summary is:

> **Resources supply the evidence, prompts frame the task, and tools perform the work.**

---

## 3. Why an architect cares

Choosing the wrong primitive creates unnecessary complexity.

Suppose a team exposes a 200-page employee handbook through a tool named:

```text
get_employee_handbook
```

Claude must now decide whether to call that tool, even though selecting and loading appropriate documents may be better controlled by the application.

Conversely, suppose a team represents this action as a resource:

```text
payroll://change-bank-account
```

That hides a consequential state change behind something that appears to be contextual data.

Poor primitive selection can cause:

* unclear consent boundaries;
* accidental side effects;
* excessive model autonomy;
* unnecessary tool calls;
* poor discoverability;
* confusion about who initiates a workflow;
* difficult audit and security reviews.

Good primitive selection also supports the **simplest sufficient architecture**. Not every piece of data needs a callable tool, and not every repeated task needs custom orchestration code.

---

## 4. Architect’s lens

When deciding among the three primitives, ask:

1. **Is the capability informational or operational?**
   Information normally points toward a resource; an external query, computation or state change normally points toward a tool.

2. **Who should initiate it?**
   Use a prompt when the user should deliberately start a named workflow. Use a tool when the model should decide whether an operation is needed during reasoning.

3. **Where should context selection occur?**
   If the application should control which documents or records enter the context window, expose them as resources rather than forcing the model to retrieve everything through tools.

---

## 5. Real-life example

A company is building a supplier-onboarding assistant.

The assistant must:

* read the organisation’s due-diligence policy;
* inspect the supplier’s submitted documents;
* check sanctions and adverse-media services;
* prepare a recommendation;
* require a human to make the final approval decision.

A well-structured MCP server might expose:

### Resources

```text
policy://supplier-due-diligence
supplier://SUP-482/documents
supplier://SUP-482/profile
```

These provide context. The host can choose the relevant documents, inspect their size and include only the material needed for the review.

### Tools

```text
check_sanctions
search_adverse_media
record_review_findings
```

These perform operations. Claude may decide that a sanctions check is required, but the host can validate permissions and ask for approval before executing sensitive operations.

### Prompt

```text
/supplier-due-diligence-review
```

The user explicitly starts this standard workflow. The prompt instructs Claude to:

1. examine the selected supplier resources;
2. identify missing evidence;
3. invoke approved screening tools where necessary;
4. prepare a risk-based recommendation;
5. avoid making the final approval decision.

Notice that the prompt coordinates the activity but does not replace the resources or tools.

---

## 6. Exam-style question

These are **practice-derived scenarios**, not authentic certification questions.

A legal department is designing an MCP server for contract review.

The requirements are:

* Users explicitly choose a standard “Review contract” workflow.
* The host selects the contract and approved legal playbook to place in context.
* Claude may query the contract-management system for linked amendments.
* The system must clearly distinguish contextual data from executable operations.

Which design best fits MCP’s intended primitive model?

**A.** Expose the contract, playbook, linked-amendment query and review workflow as tools.

**B.** Expose the contract and playbook as resources, the amendment query as a tool, and the standard review workflow as a prompt.

**C.** Expose the contract as a prompt, the playbook as a tool and the amendment query as a resource.

**D.** Combine everything into one `review_contract` tool so Claude controls the complete process.

---

## 7. Spot the clue

Three phrases determine the answer:

> **“Users explicitly choose a standard workflow”**

This points to a **prompt**.

> **“The host selects the contract and approved legal playbook to place in context”**

This points to **resources**.

> **“Claude may query the contract-management system”**

This points to a **tool**.

Break the scenario into **workflow, context and action** before evaluating the options.

---

## 8. Answer reasoning

**Correct answer: B**

The contract and playbook are contextual information selected by the host, so resources are the natural representation. The amendment lookup is an operation that Claude may decide to invoke, so it belongs behind a tool. The named contract-review procedure is deliberately started by the user, making it suitable as a prompt. This follows MCP’s documented application-driven, model-controlled and user-controlled interaction patterns.

### Why D is tempting but weaker

A single `review_contract` tool appears operationally simple. It gives Claude one interface and hides the underlying steps.

However, it collapses three different concerns:

* selection of trusted context;
* execution of external operations;
* initiation of a standard user workflow.

This reduces transparency and makes it harder for the host to control which documents enter the context, obtain consent for particular operations and explain what the system is doing.

A coarse-grained tool could still be appropriate for a tightly bounded, deterministic backend service. It is weaker here because the requirements deliberately assign different responsibilities to the user, host and model.

### Why A is weaker

Representing all contextual information as tools makes the model responsible for deciding whether to retrieve fundamental material such as the contract itself and the approved legal playbook. The host already knows these materials are required and should control their inclusion.

### What additional fact could change the decision?

Suppose the “linked amendments” were already known, immutable documents that users selected alongside the contract. They could then be exposed as resources rather than retrieved through a tool.

Conversely, if retrieving an amendment required a live search, permission validation or interaction with an external system, a tool would remain appropriate.

---

## 9. One-line architect rule

> **Use resources for context, tools for operations and prompts for user-invoked reusable workflows.**

---

## 10. Source basis

* Official MCP documentation describing the three core server features and their intended controllers.
* Official MCP specification for application-driven resources and resource retrieval.
* Official MCP documentation for user-controlled, parameterised prompts.
* Practice-derived exam scenario based on official architectural patterns; it is not an authentic certification question.
