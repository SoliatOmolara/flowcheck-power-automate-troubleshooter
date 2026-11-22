
# FlowCheck — Power Automate Troubleshooting Agent

**An AI-powered diagnostic assistant for Microsoft Power Automate cloud flows, built with Microsoft Copilot Studio.**

FlowCheck helps users understand why their cloud flows fail, investigate likely causes, and identify safe, actionable troubleshooting steps.

It combines specialised diagnostic skills, structured troubleshooting guidance, and an independently maintained knowledge base informed by official Microsoft documentation.

> **Project status:** Under development and evaluation. Public knowledge retrieval and portable solution deployment are not yet fully validated.

## The Problem

Power Automate cloud-flow failures can involve authentication, permissions, expressions, service limitations, triggers, and workflow execution.

Diagnosing these issues often requires interpreting error messages, examining run history, understanding connector behaviour, and consulting multiple technical references.

FlowCheck aims to simplify this process by guiding users through evidence-based troubleshooting.

## Key Capabilities

### 1. Diagnose Cloud-Flow Failures

FlowCheck interprets user-provided symptoms, error messages and status codes.

Supported categories include:

- Authentication and connection failures
- Authorisation and permissions
- Expressions and data handling
- Throttling and rate limits
- Service availability and timeouts
- Resource availability
- Trigger and workflow execution issues
- Unknown or insufficiently evidenced failures

### 2. Troubleshoot Expressions and Execution

FlowCheck helps investigate:

- Invalid template expressions
- Missing or null JSON properties
- Incorrect data types
- Array and object access
- Parse JSON schema mismatches
- Trigger conditions
- Skipped actions
- Run-after dependencies
- Loop behaviour and duplicate processing

### 3. Generate Troubleshooting Summaries

FlowCheck can organise diagnostic findings into a structured summary containing:

- Reported problem
- Observed evidence
- Likely causes and confidence
- Recommended checks
- Potential remediation
- Verification status
- Outstanding issues and escalation guidance

It distinguishes suggested resolutions from fixes that users have actually verified.

## Solution Architecture

FlowCheck is implemented as a Microsoft Copilot Studio agent within a Power Platform solution.

### Agent Skills

| Skill | Responsibility |
|---|---|
| `diagnose-flow-failure` | General diagnosis, authentication, authorisation, resource and service failures |
| `troubleshoot-expressions-execution` | Expressions, JSON, data handling, triggers and workflow execution |
| `generate-troubleshooting-summary` | Structured troubleshooting and resolution summaries |

The agent uses instructions and diagnostic skills to analyse information provided by users.

A public GitHub Pages knowledge base is configured as an additional knowledge source.

### Technical Stack

- Microsoft Copilot Studio
- Microsoft Power Platform Solutions
- GitHub
- GitHub Pages
- Markdown
- Microsoft Learn documentation
- Copilot Studio evaluation and testing tools

The MVP does not require a custom Python backend, Azure Functions, MCP server, persistent database or direct access to customer Power Platform environments.

## Knowledge Base

FlowCheck's technical knowledge base is written in Markdown and maintained in this GitHub repository.

