# Framework de Implementação D365FO com Agentes de IA

> **Status**: Draft v2 — 28/Mar/2026  
> **Plataforma**: GitHub Copilot Agent Mode  
> **Escopo**: Projetos D365 FnO (SCM / HR)  
> **Approach**: Human-in-the-loop com checkpoints entre cada fase

## TL;DR

Framework reutilizável que acelera, padroniza e aumenta a performance de consultores e desenvolvedores D365FO usando 4 agentes de IA em fases sequenciais: **FDD → TDD → X++ → Code Review**. Agentes consomem conhecimento via MS Learn MCP, Ninja MCP Server (584k+ objetos indexados), D365FO MCP Server oficial da Microsoft, e manifesto do arquiteto. Output de FDD/TDD vai para Azure DevOps; X++ vai para a model local do desenvolvedor; Code Review opera nos PRs do GitHub.

### Visão Geral das Fases

| Fase | Agente | Status | Input | Output |
|------|--------|--------|-------|--------|
| 1 | FDD Generator | ✅ Implementado | WI do DevOps (PBI/CR) | FDD anexada ao DevOps |
| 2 | TDD Generator | 🔲 A implementar | FDD do DevOps | TDD + Prompt Detalhado anexados ao DevOps |
| 3 | X++ Code Generator | 🔲 A implementar | Prompt Detalhado do DevOps | Código X++ na model local |
| 4 | Code Review Agent | 🔲 A implementar | PR no GitHub | Comentários no PR |

---

## Fase 1: Geração do FDD (Functional Design Document)

**Status**: ✅ IMPLEMENTADO

### Workflow

1. **Input via Azure DevOps** — Agente lê o Work Item (PBI, CR, ou outro tipo) e entende o Business Requirement. Analisa todos os comentários do ticket para contexto completo.
   - Para **Change Requests**: faz download e analisa a FDD do parent WI
   - Suporta texto livre como input alternativo

2. **Consulta MS Learn** — Verifica o que pode ser atendido de forma standard pelo D365FO para minimizar customizações.

3. **Classificação de requisitos**:
   - **Standard**: atendido nativamente
   - **Configuração**: atendido via setup — *gera guia de configuração do D365FO*
   - **Customização (GAP)**: requer desenvolvimento

4. **Clarification Requirement Process**:
   - Pergunta ao consultor que está executando o agente
   - Mitiga assumptions ativamente
   - Se o consultor não pode responder: escreve comentário no DevOps marcando o usuário responsável da área
   - Quando o responsável responde, o consultor reinicia — o agente recupera as clarificações via comentários do ticket

5. **Output** — Anexa a FDD no ticket do DevOps (Word ou Markdown)

6. **Modos de operação**:
   - **Autônomo**: gera documento completo e anexa ao DevOps para review
   - **Co-autoria**: consultor revisa tópico a tópico antes do documento final

### MCP Servers — Fase 1

| MCP Server | Função |
|------------|--------|
| **Azure DevOps MCP** | Leitura de WI, comentários, upload/download de anexos, marcação de usuários |
| **MS Learn MCP** | Documentação oficial para classificação standard/config/gap |
| **D365FO MCP Server (MS oficial)** | *Será introduzido* para geração de mockups (layout dos forms standard + campos/botões da customização) |

### Capacidades confirmadas

- Leitura de WI + comentários do DevOps
- Download de FDD (Word) do parent WI para contexto em CRs
- Classificação Standard/Config/GAP com guia de configuração para itens config-only
- Clarification process assíncrono via comentários DevOps
- Flexível para diferentes estruturas de WI (PBI, CR, etc.)
- Anexação de FDD no ticket (Word/Markdown)

### Pontos de Atenção — Fase 1

| Ponto | Tipo | Detalhe |
|-------|------|---------|
| Classificação Standard vs GAP | Risco | Agente pode errar. Validação com evidência do MS Learn mitiga parcialmente |
| Clarificações sem resposta | Risco | Se o responsável não responde no DevOps, FDD fica bloqueada. Considerar timeout/escalation |
| Rastreabilidade de decisões | Sugestão | Cada classificação e clarificação deveria ser logada na FDD para auditoria |
| Versionamento de FDD em CRs | Sugestão | Nova FDD deveria referenciar o que mudou vs. FDD original (diff semântico) |

