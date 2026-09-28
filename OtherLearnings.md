# Sep 28, 206

---

**1. Every corpus holds two kinds of fact, and retrieval can only work on one.**

- *In the document* — what it says: the procedure, the clause, the torque value, the definition. This is similarity's job.
- *About the document* — its standing in the world: in force or superseded, applicable to which entity, who may see it, valid between which dates, live or withdrawn. Never reliably recoverable from the text.
- The test: **if an authority could change the fact without editing a single word of the document, it is a fact about it.** A revision is superseded by a decision, not by a rewrite.
- Facts about a document are metadata — structured, deterministic, applied before similarity runs.

**2. Presence is not addressability.** Being written in the document does not make a fact usable by retrieval. Two conditions must both hold:

- *The query's own words must point at it.* "Torque sequence for the aft flange" shares almost no vocabulary with "Effective for serials 1200–1450", so the applicability line is near-invisible to that query. Same mechanism as a clause separated from its qualifier.
- *The answer must be a passage to read, not a test to apply.* Embeddings rank by closeness. They do not evaluate booleans, ranges, dates or set membership — 1327 and 1237 are near-identical as tokens.
- Corollary: pasting the constraint into every chunk does not fix it. The disqualified document is still retrieved, the check becomes the model's judgement, and the repeated boilerplate dilutes every embedding.

**3. Passage to read → retrieval. Test to apply → structured field.** The sorting rule for any requirements list.

- Tests to apply: status, permission, membership, range, date comparison, version.
- Passages to read: procedures, definitions, clauses, explanations, values stated in context.

**4. Currency is relative, never global.** "Latest" is a property of a document; "in force" is a relationship.

- In force = **for this entity, for this task, on this date.** One document can be current for one customer, aircraft or jurisdiction and superseded for another.
- Retaining superseded versions is usually a requirement, not technical debt: audit and investigation ask what was approved on the date of the event.
- So "delete the old ones" is the wrong fix — it destroys the time dimension. Constrain per query instead.

**5. Filter or rank? One question decides it.** Does violating the constraint make the answer *wrong or unsafe*, or merely *less good*?

- Wrong or unsafe → **hard pre-filter, before similarity, fail closed.** Entitlements, currency, effectivity, withdrawal.
- Less good → **ranking signal.** Preferences, source seniority, recency bias, house style.
- Three enforcement points, in descending strength: pre-filter (never enters the pipeline) → post-filter (already read and logged — for entitlements that is the breach) → prompt instruction (probabilistic, never a control).
- A hard filter converts a fuzzy judgement into permanent, invisible deletion. Ranking demotes and leaves the material reachable.

**6. Every filter is a dependency you now own.** Determinism is not free safety.

- A missing or wrong value excludes the document with **no error**. Silent and absolute.
- Each field needs a source system, a freshness SLA, a missing-value alarm and an explicit empty-result path.
- "Add a structured field for everything" is not caution — it is more ways to lose documents invisibly.
- Engineering note: in graph-based vector indexes a very selective pre-filter can degrade recall. An implementation concern, not a reason to move a safety check downstream.

**7. Two procedures to run on any architecture question.**

*Locating the fault — the layer test:*

- State the failure as "it returned X instead of Y because Z".
- Find the stage that owns Z: ingestion → chunking → indexing → retrieval → reranking → context assembly → generation.
- Label each option with the stage it acts on. An option acting on a different stage cannot fix the fault — that is the discriminating reason, read off the labels rather than invented.

*Naming the residual risk — three questions:*

- What does this remove, and who needed it?
- What does this now depend on that did not exist before? Name the field, feed or team — not "latency" or "expense".
- How would anyone find out it went wrong? If the answer is "they would not", that is the risk.

**Canonical order:** entitlements → metadata → similarity → context expansion → rerank → generate. Deterministic before probabilistic, always.

# Sep 2, 2026

## Late Interaction Retrieval 
— ColBERT for Fine-Grained Matching

### Concept

Most embedding search compresses an entire chunk into **one vector**.

That is fast, but it can lose detail. A document about “SAP payroll integration failures” may contain many important concepts, yet all are squeezed into one representation.

**Late interaction models such as ColBERT keep multiple token-level vectors instead.** At query time, each query token is matched against the most relevant document token using **MaxSim**, preserving much finer semantic detail. ([Qdrant][1])

This is **not late chunking**. Late chunking improves how a chunk embedding is created; late interaction changes how query and document representations are compared.

### Practical case study

Suppose your support knowledge base contains a long troubleshooting article covering:

