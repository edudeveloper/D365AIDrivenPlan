# D365FO Implementation Framework with AI Agents

> **Status**: Final v1 — 30/Mar/2026  
> **Platform**: GitHub Copilot Agent Mode  
> **Scope**: D365 FnO projects (SCM / HR)  
> **Approach**: Human-in-the-loop with checkpoints between each step

## TL;DR

A reusable framework that accelerates, standardizes, and improves the performance of D365FO consultants and developers using 4 AI agents in sequential steps: **FDD → TDD → X++ → Code Review**. Agents consume knowledge through MS Learn MCP, Ninja MCP Server (584K+ indexed objects), the official Microsoft D365FO MCP Server, and the architect's manifesto. FDD/TDD output goes to Azure DevOps; X++ goes to the developer's local model; Code Review operates on GitHub PRs.

### Steps Overview

| Step | Agent | Status | Input | Output |
|------|-------|--------|-------|--------|
| 1 | FDD Generator | ✅ Implemented | DevOps WI (PBI/CR) | FDD attached to DevOps |
| 2 | TDD Generator | 🔲 To implement | FDD from DevOps | TDD + Execution Prompt attached to DevOps |
| 3 | X++ Code Generator | 🔲 To implement | Execution Prompt from DevOps | X++ code in local model |
| 4 | Code Review Agent | 🔲 To implement | PR on GitHub | Comments on the PR |

---

## Step 1: FDD Generation (Functional Design Document)

**Status**: ✅ IMPLEMENTED

### Workflow

1. **Input via Azure DevOps** — The agent reads the Work Item (PBI, CR, or other type) and understands the Business Requirement. It analyses all ticket comments for full context.
   - For **Change Requests**: downloads and analyses the FDD from the parent WI
   - Supports free text as an alternative input

2. **MS Learn Lookup** — Checks what can be handled out-of-the-box by D365FO to minimise customisations.

3. **Requirement Classification**:
   - **Standard**: handled natively
   - **Configuration**: handled via setup — *generates a D365FO configuration guide*
   - **Customisation (GAP)**: requires development

4. **Clarification Requirement Process**:
   - Asks the consultant who is running the agent
   - Actively reduces assumptions
   - If the consultant cannot answer: posts a comment on DevOps tagging the responsible area owner
   - When the owner replies, the consultant restarts — the agent picks up the clarifications from the ticket comments

5. **Output** — Attaches the FDD to the DevOps ticket (Word or Markdown)

6. **Operating modes**:
   - **Autonomous**: generates the full document and attaches it to DevOps for review
   - **Co-authoring**: the consultant reviews topic by topic before the final document

### MCP Servers — Step 1

| MCP Server | Purpose |
|------------|---------|
| **Azure DevOps MCP** | Reads WI, comments, uploads/downloads attachments, tags users |
| **MS Learn MCP** | Official documentation for Standard/Config/GAP classification |
| **D365FO MCP Server (MS official)** | *To be introduced* for mockup generation (standard form layouts + custom fields/buttons) |

### Confirmed Capabilities

- Reads WI + DevOps comments
- Downloads FDD (Word) from parent WI for context on CRs
- Standard/Config/GAP classification with configuration guide for config-only items
- Asynchronous clarification process via DevOps comments
- Flexible for different WI structures (PBI, CR, etc.)
- Attaches FDD to the ticket (Word/Markdown)

### Attention Points — Step 1

| Point | Type | Detail |
|-------|------|--------|
| Standard vs GAP classification | Risk | Agent may make mistakes. Validation with MS Learn evidence partially mitigates this |
| Clarifications without response | Risk | If the owner does not reply on DevOps, the FDD remains blocked. Consider timeout/escalation |
| Decision traceability | Suggestion | Each classification and clarification should be logged in the FDD for audit purposes |
| FDD versioning on CRs | Suggestion | The new FDD should reference what changed compared to the original FDD (semantic diff) |

---

## Step 2: TDD Generation (Technical Design Document)

**Status**: 🔲 TO IMPLEMENT

### Workflow

1. **Trigger** — The developer invokes the agent via a prompt in VS Code Copilot (or Visual Studio). The agent identifies the WI in DevOps and locates the FDD on the sibling Task.