---

## Fase 2: Geração do TDD (Technical Design Document)

**Status**: 🔲 A IMPLEMENTAR

### Workflow

1. **Trigger** — Desenvolvedor invoca o agente via prompt no VS Code Copilot (ou Visual Studio). O agente identifica o WI no DevOps e localiza a FDD na Task irmã.

2. **Consumo da FDD** — Download/leitura da FDD do DevOps (via Azure DevOps MCP). Extrai todas as customizações (GAPs) que precisam de design técnico.
   - *Futuro*: consumir mockups da FDD para mapear campos/botões a objetos AOT

3. **Análise técnica via Ninja MCP** — Para cada customização:
   - Consulta Ninja MCP Server (modo híbrido) para identificar objetos AOT a estender/criar
   - Identifica: tabelas, campos, forms, classes, data entities, extension points
   - Quando não tem certeza: inicia **Clarification Process** com o desenvolvedor
   - **Cita evidência** do Ninja MCP para cada decisão de design (ex: *"CoC em SalesTable.validateWrite() — baseado em: recommend_extension_strategy + analyze_extension_points + find_coc_extensions sem wrapper existente"*)

4. **Consulta MS Learn MCP** — Valida best practices de extensibilidade (CoC vs event handler, etc.)

5. **Aplicação do Manifesto/Constituição** — Arquivo local do arquiteto que define:
   - Prefixo da model (usado em métodos e objetos)
   - Prefixo ou sufixo para naming convention
   - Nível de detalhamento configurável: somente assinaturas + comentários **OU** scaffold completo
   - Regras específicas do projeto

6. **Clarification Process** — Similar à Fase 1: pergunta ao desenvolvedor pontos que demandam clarificação.

7. **Modos de operação**:
   - **Autônomo**: gera TDD completa, anexa ao DevOps, posta comentário pedindo review
   - **Co-autoria**: desenvolvedor revisa tópico a tópico, método a método

8. **Output** — Dois artefatos anexados à SubTask no DevOps:
   - **TDD** — documento técnico completo para review do desenvolvedor
   - **Prompt Detalhado para X++ Agent** — Markdown estruturado (schema abaixo) com instruções específicas para o agente da Fase 3. Serve como **contrato executável** entre Fase 2 e Fase 3.
   - Comentário no WI notificando que TDD está pronta para review.

### Schema do Prompt Detalhado (Fase 2 → Fase 3)

Formato: **Markdown estruturado com seções por objeto**, anexado ao DevOps como ficheiro `.md`.

```markdown
# Prompt Detalhado — X++ Agent

## Contexto
- **WI**: #{number} — {título}
- **Model primária**: {packageName}
- **Project path**: {caminho do .rnrproj}
- **Manifesto**: {versão do manifesto aplicado}
- **FDD ref**: {link ou nome do ficheiro FDD}
- **TDD ref**: {link ou nome do ficheiro TDD}

## Objetos (por ordem de geração)

### [Ordem] [Tipo] — [Nome completo com prefixo]

- **Ação**: create | extend
- **Tipo AOT**: table | class | form | enum | edt | view | query | data-entity | menu-item-display | menu-item-action | security-privilege | ...
- **Package/Model**: {packageName} (só se diferente da model primária)
- **Objeto base** *(só para extensions)*: {nome do objeto standard/ISV a estender}
- **Dependências neste prompt**: {lista de objetos deste prompt que devem existir antes}

#### Especificação

*(conteúdo varia por tipo — exemplos abaixo)*

**Para Tabelas/Table Extensions:**
- Campos: {nome} | {EDT} | {mandatory?} | {descrição}
- Indexes: {nome} | {campos} | {unique?}
- Relations: {nome} | {tabela foreign} | {campos}
- Field Groups: {nome} | {campos}
- Métodos: {assinatura} | {lógica esperada em comentário}

**Para Classes (CoC/Event Handler/Batch/SysOperation):**
- Tipo: CoC | EventHandler | BatchJob | SysOperationService | Helper
- Objeto alvo *(CoC/EH)*: {classe.método ou tabela.método}
- Extension point *(CoC/EH)*: {CoC wrap | pre-event | post-event | delegate}
- Assinatura: {assinatura completa do método}
- Lógica: {descrição da lógica ou pseudo-código}

**Para Forms/Form Extensions:**
- Datasources: {tabela} | {join type}
- Controles adicionados: {tipo} | {nome} | {datasource.field}
- Métodos/overrides: {datasource.método ou control.evento} | {lógica}

**Para Data Entities:**
- Root datasource: {tabela}
- Campos expostos: {nome} | {source field}
- Staging table: {sim/não} | {nome}

**Para Enums/EDTs:**
- (Enum) Valores: {nome} | {label} | {valor inteiro}
- (EDT) Tipo base: {str|int|real|enum|...} | {tamanho} | {enum referenciado}

**Para Security (Privilege → Duty → Role):**
- Entry point: {menu item name} | {tipo} | {grant: Read|Update|Create|Delete}
- Duty: {nome} | {privileges incluídos}
- Role: {nome} | {duties incluídos}

#### Evidência TDD
- {tool Ninja MCP} → {resultado resumido que justifica a decisão}
```