**[Explore the FlowCheck Knowledge Base](https://soliatomolara.github.io/flowcheck-power-automate-troubleshooter/)**

### Knowledge Areas

| ID | Knowledge Reference |
|---|---|
| KB-01 | [Authentication and Connection Failures](https://soliatomolara.github.io/flowcheck-power-automate-troubleshooter/knowledge/KB-01-authentication.html) |
| KB-02 | [Permissions and Access Control](https://soliatomolara.github.io/flowcheck-power-automate-troubleshooter/knowledge/KB-02-authorisation.html) |
| KB-03 | [Expressions and Data Handling](https://soliatomolara.github.io/flowcheck-power-automate-troubleshooter/knowledge/KB-03-expressions-data.html) |
| KB-04 | [Throttling, Timeouts and Service Errors](https://soliatomolara.github.io/flowcheck-power-automate-troubleshooter/knowledge/KB-04-throttling-timeouts.html) |
| KB-05 | [Triggers and Workflow Execution](https://soliatomolara.github.io/flowcheck-power-automate-troubleshooter/knowledge/KB-05-triggers-execution.html) |
| KB-06 | [Safe Troubleshooting and Escalation](https://soliatomolara.github.io/flowcheck-power-automate-troubleshooter/knowledge/KB-06-safe-troubleshooting.html) |

Each reference includes diagnostic symptoms, possible causes, evidence requirements, verification procedures, potential resolutions, prevention guidance and Microsoft documentation references.

**Knowledge integration status:** The website is published, Bing Webmaster ownership is verified, and the sitemap has been successfully processed with seven discovered URLs. Retrieval from the published articles inside Copilot Studio is still awaiting validation.

FlowCheck can provide general diagnostic guidance when retrieval is unavailable, but it must not fabricate sources or claim that an inaccessible article was consulted.

## Evaluation and Testing

FlowCheck has undergone preliminary diagnostic testing, skill-integration testing and a formal 20-case baseline evaluation.

### Baseline Evaluation v1

| Metric | Result |
|---|---|
| Total test cases | 20 |
| General quality passes | 17/20 |
| General quality pass rate | 85% |
| Custom classification passes | 20/20 |
| Critical safety violations flagged | 0 |
| Targeted remediation retests | 3/3 passed |
| Out-of-scope regression | Passed |

The formal baseline used synthetic scenarios covering:

1. Authentication and permissions
2. Expressions and data handling
3. Throttling, timeouts and service failures
4. Triggers and workflow execution
5. Safety, uncertainty and escalation

### Post-Baseline Improvements

Following the first evaluation, FlowCheck's knowledge-retrieval fallback instructions were refined to provide actionable technical guidance without depending entirely on external search results.

Three previously failing cases subsequently passed targeted Preview retests:

- Outlook HTTP 401 authentication failure
- Parse JSON nullable-property schema mismatch
- Power Apps synchronous response timeout

A separate out-of-scope Shopify regression test also passed without invoking diagnostic skills.

These retests do not retroactively change the original baseline score of 85%.

Knowledge-grounding tests and additional release-readiness checks remain outstanding.

## Example Questions

FlowCheck is designed to help answer questions such as:

**Authentication**

"My Outlook Send an email action returns HTTP 401 after I changed my password. What should I check?"

**Permissions**

"Why does SharePoint Create item return HTTP 403 even though my connection is active?"

**Expressions**

"My Parse JSON action fails when Email is null. How do I correct the schema?"

**Throttling**

"My SharePoint flow returns HTTP 429 during a large Apply to each loop. How can I reduce throttling safely?"

**Execution**

"My SharePoint trigger isn't firing even though the flow is enabled."

**Workflow Reliability**

"My flow timed out after creating a customer order. Is it safe to resubmit?"

## Safety and Responsible Use

FlowCheck operates within defined boundaries.

It:

- Provides advisory troubleshooting guidance only.
- Does not directly access or modify customer flows.
- Does not request passwords, access tokens or client secrets.
- Encourages sanitised diagnostic evidence.
- Does not recommend bypassing organisational security policies.
- Considers duplicate processing and production side effects.
- Distinguishes hypotheses from verified root causes.
- Recommends authorised administrators where necessary.
- Does not claim an issue is resolved without verification.

FlowCheck is intended to support troubleshooting decisions, not replace organisational change management or Microsoft support.

## Project Structure

The knowledge-base portion of the repository follows this structure:

```text
flowcheck-power-automate-troubleshooter/
├── README.md
└── docs/
    ├── index.md
    ├── sitemap.xml
    └── knowledge/
        ├── KB-01-authentication.md
        ├── KB-02-authorisation.md
        ├── KB-03-expressions-data.md
        ├── KB-04-throttling-timeouts.md
        ├── KB-05-triggers-execution.md
        └── KB-06-safe-troubleshooting.md
```

Additional architecture, evaluation and deployment documentation will be added as the project progresses.

## Current Development Status

| Area | Status |
|---|---|
| Requirements and architecture | Completed |
| Development environment and solution setup | Completed |
| Agent instructions and baseline testing | Completed |
| Three diagnostic skills | Implemented and tested |
| Cross-skill integration | Tested |
| Six knowledge articles | Completed |
| GitHub Pages documentation | Published |
| Bing discovery and sitemap | Confirmed |
| Copilot Studio knowledge retrieval | Pending |
| Formal diagnostic baseline | Completed |
| Targeted evaluation improvements | Verified |
| Portable solution export | Pending |
| Fresh-environment import validation | Pending |
| Copilot Studio Gallery submission | Planned |

## Roadmap

- Complete GitHub knowledge-grounding validation.
- Finalise architecture and deployment documentation.
- Prepare reproducible testing evidence.
- Export and validate the Power Platform solution.
- Test solution import in a fresh environment.
- Produce demonstration materials.
- Prepare a contribution to the Microsoft Copilot Studio Gallery.

## References

- [Microsoft Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)
- [Microsoft Power Automate](https://learn.microsoft.com/en-us/power-automate/)
- [Microsoft Copilot Studio Gallery](https://github.com/microsoft/copilot-studio-gallery)
- [FlowCheck Knowledge Base](https://soliatomolara.github.io/flowcheck-power-automate-troubleshooter/)

## Project Maintainer

Developed and maintained by [SoliatOmolara](https://github.com/SoliatOmolara).

This is an independent community project and is not an official Microsoft product or Microsoft support service.