2. **FDD Consumption** — Downloads/reads the FDD from DevOps (via Azure DevOps MCP). Extracts all customisations (GAPs) that need technical design.
   - *Future*: consume mockups from the FDD to map fields/buttons to AOT objects

3. **Technical Analysis via Ninja MCP** — For each customisation:
   - Queries Ninja MCP Server (hybrid mode) to identify AOT objects to extend/create
   - Identifies: tables, fields, forms, classes, data entities, extension points
   - When unsure: starts a **Clarification Process** with the developer
   - **Cites evidence** from Ninja MCP for every design decision (e.g. *"CoC on SalesTable.validateWrite() — based on: recommend_extension_strategy + analyze_extension_points + find_coc_extensions with no existing wrapper"*)

4. **MS Learn MCP Lookup** — Validates extensibility best practices (CoC vs event handler, etc.)

5. **Manifesto / Constitution Application** — A local file from the architect that defines:
   - Model prefix (used in methods and objects)
   - Prefix or suffix for naming conventions
   - Configurable detail level: signatures + comments only **OR** full scaffold
   - Project-specific rules

6. **Clarification Process** — Similar to Step 1: asks the developer about points that need clarification.

7. **Operating modes**:
   - **Autonomous**: generates the full TDD, attaches it to DevOps, posts a comment requesting review
   - **Co-authoring**: the developer reviews topic by topic, method by method

8. **Output** — Two artefacts attached to the SubTask in DevOps:
   - **TDD** — full technical document for developer review
   - **Execution Prompt for X++ Agent** — Structured Markdown (schema below) with specific instructions for the Step 3 agent. Acts as an **executable contract** between Step 2 and Step 3.
   - Comment on the WI notifying that the TDD is ready for review.

### Execution Prompt Schema (Step 2 → Step 3)

Format: **Structured Markdown with sections per object**, attached to DevOps as a `.md` file.

```markdown
# Execution Prompt — X++ Agent

## Context
- **WI**: #{number} — {title}
- **Primary model**: {packageName}
- **Project path**: {path to .rnrproj}
- **Manifesto**: {manifesto version applied}
- **FDD ref**: {link or filename of the FDD}
- **TDD ref**: {link or filename of the TDD}

## Objects (in generation order)

### [Order] [Type] — [Full name with prefix]

- **Action**: create | extend
- **AOT Type**: table | class | form | enum | edt | view | query | data-entity | menu-item-display | menu-item-action | security-privilege | ...
- **Package/Model**: {packageName} (only if different from the primary model)
- **Base object** *(extensions only)*: {name of the standard/ISV object to extend}
- **Dependencies in this prompt**: {list of objects in this prompt that must exist first}

#### Specification

*(content varies by type — examples below)*

**For Tables / Table Extensions:**
- Fields: {name} | {EDT} | {mandatory?} | {description}
- Indexes: {name} | {fields} | {unique?}
- Relations: {name} | {foreign table} | {fields}
- Field Groups: {name} | {fields}
- Methods: {signature} | {expected logic as comment}

**For Classes (CoC / Event Handler / Batch / SysOperation):**
- Type: CoC | EventHandler | BatchJob | SysOperationService | Helper
- Target object *(CoC/EH)*: {class.method or table.method}
- Extension point *(CoC/EH)*: {CoC wrap | pre-event | post-event | delegate}
- Signature: {full method signature}
- Logic: {logic description or pseudo-code}

**For Forms / Form Extensions:**
- Datasources: {table} | {join type}
- Added controls: {type} | {name} | {datasource.field}
- Methods/overrides: {datasource.method or control.event} | {logic}

**For Data Entities:**
- Root datasource: {table}
- Exposed fields: {name} | {source field}
- Staging table: {yes/no} | {name}

**For Enums / EDTs:**
- (Enum) Values: {name} | {label} | {integer value}
- (EDT) Base type: {str|int|real|enum|...} | {size} | {referenced enum}

**For Security (Privilege → Duty → Role):**
- Entry point: {menu item name} | {type} | {grant: Read|Update|Create|Delete}
- Duty: {name} | {included privileges}
- Role: {name} | {included duties}

#### TDD Evidence
- {Ninja MCP tool} → {summarised result that justifies the decision}
```