> **Notas sobre o schema**:
> - O TDD Agent gera apenas as seções relevantes (nem toda customização tem todos os tipos)
> - A **ordem dos objetos** segue a hierarquia de 14 steps da Fase 3 (Labels → Enums → EDTs → ... → Security)
> - O campo **Dependências neste prompt** permite ao X++ Agent validar a ordem de geração
> - O campo **Evidência TDD** preserva a rastreabilidade das decisões de design do Ninja MCP
> - O dev revisa o Prompt Detalhado junto com a TDD antes de invocar o X++ Agent

9. **Lessons Learned Loop** — Processo de aprendizado contínuo:
   - Dev revisa TDD e salva com sufixo versionado (`_rev_1`, `_rev_2`, etc.)
   - Agente compara TDD original vs. revisões e gera **Lessons Learned** estruturado
   - Atualiza arquivo de lessons learned no GitHub (parte do **Skills / Playbook** do agente)
   - Gera **Pull Request** para aprovação do Tech Lead
   - Uma vez mergeado, agente usa lições nas próximas gerações (feedback loop fechado)

### Estrutura do Playbook / Skills (Lessons Learned)

| Categoria | Exemplo |
|-----------|---------|
| **Extensibility Decisions** | "Preferir event handler sobre CoC quando X cenário" |
| **Object Mapping** | "Para validação de crédito, usar framework CustCreditLimit, não criar custom" |
| **Naming & Patterns** | "Neste projeto, usar sufixo _Handler para event handler classes" |
| **Common Mistakes** | "Não estender SalesConfirmJournalPost.postJournal, usar evento onPosted" |

> **Sugestão**: Estender o mecanismo de Lessons Learned para as Fases 1 e 3 — FDD (lições sobre classificação standard/gap) e X++ (lições sobre correções de compilação/BP).

### MCP Servers — Fase 2

| MCP Server | Função |
|------------|--------|
| **Azure DevOps MCP** | Leitura de WI/FDD, criação de SubTask, upload de TDD, comentários |
| **Ninja MCP Server (híbrido)** | Contexto completo de todos os layers (ver detalhe abaixo) |
| **MS Learn MCP** | Best practices de extensibilidade e padrões D365FO |

### Ninja MCP Server — Modo Híbrido (detalhe)

O agente tem **visibilidade completa de todos os layers D365FO** combinando duas fontes:

| Fonte | Conteúdo | Disponibilidade |
|-------|----------|-----------------|
| **Azure DB + Blob Storage** | TODO o código standard Microsoft (584k+ objetos AOT indexados), ISVs (DocCentric, etc.), internal assets do projeto | Remoto — sem depender da máquina local |
| **File Search Local** | Custom model do projeto na máquina do dev | Local — captura mudanças recentes |

**Resultado**: Standard MS → ISV → Internal Assets → Custom Model

**Sincronia**: Build machine nightly no Azure DevOps puxa branch → indexa → atualiza Azure DB/Blob. File search local cobre o gap de 1 dia.