> SAP authentication, timeout handling, certificate expiry, RFC failures, and payroll errors.

A user asks:

> “Why does payroll fail only when the SAP certificate rotates?”

A single-vector embedding may broadly classify the article as “SAP troubleshooting.”

ColBERT can independently align **payroll**, **certificate**, and **rotation** with the relevant parts of the document, making it useful as a high-precision reranker.

A practical production pattern is:

**fast dense retrieval → top 50 candidates → ColBERT reranking → top 5 → LLM**

Qdrant currently supports this two-stage pattern directly: dense retrieval can prefetch candidates and a multivector ColBERT query can rerank them in the same query operation. ([Qdrant][2])

FastEmbed currently exposes `LateInteractionTextEmbedding`, using `.embed()` for documents and `.query_embed()` for queries. Qdrant recommends disabling HNSW indexing on the ColBERT multivectors when they are used purely for reranking, because the dense index already performs candidate retrieval. ([Qdrant][1])

### When to use it / when not to

Use late interaction when retrieval quality matters for **long or multi-topic documents, technical documentation, contracts, research, and queries containing several important concepts**.

Avoid running ColBERT across your entire large corpus if ordinary dense or hybrid retrieval already works well. Multivectors require more storage and MaxSim scoring is more computationally expensive. The usual production pattern is therefore **retrieve cheaply first, rerank precisely second**. ([Qdrant][1])

### Architecture takeaway

There is an important progression in retrieval design:

**single vector → hybrid retrieval → reranking → late-interaction reranking**

You do not necessarily replace your existing vector search with ColBERT.

You can use:

**Dense/BM25 = candidate generator**
**ColBERT = precision layer**
**LLM = answer generator**

**Key principle:** *Use expensive semantic precision only on the small set of documents that have already earned the right to be examined closely.*

[1]: https://qdrant.tech/course/essentials/day-5/colbert-multivectors/ "Multivectors for Late Interaction Models - Qdrant"
[2]: https://qdrant.tech/documentation/tutorials-search-engineering/using-multivector-representations/ "Multivectors and Late Interaction - Qdrant"




# Aug 25, 2026

## Plan a Responsible AI Strategy

**Study-guide position:** The current AB-620 guide still places **“Plan responsible AI strategy”** immediately after *Plan channels and deployment*. ([Microsoft Learn][1])

The mistake to avoid is thinking Responsible AI means:

> “Microsoft already has content filters, so we're covered.”

It doesn't.

A **Responsible AI strategy** is your design for keeping the *whole agent system*—model, data, tools, users, people affected by its decisions, and operating environment—within acceptable boundaries throughout its lifecycle. Microsoft explicitly treats Responsible AI as a design concern from the first architecture decisions, not a compliance check added immediately before production. ([Microsoft Learn][2])

---

## Start with Microsoft's six principles

Microsoft's Responsible AI framework uses six principles:

| Principle                | Ask yourself                                                          |
| ------------------------ | --------------------------------------------------------------------- |
| **Fairness**             | Does the agent treat comparable people comparably?                    |
| **Reliability & safety** | Does it behave predictably and fail safely?                           |
| **Privacy & security**   | Does it protect information and respect access boundaries?            |
| **Inclusiveness**        | Can the intended range of users successfully use it?                  |
| **Transparency**         | Do users understand that AI is involved and what its limitations are? |
| **Accountability**       | Is a person or team ultimately responsible for its behavior?          |

([Microsoft Learn][2])

A memory aid:

## **F-R-P-I-T-A**

**F**air
**R**eliable
**P**rivate
**I**nclusive
**T**ransparent
**A**ccountable

But the exam is unlikely to be interesting if it merely asks you to recite six words.

You need to translate the principles into **architecture decisions**.

---

# 1. Reliability and safety starts with scope

Suppose you build an HR agent.

Its intended purpose is:

> Answer employee-benefit questions from approved HR documentation.

Now a user asks:

> “Based on everything you know about me, should the company fire my manager?”

A poor design says:

> The model can generate an answer, therefore allow it.

A responsible design asks:

> **Is this within the intended use of the system?**

It isn't.

Your agent needs a defined **scope boundary**.

Conceptually:

```text
INTENDED
Benefits
Leave policies
Expense policies
HR processes

        │
        ▼

      AGENT

        │
        ▼

OUT OF SCOPE
Performance judgments
Employment decisions
Medical diagnosis
Legal advice
```

Generative AI's ability to answer something is **not evidence that the agent should answer it**.

This is one of the most important Responsible AI ideas.

---

