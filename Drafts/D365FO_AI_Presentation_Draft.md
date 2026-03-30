# D365FO + AI Agents
## Delivery acelerada, padronizada e com qualidade embarcada

---

<!-- SLIDE 1 — ABERTURA / HOOK (30s) -->

## O Problema

Projetos D365FO sofrem dos mesmos problemas há anos:

> **FDDs inconsistentes** → **TDDs incompletas** → **Código que não compila** → **Code review manual e tardio**

Cada fase depende da qualidade da anterior. Quando a base falha, o efeito cascata é inevitável.

**E se cada fase tivesse um agente de IA especializado — com acesso ao código real do D365FO — para garantir qualidade desde o início?**

---

<!-- SLIDE 2 — A VISÃO (1 min) -->

## A Visão: 4 Agentes, 1 Pipeline

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Fase 1    │     │   Fase 2    │     │   Fase 3    │     │   Fase 4    │
│     FDD     │────▶│     TDD     │────▶│     X++     │────▶│ Code Review │
│  Generator  │     │  Generator  │     │  Generator  │     │    Agent    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
  Consultor           Desenvolvedor       Desenvolvedor       Tech Lead
  + AI Agent          + AI Agent          + AI Agent          + AI Agent

  Input:              Input:              Input:              Input:
  Work Item           FDD                 Prompt Detalhado    Pull Request
  (DevOps)            (DevOps)            (DevOps)            (GitHub)

  Output:             Output:             Output:             Output:
  FDD no DevOps       TDD + Prompt        Código X++          Comentários
                      Detalhado           na model local      no PR
```

**Human-in-the-loop**: o profissional decide, o agente executa e valida.

---

<!-- SLIDE 3 — O DIFERENCIAL (1 min) -->

## Por que funciona: Agentes com contexto real

Nossos agentes não "adivinham". Eles **consultam fontes reais** para cada decisão:

| Fonte de Conhecimento | O que fornece | Cobertura |
|----------------------|---------------|-----------|
| **Ninja MCP Server** | 584.000+ objetos AOT indexados (standard + ISV + custom) | Código real, não documentação |
| **MS Learn MCP** | Documentação oficial Microsoft | Best practices atualizadas |
| **D365FO MCP Server** (MS) | Layout de forms para mockups | Telas reais do sistema |
| **Azure DevOps MCP** | Work items, FDDs, TDDs, comentários | Contexto do projeto |

> **O agente cita evidência para cada decisão de design** — ex: *"CoC em SalesTable.validateWrite() — baseado em: recommend_extension_strategy + analyze_extension_points + find_coc_extensions sem wrapper existente"*

---

<!-- SLIDE 4 — O CICLO DE APRENDIZADO (1 min) -->

## Agentes que aprendem com o seu time

Cada agente tem um **Playbook** que evolui a cada iteração:

```
 Dev revisa TDD        Agente compara        Gera Lessons         Tech Lead
 gerada pelo       →   original vs.      →   Learned           →  aprova PR
 agente                 revisão               estruturado          no GitHub
                                                  │
                                                  ▼
                                          Agente usa lições
                                          nas próximas gerações
```

**Resultado**: quanto mais o time usa, melhor o agente fica.

Categorias do Playbook:
- **Extensibility Decisions** — quando usar CoC vs event handler
- **Object Mapping** — frameworks standard a reutilizar
- **Common Mistakes** — erros que o agente nunca mais repete

---

<!-- SLIDE 5 — FASE 1: FDD (conexões completas) -->

## Fase 1: FDD Generator — ✅ JÁ FUNCIONANDO

```
                         ┌──────────────────┐
                         │    Consultor      │ ◄─── Clarificações / Co-autoria
                         └────────┬─────────┘
                                  │ trigger + review
                                  ▼
┌──────────────┐      ┌──────────────────────┐      ┌──────────────────┐
│ Azure DevOps │◄────▶│   FDD Agent          │◄────▶│   MS Learn MCP   │
│     MCP      │      │  (Copilot Agent Mode)│      │                  │
│              │      └──────────┬───────────┘      └──────────────────┘
│ • Lê WI/PBI │                 │                   • Classifica Standard
│ • Lê comen- │                 │                     /Config/GAP
│   tários    │                 ▼                   • Best practices
│ • Download  │      ┌──────────────────────┐
│   FDD parent│      │  D365FO MCP Server   │
│ • Upload FDD│      │  (Microsoft oficial)  │
│ • Marca     │      │                      │
│   usuários  │      │ • Mockups de forms   │
└──────────────┘      │   (futuro)           │
                      └──────────────────────┘

  ┌────────────────────┐      ┌────────────────────┐
  │  📄 Templates FDD  │      │ 📘 FDD Playbook    │
  │  (GitHub repo)     │      │ (GitHub repo)       │
  │  versionados Git   │      │ Lessons Learned de  │
  └────────────────────┘      │ classificação S/C/G │
                              └────────────────────┘