Ref: https://github.com/dynamics365ninja/d365fo-mcp-server

### Ferramentas Ninja MCP usadas pelo TDD Agent

| Necessidade | Tool | Retorno |
|-------------|------|---------|
| Qual objeto estender? | `search` / `batch_search` | Busca em 584k+ símbolos |
| O que pode ser estendido? | `analyze_extension_points` | Métodos CoC-eligible, delegates, eventos |
| CoC ou event handler? | `recommend_extension_strategy` | Melhor mecanismo de extensibilidade |
| Já existe CoC nesse método? | `find_coc_extensions` | Wrappers existentes |
| Já existe event handler? | `find_event_handlers` | Handlers com classificação |
| Como outros implementam? | `analyze_code_patterns` / `suggest_method_implementation` | Padrões reais do codebase |
| Assinatura exata? | `get_method_signature` / `get_method_source` | Assinatura + código fonte |
| Qual EDT usar? | `suggest_edt` | Sugestões com confidence score |
| Campos e relações? | `get_table_info` | Schema: campos, indexes, FK |
| Best practices? | `get_xpp_knowledge` | Knowledge base D365FO |

### Pontos de Atenção — Fase 2 (todos resolvidos)

| Ponto | Status | Mitigação |
|-------|--------|-----------|
| Nível de detalhe inconsistente | ✅ | Manifesto + Lessons Learned Loop + modo co-autoria |
| Sincronia Ninja MCP híbrido | ✅ | File search local (prioridade) + nightly sync via build machine |
| Mapeamento sem mockups | ✅ | Mockups manuais (atual) + Clarification Process + mockups automáticos (futuro) |
| Decisões sem evidência | ✅ | Ninja MCP fornece evidência estruturada (54 tools) — agente cita fontes |
| TDD como contrato para Fase 3 | ✅ | Manifesto define nível mínimo obrigatório + Prompt Detalhado como contrato executável |
| Rastreabilidade FDD→TDD | ✅ | Naming no documento + links nativos DevOps (via Azure DevOps MCP) |

---

## Fase 3: Geração de Código X++

**Status**: 🔲 A IMPLEMENTAR

### Workflow

1. **Pré-requisito: Dev cria Solution e Projetos** — O desenvolvedor cria manualmente a VS Solution (.sln) e os D365FO projects (.rnrproj) dentro dela, linkando cada projeto à model correspondente. Motivo: Ninja MCP **não cria** Solutions nem ficheiros .rnrproj — apenas cria/modifica objetos AOT dentro de projetos existentes.

2. **Trigger** — Desenvolvedor invoca o agente via prompt no VS Code Copilot (ou Visual Studio).

3. **Consumo do Prompt Detalhado** — A Fase 2 (TDD Agent) gera, junto com a TDD, um **arquivo de prompt detalhado** para o X++ Agent e anexa ao DevOps. Este prompt contém instruções específicas para cada objeto a criar/estender (ex: "criar classe ABC_SalesValidation, estender método validateWrite via CoC com esta assinatura, campos X, Y, Z na tabela W").

4. **Análise de contexto local** — Antes de gerar:
   - Lê config local (.md) para pasta do VS project e model target
   - Consulta **Ninja MCP** para metadados completos dos objetos standard a estender
   - Faz **file search local** para verificar objetos já existentes na model (evitar duplicação/conflitos)

5. **Plano de geração com confirmação** — O agente analisa o Prompt Detalhado e **elenca todos os tipos de objetos** que serão gerados, organizados pela hierarquia de dependências abaixo. Apresenta o plano ao desenvolvedor e **aguarda confirmação** antes de iniciar a geração.