# 2. Grounding is partly a Responsible AI control

Consider:

> “How many weeks of parental leave do employees receive?”

Without grounding, the LLM might answer from general knowledge.

That can create a perfectly fluent but completely incorrect corporate-policy answer.

Instead:

```text
Employee question
       ↓
Retrieve approved HR policy
       ↓
Generate answer from retrieved material
       ↓
Return grounded response
```

This improves **reliability** and **transparency**.

For tightly controlled scenarios, Copilot Studio allows makers to restrict generative answers rather than allowing answers from broader general knowledge. Microsoft's current guidance specifically notes that general knowledge can be disabled when agents should answer only within configured knowledge sources. ([Microsoft Learn][3])

### Exam reasoning

Requirement:

> “The agent must answer only from approved corporate policies.”

Think:

**constrain knowledge scope + grounding**

—not:

> “Give the model a very strong instruction to never hallucinate.”

Instructions help, but instructions aren't a substitute for architectural controls.

---

# 3. Human-in-the-loop is about consequence

Consider three actions.

### Action A

> Summarize an IT troubleshooting article.

Low consequence.

The agent can normally do that autonomously.

---

### Action B

> Categorize incoming support tickets.

Some consequence, but mistakes are usually reversible.

Autonomous execution plus monitoring might be acceptable.

---

### Action C

> Terminate an employee's access and payroll.

High consequence.

The design should probably not be:

```text
Agent decides
     ↓
Terminate employee
```

Instead:

```text
Agent detects/request action
       ↓
Prepare recommendation
       ↓
Human reviewer
       ↓
Approve / reject
       ↓
Execute
```

Microsoft's current Responsible AI guidance explicitly recommends human approval for consequential actions—especially those affecting **people, money, or compliance**, or those that are difficult to reverse. ([Microsoft Learn][2])

The rule isn't:

> “Every agent needs human approval.”

That would destroy much of the benefit of automation.

The better rule is:

> **Human oversight should scale with consequence and uncertainty.**

---

# 4. Responsible AI controls should scale with risk

Compare these agents:

### Agent 1 — Cafeteria agent

Answers:

> “What's for lunch today?”

Risk is low.

---

### Agent 2 — Procurement agent

Can approve:

> €500 purchases.

Risk is higher.

---

### Agent 3 — Credit-decision agent

Makes recommendations affecting whether customers receive loans.

Risk is much higher.

You should not apply identical governance effort to all three.

Think:

```text
LOW RISK
Informational
Reversible
Low sensitivity
       ↓
Lighter controls

MEDIUM RISK
Transactional
Some sensitive data
Financial effect
       ↓
Stronger evaluation + monitoring

HIGH RISK
People / money / legal / compliance
Hard to reverse
       ↓
Human oversight
Formal review
Extensive testing
Auditability
```

Microsoft recommends risk-tiering Responsible AI reviews so the amount of review and control scales with potential impact. ([Microsoft Learn][2])

---

# 5. Built-in safety ≠ complete Responsible AI strategy

Copilot Studio provides platform protections.

Microsoft currently applies content moderation to generative AI interactions, including protections aimed at:

* harmful or offensive content,
* jailbreaks,
* prompt injection,
* prompt exfiltration,
* copyright-related risks.

Content can be checked on both input and generated output. ([Microsoft Learn][4])

That's valuable.

But consider this agent:

> An AI recruiting assistant ranks candidates.

No user writes anything harmful.

No jailbreak occurs.

No security exploit occurs.

Yet the ranking systematically disadvantages a category of applicants.

Content filtering will not magically solve that problem.

That's a **fairness problem**.

Similarly:

> The agent confidently recommends the wrong insurance policy.

That may be a **reliability problem**.

> Users don't know an AI made the recommendation.

That's a **transparency problem**.

> Nobody owns the outcome.

That's an **accountability problem**.

So remember:

```text
PLATFORM GUARDRAILS
        ≠
COMPLETE RESPONSIBLE AI
```

Responsible AI is broader than content safety.

---

# 6. Transparency has two layers

Transparency does **not** necessarily mean:

> Show the user the model's internal reasoning.

Instead, users need appropriate information such as:

* this interaction involves AI,
* what the agent is intended to do,
* what it should not be trusted to do,
* whether an answer came from authoritative sources,
* when human escalation is appropriate.

Copilot Studio's generative-answer guidance recommends informing users that AI may be used in responses. ([Microsoft Learn][3])

For example:

### Weak

> “Your refund request was rejected.”

### Better

