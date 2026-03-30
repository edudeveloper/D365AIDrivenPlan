# D365FO AI-Driven Delivery Plan

> A reusable framework that accelerates, standardizes, and improves D365FO delivery using 4 AI agents running in GitHub Copilot Agent Mode  covering the full customization lifecycle from functional design through to automated code review.

![GitHub issues](https://img.shields.io/github/issues/edudeveloper/D365AIDrivenPlan)
![GitHub](https://img.shields.io/github/license/edudeveloper/D365AIDrivenPlan)
![GitHub Repo stars](https://img.shields.io/github/stars/edudeveloper/D365AIDrivenPlan?style=social)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-2.1-4baaaa.svg)](https://avanade.github.io/code-of-conduct/)
[![Incubating InnerSource](https://img.shields.io/badge/Incubating-Ava--Maturity-%23FF5800?labelColor=yellow)](https://avanade.github.io/maturity-model/)

## Overview

This project defines an AI-assisted delivery framework for **Dynamics 365 Finance & Operations (D365FO)** projects (SCM / HR). It orchestrates 4 sequential AI agents  each with a clear input, output, and human checkpoint  to replace manual, error-prone steps in the consultant and developer workflow.

**Platform**: GitHub Copilot Agent Mode  
**Approach**: Human-in-the-loop with explicit checkpoints between each step  
**Status**: Step 1 (FDD Generator) implemented; Steps 24 planned

### The 4-Agent Pipeline

| Step | Agent | Status | Input | Output |
|------|-------|--------|-------|--------|
| 1 | **FDD Generator** |  Implemented | Azure DevOps Work Item (PBI/CR) | FDD document attached to DevOps |
| 2 | **TDD Generator** |  Planned | FDD from DevOps | TDD + Execution Prompt attached to DevOps |
| 3 | **X++ Code Generator** |  Planned | Execution Prompt from DevOps | X++ objects generated in local model |
| 4 | **Code Review Agent** |  Planned | GitHub Pull Request | Inline PR comments with fix suggestions |

### Step 1  FDD Generator

Reads an Azure DevOps Work Item, classifies requirements as **Standard / Configuration / Customization (GAP)**, runs asynchronous clarification loops via DevOps comments, and attaches a completed FDD (Word or Markdown) to the ticket.

**MCP Servers used**: Azure DevOps MCP · MS Learn MCP · D365FO MCP Server (Microsoft official, for mockups)

### Step 2  TDD Generator

Consumes the FDD from DevOps, queries the **Ninja MCP Server** (584K+ indexed AOT objects) for extensibility analysis, and produces two artefacts: a full TDD and a structured **Execution Prompt** for the X++ Agent. Every design decision is traced to Ninja MCP evidence. A continuous **Lessons Learned loop** updates the agent's playbook via GitHub PRs approved by the Tech Lead.

**MCP Servers used**: Azure DevOps MCP · Ninja MCP Server (hybrid) · MS Learn MCP

### Step 3  X++ Code Generator

Consumes the Execution Prompt from Step 2 and generates D365FO AOT objects in dependency order (Labels  Enums  EDTs  Tables  Classes  Forms  Reports  Security). Includes a **Build Validation Loop**: compile  diagnose  auto-fix  ask developer. Uses Ninja MCP for all file creation and metadata operations.

**MCP Servers used**: Azure DevOps MCP · Ninja MCP Server (hybrid) · MS Learn MCP

### Step 4  Code Review Agent

Analyses the GitHub PR diff against official MS documentation, project rules (Manifesto), and the Code Review Playbook (lessons synthesized from Tech Lead PR comments). Posts actionable inline comments. Trigger mechanism (GitHub Action vs. manual) is under research.

**MCP Servers used**: MS Learn MCP · GitHub MCP · Manifesto / Context (local)

### Knowledge Sources

| Source | Content | Used by |
|--------|---------|---------|
| MS Learn MCP | Official Microsoft documentation and best practices | Steps 14 |
| Ninja MCP Server (hybrid) | 584K+ indexed AOT objects  standard, ISV, internal assets, local model | Steps 23 |
| D365FO MCP Server (MS official) | Standard form layouts for mockup generation | Step 1 |
| Azure DevOps MCP | Work items, FDD/TDD, comments, attachments | Steps 13 |
| Manifesto / Constitution | Project-specific rules: prefix, naming, detail level | Steps 24 |
| Playbook / Skills | Lessons learned per agent, approved by Tech Lead via GitHub PR | Steps 24 |

## Presentation

The `presentation.html` file in the root of this repository is a self-contained interactive overview of the framework  open it directly in a browser. Each of the 4 agents has a dedicated detail page (`step-1-fdd.html`, `step-2-tdd.html`, `step-3-xpp.html`, `step-4-cr.html`). The `detailed-plan.html` page contains the full implementation plan.

## Licensing

This project is for internal Avanade use only, without legal review. See the Avanade Open Source site to start legal approvals.

## Solutions Referenced

- [Ninja MCP Server for D365FO](https://github.com/dynamics365ninja/d365fo-mcp-server)  584K+ indexed AOT objects, hybrid local/cloud mode
- [Azure DevOps MCP](https://learn.microsoft.com/en-us/azure/devops/)  Work item and attachment management
- [MS Learn MCP](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/)  Official D365FO documentation
- [GitHub Copilot Agent Mode](https://docs.github.com/en/copilot)  AI agent execution platform

## Documentation

The `docs` folder contains [more detailed documentation](./docs/start-here.md) and the full [implementation plan](./D365FO_AI_Implementation_Plan_v1.md).

## Contact

Feel free to [raise an issue on GitHub](https://github.com/edudeveloper/D365AIDrivenPlan/issues), or see our [security disclosure](./SECURITY.md) policy.

## Contributing

Contributions are welcome. See information on [contributing](./CONTRIBUTING.md), as well as our [code of conduct](https://avanade.github.io/code-of-conduct/). Avanade asks that all commits sign the [Developer Certificate of Origin](https://developercertificate.org/).

If you're happy to follow these guidelines, then check out the [getting started](./docs/start-here.md) guide.

## Who are Avanade?

[Avanade](https://www.avanade.com) is the world's leading expert on Microsoft. Trusted by over 7,000 clients worldwide, we deliver AI-driven solutions that unlock the full potential of people and technology, optimize operations, foster innovation and drive growth.

As Microsoft's Global SI Partner we combine global scale with local expertise in AI, cloud, data analytics, cybersecurity, and ERP to design solutions that prioritize people and drive meaningful impact.

We champion diversity, inclusion, and sustainability, ensuring our work benefits society and business.

Learn more at [www.avanade.com](https://www.avanade.com)