6. **Geração hierárquica de objetos** — Gera objetos seguindo a ordem de dependências de compilação do D365FO. Cada step é confirmável individualmente:

   | Ordem | Tipo de Objeto | Motivo da posição | Ninja MCP Tool |
   |-------|---------------|-------------------|----------------|
   | 1 | **Labels** | Sem dependências — tudo os referencia | `create_label` (busca antes com `search_labels`) |
   | 2 | **Base Enums + Enum Extensions** | EDTs podem ter EnumType que referencia enums | `create_d365fo_file` (enum, enum-extension) |
   | 3 | **EDTs + EDT Extensions** | Após enums; tabelas/campos referenciam EDTs | `create_d365fo_file` (edt, edt-extension) / `suggest_edt` |
   | 4 | **Tables + Fields + Indexes + Relations + Field Groups** | Após EDTs/Enums — base de toda estrutura de dados | `create_d365fo_file` (table) / `generate_smart_table` |
   | 5 | **Table Extensions** | Campos adicionados podem ser referenciados por views, entities, forms | `create_d365fo_file` (table-extension) / `modify_d365fo_file` |
   | 6 | **Views** | Referenciam tabelas como datasource | `create_d365fo_file` (view) |
   | 7 | **Queries** | Referenciam tabelas e views | `create_d365fo_file` (query) |
   | 8 | **Data Entities + Data Entity Extensions** | Após tabelas, views e queries (usam como datasource) | `create_d365fo_file` (data-entity, data-entity-extension) |
   | 9 | **Classes** (CoC, Event Handlers, Batch Jobs, SysOperation, helpers) | Após tabelas/EDTs/enums — lógica de negócio | `generate_code` / `create_d365fo_file` (class, class-extension) |
   | 10 | **Forms + Form Extensions** | Após tabelas e classes (datasources + lógica) | `create_d365fo_file` (form, form-extension) / `generate_smart_form` |
   | 11 | **Reports** (SSRS: TmpTable → Contract → DP → Controller → AxReport) | Após tabelas e classes (5 objetos interdependentes) | `generate_smart_report` |
   | 12 | **Menu Items** (Display, Action, Output + Extensions) | Após forms, classes, reports (apontam para eles) | `create_d365fo_file` (menu-item-*) |
   | 13 | **Menus + Menu Extensions** | Após menu items | `create_d365fo_file` (menu, menu-extension) |
   | 14 | **Security** (Privileges → Duties → Roles) | Por último — privileges referenciam entry points (menu items) | `create_d365fo_file` (security-privilege, security-duty, security-role) |

   > **Nota**: Nem todos os steps são necessários em toda customização. O agente gera apenas os steps relevantes conforme o Prompt Detalhado. Steps podem ser agrupados ou pulados com confirmação do dev.

7. **Build Validation Loop** — Após cada step (ou grupo de steps):
   - Compila localmente via `build_d365fo_project` (Ninja MCP)
   - Analisa erros de compilação via `get_d365fo_error_help`
   - Roda best practice checks via `run_bp_check`
   - **Tenta resolver erros e BP warnings automaticamente**
   - Se não consegue resolver: pergunta ao desenvolvedor
   - Pergunta ao dev se deseja atualizar o **Lessons Learned** do X++ Agent com a solução

8. **Output** — Arquivos gerados na pasta da model local do desenvolvedor. Dev faz PR manualmente no GitHub.

### Lessons Learned — X++ Generator

- Escopo: **auto-correções de compilação e BP warnings** + feedbacks do dev sobre como resolver erros
- Diferente do Code Review (Fase 4, que cobre melhores práticas arquiteturais)
- Exemplos:
  - "Quando CoC em validateWrite, sempre chamar next primeiro antes de validação custom"
  - "BP warning X resolvido adicionando SysObsoleteAttribute"
  - "Erro de compilação Y em form extension: usar formDataSourceStr() em vez de string literal"

### MCP Servers — Fase 3

| MCP Server | Função |
|------------|--------|
| **Azure DevOps MCP** | Download do prompt detalhado e TDD |
| **Ninja MCP Server (híbrido)** | Geração de objetos, metadados, build, BP check, file operations |
| **MS Learn MCP** | Consulta de best practices quando encontra erros |

### Ferramentas Ninja MCP usadas pelo X++ Generator