> “Based on the current refund policy and the order information available to me, this request doesn't appear eligible. You can request human review if you believe the information is incorrect.”

The second design preserves more **user agency**.

---

# 7. Responsible AI continues after deployment

Suppose your agent passed every test in January.

By June:

* documents have changed,
* users have discovered new prompting patterns,
* a model version has changed,
* business policy has changed,
* the agent has been given another tool.

Does the January approval prove the June agent is still responsible?

No.

Microsoft's current guidance explicitly treats Responsible AI as **continuous**: monitor groundedness, safety, escalations, and reported issues because models, data, usage, and regulations can change. ([Microsoft Learn][2])

Think lifecycle:

```text
DESIGN
  ↓
RISK ASSESSMENT
  ↓
TEST
  ↓
RELEASE
  ↓
MONITOR
  ↓
LEARN
  ↓
REASSESS
```

Responsible AI isn't a document.

It's a **feedback loop**.

---

# Enterprise example: Procurement Agent

Contoso wants an agent that:

1. explains procurement policy,
2. recommends suppliers,
3. raises purchase requests,
4. automatically approves low-value purchases.

A Responsible AI strategy might look like this:

### Explain policy

Use approved policy sources and grounded responses.

**Concern:** reliability.

---

### Recommend suppliers

Test recommendations for systematic bias and ensure recommendation criteria are defensible.

**Concern:** fairness + transparency.

---

### Raise a purchase request

Clearly show what data/action will be submitted.

**Concern:** transparency + accountability.

---

### Auto-approve €100 stationery purchases

Potentially acceptable with limits and audit logs.

**Concern:** reliability + accountability.

---

### Auto-approve a €750,000 supplier contract

Very different risk.

Add human review.

```text
Agent recommendation
      ↓
Procurement officer
      ↓
Approval
      ↓
ERP execution
```

Same technology.

Different **risk architecture**.

That distinction is exactly how you should approach scenario questions.

---

# Exam trap: Responsible AI vs security/governance

The **next AB-620 objective is security and governance**, so Microsoft deliberately treats these as related but separate planning concerns. ([Microsoft Learn][1])

A useful distinction:

### Responsible AI

> **Should the system behave this way, and what impact could its behavior have?**

Examples:

* bias,
* hallucination,
* unsafe autonomy,
* transparency,
* appropriate human oversight.

### Security/governance

> **What technical and administrative controls govern access and use?**

Examples:

* DLP/data policies,
* RBAC,
* environment governance,
* auditing,
* tenant policies.

There is overlap—especially around privacy—but don't collapse them into the same concept.

---

## Memory model: **S-G-H-M**

Before releasing an agent, ask:

**S — Scope**
What is it allowed to do?

**G — Ground**
What trusted information should support its answers?

**H — Human**
Which decisions/actions require human judgment?

**M — Monitor**
How will we detect failures after deployment?

If a scenario involves high-impact AI, mentally run **S-G-H-M**.

---

# Quick check

### 1. A benefits agent must answer only from approved HR policy documents. What's the strongest design direction?

**Answer:** Ground responses in approved sources and constrain the agent from relying on unrestricted general knowledge.

A prompt saying “please don't hallucinate” alone is not enough.

---

### 2. An autonomous agent wants to close a €2-million supplier contract after evaluating bids. What's the key Responsible AI concern?

**Answer:** The consequence of the action warrants **human oversight/approval**.

The question isn't simply whether the agent is technically capable of executing it.

---

### 3. Copilot Studio already provides content moderation. Does that eliminate the need to test an employee-selection agent for bias?

**Answer: No.**

Content safety and fairness are different Responsible AI concerns. Microsoft treats fairness, reliability/safety, privacy/security, inclusiveness, transparency, and accountability as separate principles. ([Microsoft Learn][2])

## One-sentence takeaway

> **For AB-620, a Responsible AI strategy means defining the agent's safe scope, grounding high-value answers, scaling human oversight to consequence, being transparent with users, and continuously evaluating the system—not merely relying on built-in content filters.**

[1]: https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ab-620?utm_source=chatgpt.com "Study guide for Exam AB-620: Designing and Building Integrated AI Solutions in Copilot Studio | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/agents/center-of-excellence/responsible-ai?utm_source=chatgpt.com "Apply responsible AI | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-generative-answers?utm_source=chatgpt.com "FAQ for generative answers - Microsoft Copilot Studio | Microsoft Learn"
[4]: https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-agent-creation?utm_source=chatgpt.com "FAQ for the agent creation from natural language experience - Microsoft Copilot Studio | Microsoft Learn"