```

**Modos**: Autônomo (gera tudo) ou Co-autoria (tópico a tópico)
**Clarificações**: se o consultor não sabe → agente posta no DevOps marcando o responsável da área

---

<!-- SLIDE 6 — FASE 2: TDD (conexões completas) -->

## Fase 2: TDD Generator — Próximo passo

```
                         ┌──────────────────┐
                         │  Desenvolvedor    │ ◄─── Clarificações / Co-autoria
                         └────────┬─────────┘
                                  │ trigger + review TDD
                                  ▼
┌──────────────┐      ┌──────────────────────┐      ┌──────────────────┐
│ Azure DevOps │◄────▶│   TDD Agent          │◄────▶│   MS Learn MCP   │
│     MCP      │      │  (Copilot Agent Mode)│      │                  │
│              │      └──────────┬───────────┘      └──────────────────┘
│ • Download   │                 │                   • Best practices
│   FDD        │                 │                     extensibilidade
│ • Cria Sub-  │                 ▼                   • CoC vs event
│   Task       │      ┌──────────────────────┐         handler
│ • Upload TDD │      │  Ninja MCP Server    │
│ • Upload     │      │  (modo híbrido)      │
│   Prompt Det.│      │                      │
│ • Comentários│      │ • 584k+ objetos AOT  │
└──────────────┘      │ • Azure DB (standard │
                      │   + ISV + assets)    │
  ┌────────────────┐  │ • File Search Local  │
  │ 📜 Manifesto/  │  │   (custom model)     │
  │ Constituição   │  │ • Evidência por      │
  │ (arquivo local │  │   decisão de design  │
  │  do arquiteto) │  └──────────────────────┘
  └────────────────┘
                      ┌────────────────────┐      ┌────────────────────┐
                      │  📄 Templates TDD  │      │ 📘 TDD Playbook    │
                      │  (GitHub repo)     │      │ (GitHub repo)       │
                      │  versionados Git   │      │ • Extensibility Dec.│
                      └────────────────────┘      │ • Object Mapping    │
                                                  │ • Naming & Patterns │
                      ┌────────────────────┐      │ • Common Mistakes   │
                      │ 🔄 Lessons Learned │      └────────────────────┘
                      │ Loop               │
                      │ Dev revisa TDD →   │
                      │ Agente compara →   │
                      │ Gera PR no GitHub →│
                      │ Tech Lead aprova   │
                      └────────────────────┘
```

**Output**: TDD + **Prompt Detalhado** (Markdown estruturado — contrato executável com schema por objeto para o X++ Agent)

---

<!-- SLIDE 7 — FASE 3: X++ (conexões completas) -->

## Fase 3: X++ Generator — Código com validação automática

```
                         ┌──────────────────┐
                         │  Desenvolvedor    │ ◄─── Confirma plano / resolve erros
                         └────────┬─────────┘
                                  │ cria Solution + projetos → trigger
                                  ▼
┌──────────────┐      ┌──────────────────────┐      ┌──────────────────┐
│ Azure DevOps │◄────▶│   X++ Agent          │◄────▶│   MS Learn MCP   │
│     MCP      │      │  (Copilot Agent Mode)│      │                  │
│              │      └──────────┬───────────┘      └──────────────────┘
│ • Download   │                 │                   • Consulta quando
│   Prompt Det.│                 │                     encontra erros
│ • Download   │                 ▼
│   TDD        │      ┌──────────────────────┐
└──────────────┘      │  Ninja MCP Server    │
                      │  (modo híbrido)      │
  ┌────────────────┐  │                      │
  │ 📜 Manifesto/  │  │ GERA:               │      ┌────────────────────┐
  │ Constituição   │  │ • create_d365fo_file │      │  📁 VS Solution    │
  │ (prefixo,      │  │ • modify_d365fo_file │      │  (.sln + .rnrproj) │
  │  naming,       │  │ • generate_code      │      │  criada pelo dev   │
  │  regras)       │  │ • generate_smart_*   │      │  antes do trigger  │
  └────────────────┘  │                      │      └────────────────────┘
                      │ VALIDA:              │
  ┌────────────────┐  │ • build_d365fo_proj  │      ┌────────────────────┐
  │ ⚙️  Config     │  │ • run_bp_check      │      │  📂 Model local    │
  │ Local (.md)    │  │ • get_error_help    │      │  (output dos       │
  │ • project path │  │ • trigger_db_sync   │      │   objetos gerados) │
  │ • model target │  └──────────────────────┘      └────────────────────┘
  └────────────────┘

  ┌────────────────────┐      ┌────────────────────┐
  │ 📘 X++ Playbook    │      │ 🔄 Lessons Learned │
  │ (GitHub repo)       │      │ Loop               │
  │ • Correções de     │      │ Erros resolvidos → │
  │   compilação       │      │ Dev confirma →     │
  │ • BP warning fixes │      │ Atualiza Playbook  │
  │ • Padrões de código│      │ via PR no GitHub   │
  └────────────────────┘      └────────────────────┘