| Ação | Tool |
|------|------|
| Criar objetos AOT | `create_d365fo_file` (26+ tipos suportados) |
| Modificar objetos | `modify_d365fo_file` (25+ operações) |
| Gerar boilerplate | `generate_code` (CoC, batch, SysOperation, event handler, etc.) |
| Gerar tabela inteligente | `generate_smart_table` |
| Gerar form inteligente | `generate_smart_form` |
| Gerar report SSRS inteligente | `generate_smart_report` |
| Gerar XML AOT | `generate_d365fo_xml` |
| Buscar assinaturas exatas | `get_method_signature` |
| Verificar CoC existentes | `find_coc_extensions` |
| Sugerir EDT | `suggest_edt` |
| Criar labels | `create_label` (busca existentes antes com `search_labels`) |
| Compilar | `build_d365fo_project` |
| Diagnóstico de erros | `get_d365fo_error_help` |
| Best practice check | `run_bp_check` |
| DB sync | `trigger_db_sync` |
| Verificar projeto | `verify_d365fo_project` |
| Atualizar índice | `update_symbol_index` |
| Desfazer mudança | `undo_last_modification` |

### Pontos de Atenção — Fase 3

| Ponto | Tipo | Status | Detalhe |
|-------|------|--------|---------|
| Código não compila | Risco | ✅ Mitigado | Build Validation Loop: compila → diagnostica → corrige → pergunta ao dev |
| Metadata XML mal formada | Risco | ✅ Mitigado | Ninja MCP gera XML via C# bridge (IMetadataProvider) |
| Conflitos com objetos existentes | Risco | ✅ Mitigado | File search local obrigatório; Ninja MCP verifica projeto |
| BP warnings ignorados | Risco | ✅ Mitigado | `run_bp_check` integrado no loop de validação |
| Contexto ISV | Risco | Parcial | Ninja MCP indexa ISVs, mas extension points podem ser limitados |
| Prompt detalhado (Fase 2→3) | Design | ✅ | Markdown estruturado com seções por objeto — schema definido na Fase 2 |
| Geração incremental | Sugestão | — | Gerar 1 objeto por vez facilita debug; Build Validation confirma cada step |

---

## Fase 4: Code Review Agent

**Status**: 🔲 A IMPLEMENTAR

### Workflow

1. **Trigger** — A ser definido: GitHub Action no PR (*pesquisar viabilidade*) ou invocação manual pelo dev/tech lead.

2. **Consumo do PR** — Analisa o diff do código no Pull Request do GitHub.

3. **Validação contra best practices**:
   - Consulta **MS Learn MCP** para validação de padrões oficiais
   - Consulta fontes de referência de melhores práticas especificadas no **Manifesto / Contexto**
   - Consulta **Playbook / Skills** do Code Review Agent (lessons learned de reviews anteriores)

4. **Output** — Deixa comentários detalhados no PR do GitHub com:
   - Issues encontradas (naming, patterns, security, performance)
   - Sugestões de correção
   - Referência à best practice violada

5. **Human checkpoint** — Tech Lead avalia comentários e dá aprovação final do PR.

### Lessons Learned — Code Review Agent

- Escopo: **melhores práticas arquiteturais** sintetizadas dos comentários do Tech Lead nos PRs
- O agente analisa os comentários do PR do Tech Lead + código em contexto → sintetiza em regras para PRs futuros
- Gera PR no GitHub atualizando o Playbook para aprovação do Tech Lead
- Exemplos:
  - "Sempre validar TTS level antes de chamar external service"
  - "Preferir set-based operations sobre row-by-row para tabelas com > 1000 registros"
  - "Security: nunca usar unchecked(classStr()) em privilege assignments"

### MCP Servers — Fase 4

| MCP Server | Função |
|------------|--------|
| **MS Learn MCP** | Best practices oficiais |
| **GitHub MCP** | Leitura de PR, comentários (*a confirmar trigger via GitHub Action*) |
| **Manifesto / Contexto** | Regras customizadas do projeto |

### Pontos de Atenção — Fase 4

| Ponto | Tipo | Status | Detalhe |
|-------|------|--------|---------|
| Code Review trigger | Pesquisa | 🔲 | Investigar GitHub Actions para invocar agente automaticamente no PR |
| Escopo do review | Design | 🔲 | Definir checklist de validação (naming, security, performance, patterns) |

---

## Arquitetura de Knowledge Sources