> **Notes on the schema**:
> - The TDD Agent generates only the relevant sections (not every customisation has all types)
> - The **object order** follows the 14-step hierarchy from Step 3 (Labels → Enums → EDTs → ... → Security)
> - The **Dependencies in this prompt** field allows the X++ Agent to validate the generation order
> - The **TDD Evidence** field preserves the traceability of design decisions from Ninja MCP
> - The developer reviews the Execution Prompt together with the TDD before invoking the X++ Agent

9. **Lessons Learned Loop** — Continuous learning process:
   - Developer reviews the TDD and saves it with a versioned suffix (`_rev_1`, `_rev_2`, etc.)
   - The agent compares the original TDD with the revisions and generates structured **Lessons Learned**
   - Updates the lessons learned file on GitHub (part of the agent's **Skills / Playbook**)
   - Creates a **Pull Request** for the Tech Lead's approval
   - Once merged, the agent uses the lessons in future generations (closed feedback loop)

### Playbook / Skills Structure (Lessons Learned)

| Category | Example |
|----------|---------|
| **Extensibility Decisions** | "Prefer event handler over CoC when X scenario" |
| **Object Mapping** | "For credit validation, use the CustCreditLimit framework, do not create custom" |
| **Naming & Patterns** | "In this project, use _Handler suffix for event handler classes" |
| **Common Mistakes** | "Do not extend SalesConfirmJournalPost.postJournal, use the onPosted event" |

> **Suggestion**: Extend the Lessons Learned mechanism to Steps 1 and 3 — FDD (lessons about Standard/GAP classification) and X++ (lessons about compilation/BP fixes).

### MCP Servers — Step 2

| MCP Server | Purpose |
|------------|---------|
| **Azure DevOps MCP** | Reads WI/FDD, creates SubTask, uploads TDD, posts comments |
| **Ninja MCP Server (hybrid)** | Full context across all layers (see detail below) |
| **MS Learn MCP** | Extensibility best practices and D365FO patterns |

### Ninja MCP Server — Hybrid Mode (Detail)

The agent has **full visibility across all D365FO layers** by combining two sources:

| Source | Content | Availability |
|--------|---------|--------------|
| **Azure DB + Blob Storage** | All Microsoft standard code (584K+ indexed AOT objects), ISVs (DocCentric, etc.), internal project assets | Remote — no dependency on the local machine |
| **Local File Search** | Project's custom model on the developer's machine | Local — captures recent changes |

**Result**: Standard MS → ISV → Internal Assets → Custom Model

**Sync**: The nightly build machine on Azure DevOps pulls the branch → indexes → updates Azure DB/Blob. Local file search covers the 1-day gap.

Ref: https://github.com/dynamics365ninja/d365fo-mcp-server

### Ninja MCP Tools Used by the TDD Agent

| Need | Tool | Returns |
|------|------|---------|
| Which object to extend? | `search` / `batch_search` | Search across 584K+ symbols |
| What can be extended? | `analyze_extension_points` | CoC-eligible methods, delegates, events |
| CoC or event handler? | `recommend_extension_strategy` | Best extensibility mechanism |
| Does a CoC already exist on this method? | `find_coc_extensions` | Existing wrappers |
| Does an event handler already exist? | `find_event_handlers` | Handlers with classification |
| How do others implement this? | `analyze_code_patterns` / `suggest_method_implementation` | Real patterns from the codebase |
| Exact signature? | `get_method_signature` / `get_method_source` | Signature + source code |
| Which EDT to use? | `suggest_edt` | Suggestions with confidence score |
| Fields and relations? | `get_table_info` | Schema: fields, indexes, FK |
| Best practices? | `get_xpp_knowledge` | D365FO knowledge base |

### Attention Points — Step 2 (All Resolved)

| Point | Status | Mitigation |
|-------|--------|------------|
| Inconsistent detail level | ✅ | Manifesto + Lessons Learned Loop + co-authoring mode |
| Ninja MCP hybrid sync | ✅ | Local file search (priority) + nightly sync via build machine |
| Mapping without mockups | ✅ | Manual mockups (current) + Clarification Process + automatic mockups (future) |
| Decisions without evidence | ✅ | Ninja MCP provides structured evidence (54 tools) — agent cites sources |
| TDD as a contract for Step 3 | ✅ | Manifesto defines the minimum required level + Execution Prompt as executable contract |
| FDD→TDD traceability | ✅ | Document naming + native DevOps links (via Azure DevOps MCP) |

---

## Step 3: X++ Code Generation

**Status**: 🔲 TO IMPLEMENT

### Workflow

1. **Pre-requisite: Developer creates Solution and Projects** — The developer manually creates the VS Solution (.sln) and the D365FO projects (.rnrproj) inside it, linking each project to the corresponding model. Reason: Ninja MCP **does not create** Solutions or .rnrproj files — it only creates/modifies AOT objects inside existing projects.

2. **Trigger** — The developer invokes the agent via a prompt in VS Code Copilot (or Visual Studio).

3. **Execution Prompt Consumption** — Step 2 (TDD Agent) generates, together with the TDD, an **Execution Prompt file** for the X++ Agent and attaches it to DevOps. This prompt contains specific instructions for each object to create/extend (e.g. "create class ABC_SalesValidation, extend method validateWrite via CoC with this signature, fields X, Y, Z on table W").

4. **Local Context Analysis** — Before generating:
   - Reads local config (.md) for the VS project folder and target model
   - Queries **Ninja MCP** for full metadata of the standard objects to extend
   - Runs **local file search** to check for objects that already exist in the model (to avoid duplication/conflicts)

5. **Generation Plan with Confirmation** — The agent analyses the Execution Prompt and **lists all the object types** that will be generated, organised by the dependency hierarchy below. It presents the plan to the developer and **waits for confirmation** before starting generation.

6. **Hierarchical Object Generation** — Generates objects following D365FO's compilation dependency order. Each step can be confirmed individually:

   | Order | Object Type | Reason for Position | Ninja MCP Tool |
   |-------|-------------|---------------------|----------------|
   | 1 | **Labels** | No dependencies — everything references them | `create_label` (search first with `search_labels`) |
   | 2 | **Base Enums + Enum Extensions** | EDTs can have an EnumType that references enums | `create_d365fo_file` (enum, enum-extension) |
   | 3 | **EDTs + EDT Extensions** | After enums; tables/fields reference EDTs | `create_d365fo_file` (edt, edt-extension) / `suggest_edt` |
   | 4 | **Tables + Fields + Indexes + Relations + Field Groups** | After EDTs/Enums — the foundation of all data structures | `create_d365fo_file` (table) / `generate_smart_table` |
   | 5 | **Table Extensions** | Added fields may be referenced by views, entities, forms | `create_d365fo_file` (table-extension) / `modify_d365fo_file` |
   | 6 | **Views** | Reference tables as datasource | `create_d365fo_file` (view) |
   | 7 | **Queries** | Reference tables and views | `create_d365fo_file` (query) |
   | 8 | **Data Entities + Data Entity Extensions** | After tables, views and queries (used as datasource) | `create_d365fo_file` (data-entity, data-entity-extension) |
   | 9 | **Classes** (CoC, Event Handlers, Batch Jobs, SysOperation, helpers) | After tables/EDTs/enums — business logic | `generate_code` / `create_d365fo_file` (class, class-extension) |
   | 10 | **Forms + Form Extensions** | After tables and classes (datasources + logic) | `create_d365fo_file` (form, form-extension) / `generate_smart_form` |
   | 11 | **Reports** (SSRS: TmpTable → Contract → DP → Controller → AxReport) | After tables and classes (5 interdependent objects) | `generate_smart_report` |
   | 12 | **Menu Items** (Display, Action, Output + Extensions) | After forms, classes, reports (point to them) | `create_d365fo_file` (menu-item-*) |
   | 13 | **Menus + Menu Extensions** | After menu items | `create_d365fo_file` (menu, menu-extension) |
   | 14 | **Security** (Privileges → Duties → Roles) | Last — privileges reference entry points (menu items) | `create_d365fo_file` (security-privilege, security-duty, security-role) |

   > **Note**: Not all steps are needed for every customisation. The agent generates only the relevant steps based on the Execution Prompt. Steps can be grouped or skipped with the developer's confirmation.

7. **Build Validation Loop** — After each step (or group of steps):
   - Compiles locally via `build_d365fo_project` (Ninja MCP)
   - Analyses compilation errors via `get_d365fo_error_help`
   - Runs best practice checks via `run_bp_check`
   - **Tries to fix errors and BP warnings automatically**
   - If it cannot fix: asks the developer
   - Asks the developer whether to update the **Lessons Learned** of the X++ Agent with the solution

8. **Output** — Generated files in the developer's local model folder. The developer creates the PR on GitHub manually.

### Lessons Learned — X++ Generator

- Scope: **auto-corrections of compilation and BP warnings** + developer feedback on how to fix errors
- Different from Code Review (Step 4, which covers architectural best practices)
- Examples:
  - "When using CoC on validateWrite, always call next first before custom validation"
  - "BP warning X resolved by adding SysObsoleteAttribute"
  - "Compilation error Y on form extension: use formDataSourceStr() instead of string literal"

### MCP Servers — Step 3

| MCP Server | Purpose |
|------------|---------|
| **Azure DevOps MCP** | Downloads the Execution Prompt and TDD |
| **Ninja MCP Server (hybrid)** | Object generation, metadata, build, BP check, file operations |
| **MS Learn MCP** | Best practices lookup when encountering errors |

### Ninja MCP Tools Used by the X++ Generator

| Action | Tool |
|--------|------|
| Create AOT objects | `create_d365fo_file` (26+ supported types) |
| Modify objects | `modify_d365fo_file` (25+ operations) |
| Generate boilerplate | `generate_code` (CoC, batch, SysOperation, event handler, etc.) |
| Generate smart table | `generate_smart_table` |
| Generate smart form | `generate_smart_form` |
| Generate smart SSRS report | `generate_smart_report` |
| Generate AOT XML | `generate_d365fo_xml` |
| Look up exact signatures | `get_method_signature` |
| Check existing CoC | `find_coc_extensions` |
| Suggest EDT | `suggest_edt` |
| Create labels | `create_label` (searches existing ones first with `search_labels`) |
| Compile | `build_d365fo_project` |
| Error diagnostics | `get_d365fo_error_help` |
| Best practice check | `run_bp_check` |
| DB sync | `trigger_db_sync` |
| Verify project | `verify_d365fo_project` |
| Update index | `update_symbol_index` |
| Undo change | `undo_last_modification` |

### Attention Points — Step 3

| Point | Type | Status | Detail |
|-------|------|--------|--------|
| Code does not compile | Risk | ✅ Mitigated | Build Validation Loop: compile → diagnose → fix → ask the developer |
| Malformed metadata XML | Risk | ✅ Mitigated | Ninja MCP generates XML via C# bridge (IMetadataProvider) |
| Conflicts with existing objects | Risk | ✅ Mitigated | Local file search is mandatory; Ninja MCP verifies the project |
| Ignored BP warnings | Risk | ✅ Mitigated | `run_bp_check` is integrated into the validation loop |
| ISV context | Risk | Partial | Ninja MCP indexes ISVs, but extension points may be limited |
| Execution Prompt (Step 2→3) | Design | ✅ | Structured Markdown with sections per object — schema defined in Step 2 |
| Incremental generation | Suggestion | — | Generating 1 object at a time makes debugging easier; Build Validation confirms each step |

---

## Step 4: Code Review Agent

**Status**: 🔲 TO IMPLEMENT

### Workflow

1. **Trigger** — To be defined: GitHub Action on the PR (*research feasibility*) or manual invocation by the developer/tech lead.

2. **PR Consumption** — Analyses the code diff in the GitHub Pull Request.

3. **Validation against best practices**:
   - Queries **MS Learn MCP** to validate against official patterns
   - Queries best practice reference sources specified in the **Manifesto / Context**
   - Queries the Code Review Agent's **Playbook / Skills** (lessons learned from previous reviews)

4. **Output** — Posts detailed comments on the GitHub PR with:
   - Issues found (naming, patterns, security, performance)
   - Fix suggestions
   - Reference to the violated best practice

5. **Human checkpoint** — The Tech Lead evaluates the comments and gives the final PR approval.

### Lessons Learned — Code Review Agent

- Scope: **architectural best practices** synthesised from the Tech Lead's comments on PRs
- The agent analyses the Tech Lead's PR comments + code in context → synthesises them into rules for future PRs
- Generates a PR on GitHub updating the Playbook for the Tech Lead's approval
- Examples:
  - "Always validate TTS level before calling an external service"
  - "Prefer set-based operations over row-by-row for tables with > 1,000 records"
  - "Security: never use unchecked(classStr()) in privilege assignments"

### MCP Servers — Step 4

| MCP Server | Purpose |
|------------|---------|
| **MS Learn MCP** | Official best practices |
| **GitHub MCP** | Reads PR, posts comments (*trigger via GitHub Action to be confirmed*) |
| **Manifesto / Context** | Project-specific custom rules |

### Attention Points — Step 4

| Point | Type | Status | Detail |
|-------|------|--------|--------|
| Code Review trigger | Research | 🔲 | Investigate GitHub Actions to invoke the agent automatically on a PR |
| Review scope | Design | 🔲 | Define the validation checklist (naming, security, performance, patterns) |

---

## Knowledge Sources Architecture

| Source | Type | Content | Steps |
|--------|------|---------|-------|
| MS Learn MCP | External | Official MS documentation, best practices | 1, 2, 3, 4 |
| Ninja MCP Server (hybrid) | Custom | 584K+ indexed AOT objects (standard/ISV/assets) + local model | 2, 3 |
| D365FO MCP Server (MS official) | External | Standard form layouts for mockups | 1 |
| Azure DevOps MCP | External | Work items, FDD/TDD, comments, attachments | 1, 2, 3 |
| Manifesto / Constitution | Local | Project rules (prefix, naming, detail level) | 2, 3, 4 |
| Playbook / Skills (Lessons Learned) | GitHub | Categorised lessons learned per agent | 2, 3, 4 |
| Local Config (.md) | Local | VS project folder, target model | 3 |

### Playbooks per Agent

| Playbook | Agent | Lessons Scope |
|----------|-------|---------------|
| **TDD Playbook** | Step 2 — TDD Generator | Extensibility decisions, object mapping, naming, design mistakes |
| **X++ Generator Playbook** | Step 3 — X++ Generator | Compilation fixes, BP warnings, code patterns |
| **Code Review Playbook** | Step 4 — Code Review | Architectural best practices synthesised from Tech Lead PRs |
| FDD Playbook *(future)* | Step 1 — FDD Generator | Standard/GAP classification, recurring clarifications |

---

## Captured Decisions

- **Platform**: GitHub Copilot Agent Mode
- **Approach**: Human-in-the-loop with checkpoints between each step
- **Templates**: Word today, migration to Markdown planned
- **FDD/TDD output**: Azure DevOps work items (attachments + SubTasks)
- **TDD→X++ output**: Execution Prompt generated by the TDD Agent, attached to DevOps as an executable contract
- **X++ output**: Local model folder (developer creates the PR manually)
- **Code Review**: Separate agent (Step 4) on GitHub PR, Tech Lead approves
- **Lessons Learned**: Playbook per agent on GitHub with PR for Tech Lead approval
- **Ninja MCP sync**: Nightly build machine + local file search with priority
- **Mockups**: Manual by consultants (current), automatic via the MS D365FO MCP Server (future)
- **Solution/Projects**: Created manually by the developer before invoking the X++ Agent (Ninja MCP does not create .sln/.rnrproj — only AOT objects inside existing projects)

---

## Pending Items (Cross-Step)

| # | Item | Options | Recommendation |
|---|------|---------|----------------|
| 1 | **Orchestration between steps** — How will the handoff Step 1→2→3→4 be managed? | (A) Manual — developer invokes each agent; (B) Semi-auto — agent notifies when the previous step is approved; (C) DevOps pipeline trigger | Start with (A), evolve to (B) |
| 2 | **Template versioning** — Templates change, old FDDs become inconsistent | Version templates together with the Playbook | To be defined |
| 3 | **Reverse feedback loop** — Code Review (Step 4) finds architectural issues: should it feed back into the TDD? | (A) Only fix the code; (B) Feed back into TDD + Lessons Learned | (B) via Lessons Learned |
| 4 | **Multi-model** — Project with integration + functional models | ✅ Developer creates Solution + projects per model; Ninja MCP accepts `packageName`/`projectPath` per tool call | Decided |
| 5 | **Partial regeneration** — Redo 1 of 5 customisations without affecting the others | Granular FDD→TDD→code traceability | Feasible with DevOps links |
| 6 | **Execution Prompt (Step 2→3)** — Define schema/format | ✅ Structured Markdown with sections per object (schema defined in Step 2) | Decided |
| 7 | **Code Review trigger (Step 4)** — How to invoke the agent on a PR? | GitHub Action vs. manual invocation | 🔲 Research |

---

*Next step: discuss pending cross-step items and/or start implementing Step 2*