```

**Geração hierárquica**: Labels → Enums → EDTs → Tables → Views → Queries → Data Entities → Classes → Forms → Reports → Menu Items → Menus → Security

**Build Validation Loop**: Compila → Erros? → Corrige auto → Não consegue? → Pergunta ao dev → BP Check → OK ✅ → Próximo grupo

---

<!-- SLIDE 8 — FASE 4: CODE REVIEW (conexões completas) -->

## Fase 4: Code Review Agent — Qualidade no PR

```
                         ┌──────────────────┐
                         │    Tech Lead      │ ◄─── Aprova PR + comenta
                         └────────┬─────────┘
                                  │ review final
                                  ▼
┌──────────────┐      ┌──────────────────────┐      ┌──────────────────┐
│  GitHub MCP  │◄────▶│  Code Review Agent   │◄────▶│   MS Learn MCP   │
│              │      │  (Copilot Agent Mode)│      │                  │
│ • Lê diff PR │      └──────────────────────┘      └──────────────────┘
│ • Posta      │                 │                   • Best practices
│   comentários│                 │                     oficiais
│ • Lê comen-  │                 ▼
│   tários TL  │      ┌────────────────────┐
└──────────────┘      │ 📜 Manifesto/      │
                      │ Constituição       │
                      │ (regras do projeto)│
                      └────────────────────┘

  ┌────────────────────┐      ┌────────────────────┐
  │ 📘 CR Playbook     │      │ 🔄 Lessons Learned │
  │ (GitHub repo)       │      │ Loop               │
  │ • Melhores práticas│      │ Comentários do TL →│
  │   arquiteturais    │      │ Agente sintetiza → │
  │ • Regras de        │      │ Gera PR Playbook → │
  │   segurança        │      │ TL aprova merge    │
  │ • Performance      │      └────────────────────┘
  │   patterns         │
  └────────────────────┘
```

**O agente valida**: naming, security, performance, patterns, extensibilidade
**O Tech Lead decide**: comentários do TL viram novas regras no Playbook

---

<!-- SLIDE 9 — ARQUITETURA RESUMIDA (30s) -->

## Arquitetura

```
                    ┌─────────────────┐
                    │    Manifesto    │  Regras do arquiteto
                    │  /Constituição  │  (prefixo, naming, nível detalhe)
                    └────────┬────────┘
                             │
┌──────────┐  ┌──────────┐  │  ┌──────────────┐  ┌──────────────┐
│ MS Learn │  │ DevOps   │  │  │  Ninja MCP   │  │  D365FO MCP  │
│   MCP    │  │   MCP    │  │  │   (híbrido)  │  │   (MS)       │
└────┬─────┘  └────┬─────┘  │  └──────┬───────┘  └──────┬───────┘
     │              │        │         │                  │
     └──────────────┴────────┴─────────┴──────────────────┘
                             │
              ┌──────────────┴──────────────┐
              │   GitHub Copilot Agent Mode  │
              │                              │
              │  Fase 1 → 2 → 3 → 4         │
              │  Human-in-the-loop           │
              │  Playbook / Lessons Learned  │
              └──────────────────────────────┘
```

**Ninja MCP Híbrido**: Azure DB (standard + ISV) + File Search Local (custom model)
Sincronia: build machine nightly + file search local cobre gap de 1 dia

---

<!-- SLIDE 10 — IMPACTO / CALL TO ACTION (30s) -->

## O que muda

| Antes | Depois |
|-------|--------|
| FDD depende 100% da experiência do consultor | Agente classifica Standard/Config/GAP com evidência do MS Learn |
| TDD sem visibilidade do código real | Agente consulta 584k+ objetos e cita evidência para cada decisão |
| Código X++ manual, propenso a erros | Geração hierárquica com compilação e BP check automáticos |
| Code review tardio e inconsistente | Agente revisa no PR com regras do Playbook |
| Conhecimento se perde entre projetos | Playbook evolui a cada revisão — agentes aprendem com o time |

---

## Status e Próximos Passos

| Fase | Status | Próxima ação |
|------|--------|-------------|
| 1 — FDD Generator | ✅ Funcionando | Introduzir mockups automáticos (D365FO MCP Server MS) |
| 2 — TDD Generator | 🔲 Próximo | Implementar agente + schema do Prompt Detalhado |
| 3 — X++ Generator | 🔲 Planejado | Depende da Fase 2 |
| 4 — Code Review | 🔲 Planejado | Pesquisar trigger via GitHub Action |

**Plataforma**: GitHub Copilot Agent Mode
**Approach**: Human-in-the-loop — o profissional decide, o agente executa

---

*Framework reutilizável para qualquer projeto D365 FnO (SCM / HR)*