| Source | Tipo | Conteúdo | Fases |
|--------|------|----------|-------|
| MS Learn MCP | Externo | Documentação oficial MS, best practices | 1, 2, 3, 4 |
| Ninja MCP Server (híbrido) | Custom | 584k+ objetos AOT standard/ISV/assets indexados + model local | 2, 3 |
| D365FO MCP Server (MS oficial) | Externo | Layout de forms standard para mockups | 1 |
| Azure DevOps MCP | Externo | Work items, FDD/TDD, comentários, anexos | 1, 2, 3 |
| Manifesto/Constituição | Local | Regras do projeto (prefixo, naming, nível de detalhe) | 2, 3, 4 |
| Playbook/Skills (Lessons Learned) | GitHub | Lições aprendidas categorizadas por agente | 2, 3, 4 |
| Config Local (.md) | Local | Pasta do VS project, model target | 3 |

### Playbooks por Agente

| Playbook | Agente | Escopo das Lições |
|----------|--------|-------------------|
| **TDD Playbook** | Fase 2 — TDD Generator | Decisões de extensibilidade, object mapping, naming, erros de design |
| **X++ Generator Playbook** | Fase 3 — X++ Generator | Correções de compilação, BP warnings, padrões de código |
| **Code Review Playbook** | Fase 4 — Code Review | Melhores práticas arquiteturais sintetizadas dos PRs do Tech Lead |
| FDD Playbook *(futuro)* | Fase 1 — FDD Generator | Classificação standard/gap, clarificações recorrentes |

---

## Decisões Capturadas

- **Plataforma**: GitHub Copilot Agent Mode
- **Approach**: Human-in-the-loop com checkpoints entre cada fase
- **Templates**: Word hoje, migração para Markdown planejada
- **Output FDD/TDD**: Azure DevOps work items (anexos + SubTasks)
- **Output TDD→X++**: Prompt Detalhado gerado pelo TDD Agent, anexado ao DevOps como contrato executável
- **Output X++**: Pasta local da model (dev faz PR manualmente)
- **Code Review**: Agente separado (Fase 4) no GitHub PR, tech lead aprova
- **Lessons Learned**: Playbook por agente no GitHub com PR para aprovação do Tech Lead
- **Sincronia Ninja MCP**: Build machine nightly + file search local com prioridade
- **Mockups**: Manuais pelos consultores (atual), automáticos via D365FO MCP Server MS (futuro)
- **Solution/Projetos**: Criados manualmente pelo dev antes de invocar o X++ Agent (Ninja MCP não cria .sln/.rnrproj — apenas objetos AOT dentro de projetos existentes)

---

## Pontos Pendentes (Cross-Fase)

| # | Ponto | Opções | Recomendação |
|---|-------|--------|--------------|
| 1 | **Orquestração entre fases** — Como o handoff Fase 1→2→3→4 será gerenciado? | (A) Manual — dev invoca cada agente; (B) Semi-auto — agente notifica quando fase anterior aprovada; (C) Pipeline DevOps trigger | Começar com (A), evoluir para (B) |
| 2 | **Versionamento de templates** — Templates mudam, FDDs antigos ficam inconsistentes | Versionar templates junto com o Playbook | A definir |
| 3 | **Feedback loop reverso** — Code review (Fase 4) encontra problemas arquiteturais: retroalimenta TDD? | (A) Só corrige código; (B) Retroalimenta TDD + Lessons Learned | (B) via Lessons Learned |
| 4 | **Multi-model** — Projeto com models de integração + funcional | ✅ Dev cria Solution + projetos por model; Ninja MCP aceita `packageName`/`projectPath` por tool call | Decidido |
| 5 | **Regeneração parcial** — Refazer 1 de 5 customizações sem afetar as outras | Rastreabilidade FDD→TDD→código granular | Viável com DevOps links |
| 6 | **Prompt Detalhado (Fase 2→3)** — Definir schema/formato | ✅ Markdown estruturado com seções por objeto (schema definido na Fase 2) | Decidido |
| 7 | **Code Review trigger (Fase 4)** — Como invocar o agente no PR? | GitHub Action vs. invocação manual | 🔲 Pesquisar |

---

*Próximo passo: discutir pontos pendentes cross-fase e/ou iniciar implementação da Fase 2*
