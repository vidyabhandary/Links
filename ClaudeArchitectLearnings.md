# Some learnings for Claude Architect 

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
