# Some learnings for Claude Architect 

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
