# THEMISIA – Auditoria Interna SOX · Petrobras
## Documentação Técnica Completa do Projeto

> *THEMISIA — nome inspirado em Themis, deusa grega da justiça e conformidade. Automação de produtividade para a Certificação SOX da Petrobras.*

> **Projeto:** THEMISIA – Automação da Auditoria Interna SOX
> **Versão:** 1.5
> **Responsável:** Auditoria Interna / CONF/CI
> **Publicado em:** https://suellen1986.github.io/themisia/
> **Data:** Junho de 2026

---

## Índice

1. [Visão Geral e Premissas Técnicas](#1-visão-geral-e-premissas-técnicas)
2. [Arquitetura Técnica](#2-arquitetura-técnica)
3. [Lógica extraída do Guia Prático SOX 2025](#3-lógica-extraída-do-guia-prático-sox-2025)
4. [Módulo 1 — Gerador de RPA + Folha de Teste](#4-módulo-1--gerador-de-rpa--folha-de-teste)
5. [Módulo 2 — Revisor AI SOX](#5-módulo-2--revisor-ai-sox)
6. [Módulo 3 — Painel de Avaliação de Metas](#6-módulo-3--painel-de-avaliação-de-metas)
7. [Mapeamento Completo do CPT](#7-mapeamento-completo-do-cpt)
8. [Avaliação do Desenho do Controle](#8-avaliação-do-desenho-do-controle)
9. [Assertivas SOX](#9-assertivas-sox)
10. [Atributos do Teste](#10-atributos-do-teste)
11. [Validação da População](#11-validação-da-população)
12. [Tabela de Amostragem SOX](#12-tabela-de-amostragem-sox)
13. [Tecnologias](#13-tecnologias)
14. [Como Usar](#14-como-usar)
15. [Arquitetura Backend para Petrobras (Roadmap)](#15-arquitetura-backend-para-petrobras-roadmap)
16. [Histórico de Versões](#16-histórico-de-versões)
17. [Glossário](#17-glossário)

---

## 1. Visão Geral e Premissas Técnicas

THEMISIA automatiza o ciclo completo de certificação SOX da Petrobras em três módulos integrados.

### ⚠️ Premissas fundamentais — resposta às perguntas frequentes

| Pergunta | Resposta |
|---------|---------|
| **O sistema usa LLM / IA generativa?** | **Não.** Nenhum campo é gerado por modelo de linguagem. Toda lógica é *rule-based*: regex, templates de texto e regras codificadas manualmente com base no Guia SOX Petrobras. O nome "Revisor AI SOX" é um *label* funcional, não técnico. |
| **Todos os campos são gerados automaticamente?** | Os campos estruturais (RPA, FT, atributos, validação de população) são gerados por extração textual das colunas do CPT. O auditor preenche o conteúdo substantivo (SIM/NÃO, procedimentos executados, evidências). |
| **Roda no backend / servidor?** | **Não. 100% client-side (frontend).** É um único arquivo HTML executado no navegador do usuário. Não há servidor, banco de dados, backend, API externa ou serviço de nuvem. A única requisição de rede é o download do `CPT.xlsx` do GitHub Pages no carregamento. |
| **Onde está a lista de colunas do CPT?** | Seção 7 deste documento — mapeamento completo de todas as 40 colunas utilizadas (das 118 existentes no CPT). |
| **Como o sistema "entendeu" o Guia SOX?** | As regras de auditoria foram **codificadas manualmente** pela equipe de auditoria no JavaScript do sistema, com base no Guia Prático SOX 2025 e no PCAOB AS 2301. Seção 3 documenta exatamente quais regras foram extraídas do Guia e como cada uma foi implementada. |

### Módulos

| Módulo | Nome | Status |
|--------|------|--------|
| 1 | **Gerador de RPA + Folha de Teste** | ✅ v1.5 |
| 2 | **Revisor AI SOX** | ✅ v1.5 |
| 3 | **Painel de Avaliação de Metas** | ✅ v1.5 |

---

## 2. Arquitetura Técnica

```
┌─────────────────────────────────────────────────────────────────┐
│                      THEMISIA  v1.5 — MVP                       │
│           https://suellen1986.github.io/themisia                │
│     Single HTML file · GitHub Pages · 100% Client-Side         │
│     Sem LLM · Sem Backend · Sem API · Sem Banco de Dados       │
└──────────────┬──────────────────┬────────────────┬─────────────┘
               │                  │                │
         MÓDULO 1           MÓDULO 2          MÓDULO 3
      Gerador RPA         Revisor AI         Painel Metas
      Folha Teste         Rule-based         localStorage
      CPT auto-load       DOM + PDF          CSV export
               │                  │                │
    ┌──────────┴──────┐  ┌────────┴──────┐  ┌─────┴────────┐
    │ SheetJS (emb.)  │  │ PDF.js (CDN)  │  │ localStorage │
    │ Fetch CPT.xlsx  │  │ Regex engine  │  │ var global   │
    └─────────────────┘  └───────────────┘  └──────────────┘
```

### Fluxo completo do ciclo

```
[GitHub Pages]         CPT.xlsx auto-carregado via fetch()
      ↓
[Usuário]              Seleciona controle → Gera RPA + Folha de Teste
      ↓
[Auditor]              Preenche Folha de Teste (SIM/NÃO, procedimentos, evidências)
      ↓
[Módulo 2]             Revisor analisa DOM da FT → Emite parecer rule-based
      ↓
[Coordenador]          Avalia auditor no Módulo 3 → Dados persistem em localStorage
      ↓
[Exportação]           PDF (impressão) / HTML / CSV
```

### Componentes do arquivo único (`index.html`)

| Componente | Tamanho aprox. | Função |
|-----------|--------------|--------|
| `<script>` #1 — SheetJS `xlsx.full.min.js` | ~640 KB | Leitura do CPT.xlsx |
| `<script>` #2 — App JS | ~90 KB | Lógica dos 3 módulos |
| CSS inline | ~8 KB | Identidade visual Petrobras |
| HTML estrutural | ~5 KB | Estrutura das abas e painéis |

### Por que JavaScript ES5?

O código usa ES5 puro (sem arrow functions, sem template literals, sem `const`/`let` fora de escopos seguros) para garantir compatibilidade máxima com os navegadores corporativos da Petrobras sem necessidade de transpilação ou build.

---

## 3. Lógica extraída do Guia Prático SOX 2025

Esta seção documenta **como cada regra do Guia Prático SOX 2025 foi traduzida em código** no THEMISIA. O Guia é o documento metodológico oficial da Petrobras (v09/2025), fundamentado no PCAOB AS 2301 e nas normas da SEC.

### 3.1 Processo de Certificação SOX — fluxo geral (Guia, pág. 5)

O Guia define o seguinte fluxo entre os atores:

```
Gestores → Autoavaliação (sign-off GRC-PC)
CONF/CI  → Submete controles, calcula materialidade, define escopo
AI       → Realiza testes de efetividade
ADC      → Realiza testes externos (KPMG)
```

**Como o THEMISIA implementa:** O Módulo 1 apoia a fase de **execução dos testes pela AI**. O Módulo 2 apoia a **revisão pelo coordenador**. O Módulo 3 apoia o **acompanhamento de metas da equipe**.

### 3.2 Arquivo CPT — fonte primária de dados (Guia, págs. 26–30)

O Guia define o CPT como:

> *"CPT (Control Performance Test) — relatório de controle do Process Control, disponível no SAP ERP (sistema IP6). Contém as informações do catálogo do GRC-PC exportado para planilha Excel, no formato de banco de dados."*

Regras do Guia implementadas no THEMISIA:

| Regra do Guia | Implementação no THEMISIA |
|--------------|--------------------------|
| Filtrar coluna "U" (Nível de Prova = 05) para escopo AI | `C.NIVEL_PROVA` (índice 26) — campo exibido na seção de Identificação do RPA |
| Verificar coluna BX (ano do controle) | `C.ANO` (índice 75) — validação de escopo |
| Usar colunas BJ (Situação Atual) e V (Sugestão de Abordagem) para criar procedimento de teste | `C.SITUACAO_ATUAL` (61) e `C.ABORDAGEM` (21) — base para `buildAtributos()` |
| Usar coluna BK (Desc. Extração) para validação da população | `C.DESC_EXTR` (62) — base para `buildValidacaoPopulacao()` |
| Usar coluna AF (Evidência Sugerida) para solicitar evidências | `C.EVIDENCIA_SUG` (31) — seção 5 da Folha de Teste |

### 3.3 Avaliação do Desenho do Controle (Guia, págs. 71–72)

O Guia define **8 itens de avaliação do desenho** (resposta: SIM / NÃO / N/A):

| # | Pergunta do Guia | Coluna CPT de referência |
|---|-----------------|------------------------|
| 1 | A situação atual atende à melhor prática, mitiga os riscos e é capaz de prevenir/detectar erros? | BJ (Situação Atual), V (Abordagem), AU (Risco) |
| 2 | A descrição do desenho reflete sua efetiva operação? | BJ (Situação Atual) |
| 3 | A evidência de operação anexada está de acordo com a situação atual? | AF (Evidência Sugerida), BJ |
| 4 | As pessoas que operam o controle possuem a devida competência? | BQ, BR, BS, BT (colunas de responsáveis) |
| 5 | O controle está ligado a uma assertiva? | Coluna AU (Risco/Assertiva) |
| 6 | As assertivas estão ligadas aos controles, contas e processos atualizados? | AU |
| 7 | A forma de obtenção e validação da população foi registrada? | BK (Desc. Extração) |
| 8 | A descrição da extração descreve adequadamente como a população pode ser obtida? | BK |

**Implementação no THEMISIA:** A seção "Análise do Desenho" do RPA (seção 5) exibe esses 8 itens gerados a partir das colunas `FINALIDADE` (36), `AUTOMACAO` (37), `STATUS_DESIGN` (56) e `CLASSIF_DESIGN` (60).

### 3.4 Validação da Totalidade da População (Guia, págs. 33–39)

O Guia define que a validação da população deve:

1. Verificar se a Situação Atual ou Descrição da Extração (coluna BK do CPT) apresenta como a população será extraída
2. Verificar quais empresas estão no escopo (`C.EMPRESA`, índice 106)
3. Verificar a data de início do controle (`C.INICIO_OP`, índice 67)
4. Evidenciar período e filtros aplicados
5. Para query BW: verificar se está na lista de Queries SOX homologadas
6. A validação deve responder: *"Essa forma de extração traz com integridade todo o universo da base que é parte do escopo?"*

**Implementação no THEMISIA:** A função `buildValidacaoPopulacao()` lê `C.DESC_EXTR` (índice 62) e detecta automaticamente o tipo de extração por regex, gerando instruções específicas para cada caso (Blackline, SAP, e-mail, BDOC, planilha). Ver seção 11.

### 3.5 Critério de Seleção da Amostra (Guia, págs. 41–54)

O Guia define dois critérios:

**a) Teste Populacional (abordagem preferencial):** Usar quando a população é de baixo volume, quando há ferramenta automatizada efetiva, ou quando amostras não seriam suficientes.

**b) Tabela Amostral SOX (probabilístico):** Seleção aleatória simples via ACL Analytics, com registro da semente (Seed). A quantidade de amostras depende da **categoria do risco** (A, B ou C) e da **frequência** do controle.

Categorias de risco (Guia, pág. 44):
- **Risco A:** Controles com Fraqueza Material, Deficiência Significativa, indicativo de fraude
- **Risco B:** Controles novos ou alterados, outras deficiências, ineficácia nos últimos 5 anos
- **Risco C:** Demais controles

Para controles **por evento** (sem frequência definida): anualizar a população via regra de três.

**Implementação no THEMISIA:** A seção 8 do RPA ("Procedimento de Teste") calcula automaticamente o tamanho amostral usando `C.FREQUENCIA` (índice 34) e `C.RISCO_CTRL` (índice 28), aplicando a tabela amostral codificada. Ver seção 12.

### 3.6 Procedimentos / Atributos do Teste (Guia, págs. 66–69)

O Guia define:

> *"No CPT, verificar a Sugestão de Abordagem do controle - Melhor Prática - (coluna V) e a Situação Atual (coluna BJ). No atributo, incluir a(s) alternativa(s) cadastrada(s) no procedimento em forma de pergunta. Cada atributo cadastrado será uma pergunta a ser respondida para cada item de amostra."*

O Guia ainda especifica que a aba "Detalhamento" deve conter:
- Avaliação do Desenho (Sim/Não/N/A)
- Análise de Riscos (alinhamento Situação Atual × Objetivo × Riscos)
- Introdução (empresas, assertivas)
- **População e Amostra** (como foi validada a extração, como foi selecionada a amostra, caminho das evidências)
- Evidências Solicitadas (coluna AF do CPT)
- Análise das Evidências (verificações por amostra × atributo)
- Conclusão (eficaz ou ineficaz, com exceções em Nota)

**Implementação no THEMISIA:** Cada atributo na Folha de Teste exibe: pergunta → SIM/NÃO → "Como verificar" → campo de procedimentos executados → campo de caminho dos papéis de suporte. Ver seção 10.

### 3.7 Check List de Revisão (Guia, pág. 108)

O Guia define 6 blocos de verificação para o revisor:

| # | Bloco | O que verificar |
|---|-------|----------------|
| 1 | Validação da População | Frequência, Situação Atual/Desc. Extração no CPT, empresas, filtros, período, data início |
| 2 | Critério de Amostragem | Tabela amostral, anualização (se evento), seleção aleatória ACL, seed salva |
| 3 | Procedimentos de Testes | Ações executadas, arquivos utilizados, arquivos por atributo, explicações de exceções sem desvio |
| 4 | Conclusão | Resultado claro (eficaz/ineficaz), exceções destacadas em Nota |
| 5 | Ponto Crítico | Ponto criado no RM SOX e no GRC-PC quando há desvio |
| 6 | Oport. de Melhoria | Oportunidades identificadas listadas na aba correspondente |

**Implementação no THEMISIA:** Os 11 critérios do Módulo 2 (Revisor AI SOX) foram derivados diretamente desses 6 blocos, com decomposição em critérios mensuráveis. Ver seção 5.

### 3.8 Fluxo de Finalização do Teste (Guia, págs. 114–128)

O Guia define as etapas de finalização:

1. Auditor preenche: Data fim, Conclusão, Resultado (Eficaz / Ineficaz / Sem Amostra Mínima / NPT)
2. Auditor muda status no RM para "Aguardando Revisão"
3. Revisor assume o teste, preenche "Revisor" e muda para "Em Revisão"
4. Se precisar de ajuste: "Aguardando Providências" → auditor corrige → revisor encerra
5. Após revisão: auditor salva abas (Dados do Controle + Dados do Teste) como PDF e arquiva no BDOC
6. Auditor cadastra o resultado no GRC-PC (data, resultado, PDF em anexo)
7. Auditor envia e-mail ao gestor + CONF/CI com o resultado (com cópia ao coordenador e líder SOX)
8. Se ineficaz: alinha com revisor, coordenador e gerente; cria Ponto Crítico no GRC-PC; avalia possibilidade de Reteste

**Implementação no THEMISIA:** O Módulo 1 gera o RPA e a FT que suportam as etapas 1–2. O Módulo 2 apoia a etapa 3–4 (revisão). O Módulo 3 apoia o acompanhamento pós-ciclo.

---

## 4. Módulo 1 — Gerador de RPA + Folha de Teste

### RPA — 10 Seções

| # | Seção | Colunas CPT usadas | Função no código |
|---|-------|-------------------|-----------------|
| 1 | Identificação do Controle | ID_CTRL, CTRL, FREQUENCIA, NIVEL_PROVA, CAT_CTRL, RISCO_CTRL, EMPRESA | `buildRPA()` |
| 2 | Objetivo do Controle | OBJETIVO, DESC_OBJ, CONTAS, RISCO, DESC_RISCO | `buildRPA()` |
| 3 | Descrição e Entendimento | SITUACAO_ATUAL, ABORDAGEM, NATUREZA | `buildRPA()` |
| 4 | Riscos Associados | RISCO, DESC_RISCO, CAUSA, CAT_RISCO | `buildRPA()` |
| 5 | Análise do Desenho | FINALIDADE, AUTOMACAO, STATUS_DESIGN, CLASSIF_DESIGN | `buildRPA()` |
| 6 | Evidências Necessárias | EVIDENCIA_SUG | `buildRPA()` |
| 7 | Questionamentos ao Responsável | SITUACAO_ATUAL, ABORDAGEM (extração de ações) | `buildRPA()` |
| 8 | Procedimento de Teste | FREQUENCIA, RISCO_CTRL → tabela amostral | `buildRPA()` |
| 🔍 | **Validação da População** | DESC_EXTR (col. BK) | `buildValidacaoPopulacao()` |
| 9 | Constatações Críticas / Plano de Ação | PONTO_CRIT, DESC_PC, RECOMEND | `buildRPA()` |

### Folha de Teste — 10 Seções

| # | Seção | Detalhe | Editável pelo auditor? |
|---|-------|---------|----------------------|
| — | Cabeçalho | ID, controle, frequência, data, empresa, diretoria | Não |
| 1 | Objetivo do Teste | CPT — OBJETIVO | Não |
| 2 | Descrição do Controle | CPT — SITUACAO_ATUAL | Não |
| 3 | Riscos Mitigados | CPT — RISCO + ASSERTIVAS | Não |
| 4 | Critério de Aprovação | EFICAZ / INEFICAZ (definição) | Não |
| 5 | Evidências a Solicitar | CPT — EVIDENCIA_SUG | Não |
| 6 | **Atributos do Teste** | SIM/NÃO + Como verificar + Procedimentos + Caminho papéis | **Sim** |
| 7 | **Resultado por Amostra** | Tabela editável: ID amostra × atributo × SIM/NÃO/NA + nota | **Sim** |
| 8 | Conclusão | Checkbox EFICAZ/INEFICAZ + justificativa | **Sim** |
| 9 | Constatações Críticas Identificadas | Texto livre (ex-"Pontos Críticos") | **Sim** |
| 10 | Sugestões de Plano de Ação | Texto livre | **Sim** |

---

## 5. Módulo 2 — Revisor AI SOX

Analisa a Folha de Teste preenchida e emite parecer estruturado **sem necessidade de API ou LLM**. Toda a análise é **rule-based** — as regras foram derivadas do Check List de Revisão do Guia Prático SOX 2025 (págs. 108–112) e da metodologia interna de auditoria.

### Duas formas de entrada

**Opção A (recomendada):** Revisar a Folha de Teste já aberta no THEMISIA — lê diretamente os radio buttons, textareas e checkboxes do DOM via `buildReviewFromDOM()`.

**Opção B:** Upload de PDF — extrai texto via PDF.js 3.4.120. Se o PDF não preservar camada de texto (ex: impresso pelo browser), permite colar manualmente.

### Modos de Revisão

| Modo | Threshold aprovação | Postura | Equivalência |
|------|-------------------|---------|-------------|
| 🔵 Padrão | Score ≥ 65% | Revisor interno equilibrado | Revisão interna CONF/CI |
| 🔴 Crítico (ADC) | Score ≥ 80% | Simula auditoria externa | KPMG / Big 4 |

### 11 Critérios Analisados — origem no Guia SOX

| # | Critério | Bloco Guia | Peso |
|---|---------|-----------|-----|
| 1 | Conclusão declarada (EFICAZ/INEFICAZ) | Check List bloco 4 | Alto |
| 2 | Coerência entre atributos e conclusão | Check List bloco 4 | Alto |
| 3 | Quantidade amostral suficiente (tabela SOX) | Check List bloco 2 | Alto |
| 4 | Procedimentos executados descritos | Check List bloco 3 | Alto |
| 5 | Nível de detalhe dos procedimentos *(modo crítico)* | Guia pág. 69 | Médio |
| 6 | Caminhos dos papéis de suporte informados | Guia pág. 69 | Médio |
| 7 | Período das amostras identificado | Check List bloco 1 | Médio |
| 8 | Validação de integridade da população | Check List bloco 1 | Médio |
| 9 | Constatações críticas documentadas *(se há NÃO)* | Check List bloco 5 | Alto |
| 10 | Sugestões de Plano de Ação *(se há NÃO)* | Check List bloco 6 | Médio |
| 11 | Campos críticos CPT preenchidos | Guia pág. 26 | Baixo |

### Output do Parecer

- **Status:** 🟢 APROVADO / 🟡 APROVADO COM RESSALVAS / 🟠 DEVOLVIDO / 🔴 INEFICAZ
- **Score de qualidade** (0–100%)
- **Checklist** ✅/❌ por critério
- **Pontos para complementação** numerados
- **Perguntas ao auditor** geradas automaticamente
- **Contagem SIM/NÃO** vs. esperado
- **Conclusão fundamentada** do revisor

---

## 6. Módulo 3 — Painel de Avaliação de Metas

Coordenador/revisor avalia o trabalho do auditor após a revisão. Dados persistem em `localStorage` do navegador + variável global `m3Data[]`.

### Dimensões Avaliadas (1–5 estrelas)

| Dimensão | Descrição |
|----------|-----------|
| Qualidade da Documentação | Clareza, organização e completude dos papéis |
| Suficiência das Evidências | As evidências suportam a conclusão |
| Clareza da Conclusão | Objetiva, fundamentada e consistente |
| Tempestividade | Entregue dentro do prazo do ciclo |
| Identificação de Riscos/Exceções | Desvios e pontos críticos identificados |

Nota abaixo de 3 exige justificativa obrigatória no campo Observações.

### Armazenamento de Dados

```javascript
// Estrutura de cada avaliação em m3Data[]
{
  auditor: "Nome do Auditor",
  ciclo: "2026-T2",
  controle: "CTR01 Revisar variações DCC",
  revisor: "Nome do Revisor",
  obs: "Observações",
  notas: [4, 3, 5, 4, 3],  // 5 dimensões
  media: 3.8,
  ts: 1718000000000         // timestamp
}
```

Dados persistem em `localStorage` com chave `m3data_v1` e sobrevivem ao fechamento do navegador (por computador/perfil de usuário). Para compartilhamento entre usuários, exportar via CSV.

### Vistas do Painel

**Visão "Todos os auditores"** (default ao entrar na aba):
- KPIs: Média geral, Nº auditores, Nº avaliações, Melhor dimensão, Dimensão com atenção
- Cards de média por dimensão com barra de progresso
- Ranking dos auditores clicável → abre visão individual

**Visão individual** (ao selecionar auditor no ranking):
- Cards de resumo por dimensão
- Tabela completa de controles avaliados
- Botão "← Todos os auditores"

**Painel do Ciclo:**
- Filtro por ciclo (ex: 2026-T2)
- Ranking completo: todos os auditores × médias por dimensão
- Badges coloridos: 🟢 ≥4 · 🔵 ≥3 · 🟠 ≥2 · 🔴 <2
- Exportação CSV

---

## 7. Mapeamento Completo do CPT

O CPT possui **118 colunas** (índice base 0). O THEMISIA utiliza **40 colunas**. As demais são ignoradas.

| Constante no código | Índice | Coluna Excel (aprox.) | Nome no CPT | Uso no THEMISIA |
|--------------------|---------|-----------------------|------------|----------------|
| `C.ORG` | 1 | B | Organização | RPA seção 1 |
| `C.MACROPROCESSO` | 6 | G | Macroprocesso | RPA seção 1 |
| `C.PROCESSO` | 8 | I | Processo | RPA seção 1 |
| `C.SUBPROCESSO` | 11 | L | Subprocesso | RPA seção 1 |
| `C.RISCO_SUBPROC` | 13 | N | Risco do Subprocesso | Contexto RPA |
| `C.ID_CTRL` | 19 | T | **ID do Controle** | ⭐ Busca / identificação |
| `C.CTRL` | 20 | U | **Nome do Controle** | ⭐ Busca / cabeçalhos |
| `C.ABORDAGEM` | 21 | V | **Sugestão de Abordagem (Melhor Prática)** | ⭐ Atributos + avaliação desenho |
| `C.CAT_CTRL` | 25 | Z | Categoria do Controle | RPA seção 1 |
| `C.NIVEL_PROVA` | 26 | AA | **Nível de Prova** | RPA seção 1 (escopo AI = 05) |
| `C.IMPORTANCIA` | 27 | AB | Importância | Análise |
| `C.RISCO_CTRL` | 28 | AC | **Risco do Controle (A/B/C)** | ⭐ Tabela amostral |
| `C.RELEVANCIA` | 29 | AD | Relevância | Análise |
| `C.CTRL_ASSOC` | 30 | AE | Controles Associados | RPA |
| `C.EVIDENCIA_SUG` | 31 | AF | **Evidência Sugerida** | ⭐ FT seção 5 + avaliação desenho |
| `C.NATUREZA` | 32 | AG | Natureza (Preventivo/Detectivo) | RPA seção 5 |
| `C.DATA_EVENTO` | 33 | AH | Data do Evento | RPA |
| `C.FREQUENCIA` | 34 | AI | **Frequência** | ⭐ Tabela amostral + FT cabeçalho |
| `C.FINALIDADE` | 36 | AK | Finalidade | RPA seção 5 |
| `C.AUTOMACAO` | 37 | AL | Automação | RPA seção 5 |
| `C.A_TESTAR` | 38 | AM | A Testar | RPA |
| `C.PROC_TESTE` | 39 | AN | Procedimento de Teste (padrão) | RPA seção 8 |
| `C.GRUPO` | 40 | AO | Grupo | RPA seção 1 |
| `C.SUBGRUPO` | 41 | AP | Subgrupo | RPA seção 1 |
| `C.OBJETIVO` | 42 | AQ | **Objetivo do Controle** | ⭐ FT seção 1 |
| `C.CTG_OBJ` | 43 | AR | Categoria do Objetivo | RPA |
| `C.DESC_OBJ` | 44 | AS | **Descrição do Objetivo** | ⭐ RPA seção 2 |
| `C.CAT_RISCO` | 45 | AT | Categoria do Risco | RPA seção 4 |
| `C.RISCO` | 46 | AU | **Risco(s) — Assertivas** | ⭐ RPA seção 4 + avaliação desenho |
| `C.DESC_RISCO` | 47 | AV | **Descrição do Risco** | ⭐ RPA seção 4 |
| `C.CAUSA` | 48 | AW | Causa do Risco | RPA seção 4 |
| `C.STATUS_DESIGN` | 56 | BE | Status do Desenho | RPA seção 5 |
| `C.CLASSIF_DESIGN` | 60 | BI | Classificação do Desenho | RPA seção 5 |
| `C.SITUACAO_ATUAL` | 61 | BJ | **Situação Atual do Controle** | ⭐⭐ Atributos + FT seção 2 |
| `C.DESC_EXTR` | 62 | BK | **Descrição da Extração de Dados** | ⭐⭐ Validação da população |
| `C.FINALIDADE2` | 63 | BL | Finalidade (versão atual) | RPA |
| `C.AUTOMACAO2` | 64 | BM | Automação (versão atual) | RPA |
| `C.FREQUENCIA2` | 65 | BN | Frequência (versão atual) | RPA |
| `C.INICIO_OP` | 67 | BP | **Início de Operação** | Validação população |
| `C.RESP_NOME` | 69 | BR | Nome do Responsável | RPA cabeçalho |
| `C.PERIODO` | 74 | BW | Período | RPA |
| `C.ANO` | 75 | BX | **Ano** | Validação escopo |
| `C.PONTO_CRIT` | 82 | CF | **Ponto Crítico** | ⭐ Alertas + FT seção 9 |
| `C.DESC_PC` | 89 | CM | Descrição do Ponto Crítico | RPA |
| `C.RECOMEND` | 90 | CN | Recomendação | RPA seção 10 |
| `C.EMPRESA` | 106 | DD | Empresa | RPA + FT cabeçalho |
| `C.DIRETORIA` | 107 | DE | Diretoria / Gerência Executiva | RPA |
| `C.GER_EXEC` | 108 | DF | Gerência Executiva | RPA |
| `C.CONTAS` | 117 | DN | **Contas / Assertivas** | ⭐ RPA seção 2 |

> **Nota sobre referência de colunas Excel:** O CPT usa índice base 0 no JavaScript. A coluna de índice 21 corresponde à coluna V do Excel (A=0, B=1, ... V=21). As letras Excel são aproximadas — o mapeamento exato pode variar por versão do CPT exportado do GRC.

---

## 8. Avaliação do Desenho do Controle

Conforme Guia SOX 2025 (págs. 71–72), o THEMISIA gera os 8 itens de avaliação na seção 5 do RPA:

| # | Pergunta | Fonte CPT | Resposta possível |
|---|---------|----------|------------------|
| 1 | A situação atual atende à melhor prática e mitiga os riscos associados? | SITUACAO_ATUAL, ABORDAGEM, RISCO | SIM / NÃO / N/A |
| 2 | O desenho do controle reflete sua efetiva operação? | SITUACAO_ATUAL | SIM / NÃO / N/A |
| 3 | A evidência de operação está de acordo com a situação atual? | EVIDENCIA_SUG, SITUACAO_ATUAL | SIM / NÃO / N/A |
| 4 | As pessoas que operam o controle têm a devida competência? | RESP_NOME | SIM / NÃO / N/A |
| 5 | O controle está ligado a uma assertiva? | RISCO (col. AU) | SIM / NÃO / N/A |
| 6 | As assertivas estão ligadas aos controles e contas atualizados? | CONTAS | SIM / NÃO / N/A |
| 7 | A forma de obtenção da população foi registrada na extração? | DESC_EXTR | SIM / NÃO / N/A |
| 8 | A descrição da extração descreve adequadamente como a população pode ser obtida? | DESC_EXTR | SIM / NÃO / N/A |

**Premissa do Guia (pág. 71):** *"Caso o controle não apresente o item 1 como descrito, ele possui uma deficiência de desenho. Nesse caso, sua operação não pode ser sequer verificada, e ele deve ser devolvido para a CONF/CI."*

---

## 9. Assertivas SOX

Conforme PCAOB e Guia SOX 2025 (págs. 74–77), existem 6 assertivas para controles SOX (lista exaustiva):

| Sigla | Assertiva | Descrição |
|-------|-----------|-----------|
| **T** | Totalidade | Todas as transações/ativos/passivos foram registrados |
| **VD** | Validade | Transações/eventos registrados são válidos e autorizados |
| **R** | Registro | Transações foram registradas corretamente nas contas corretas |
| **C** | Cut-off | Transações foram registradas no período contábil correto |
| **VZ** | Valorização | Ativos/passivos foram corretamente valorados |
| **A** | Apresentação | Informações foram adequadamente classificadas e divulgadas nas DFs |

O THEMISIA extrai as assertivas da coluna `C.RISCO` (índice 46) e as exibe na seção 3 da Folha de Teste (Riscos Mitigados).

---

## 10. Atributos do Teste

A função `buildAtributos()` implementa a lógica do Guia SOX (pág. 67): *"verificar a Sugestão de Abordagem (coluna V) e a Situação Atual (coluna BJ) e incluir as alternativas em forma de pergunta."*

### Extração de termos literais

O algoritmo analisa `C.SITUACAO_ATUAL` (índice 61) e extrai:

**Sistemas detectados:**
- Blackline (com módulo específico: Variância, Reconciliação, etc.)
- SAP (com transações: VFX3, YSRELIMOB, KS13, etc.)
- Oracle, SICONF, BDOC, PW SATI, SIOF, ATICA

**Objetos contábeis detectados:**
- BP (Balanço Patrimonial), DRE (Demonstração de Resultado)
- Ativo Imobilizado, Intangível, Notas Explicativas
- Faturamento, GEE (Gases de Efeito Estufa), Contencioso

**Ações detectadas:**
- variação, movimentação, conciliação, carregamento, pendentes, estorno, aprovação

**Comparativos detectados:**
- período anterior, mês subsequente, fechamento

### 5 Atributos gerados

| # | Tema | Pergunta gerada | Critério |
|---|------|----------------|---------|
| 1 | Prazo | "A execução foi realizada dentro do prazo previsto?" | Detecta padrões de tempo na Situação Atual |
| 2 | Escopo | "Todos os [objetos detectados] foram analisados?" | Usa objetos contábeis reais extraídos |
| 3 | Sistema | "A consulta/extração foi realizada via [sistema/transação] conforme procedimento?" | Usa sistema + transação literais |
| 4 | Evidência | "A execução está registrada/evidenciada em [sistema] com status concluído/aprovado?" | Usa sistema de registro detectado |
| 5 | Exceções | "Os itens pendentes/divergentes foram devidamente avaliados e justificados?" | Detecta menção a pendentes/desvios |

Cada atributo exibe campos para o auditor preencher:
- Resposta: **SIM / NÃO / N/A**
- "Como verificar" (gerado automaticamente)
- **Procedimentos executados** (texto livre — obrigatório para aprovação no Módulo 2)
- **Caminho dos papéis de suporte** (texto livre — ex: `\\servidor\pasta\controle\`)

---

## 11. Validação da População

A função `buildValidacaoPopulacao()` lê `C.DESC_EXTR` (índice 62 — coluna BK do CPT) e gera instruções específicas conforme o tipo de extração detectado.

Conforme Guia SOX 2025 (págs. 34–39), a validação deve confirmar: *"Essa forma de extração traz com integridade todo o universo da base que é parte do escopo?"*

| Sistema detectado em DESC_EXTR | Instrução gerada pelo THEMISIA |
|-------------------------------|-------------------------------|
| `Blackline – Módulo [X] ([Entidade Y])` | "Acesse o Blackline, navegue ao Módulo [X] e filtre pela entidade [Y]. Exporte o relatório do período [início-fim] e compare com a população apresentada pelo auditado." |
| `TRANSAÇÃO [XXXXX]` (regex SAP) | "Execute a transação [XXXXX] no SAP com os mesmos parâmetros e filtros utilizados pelo auditado. Compare o resultado linha a linha." |
| `por e-mail` | "Solicite os e-mails originais ao auditado. Verifique remetente, destinatário e data de envio. Compare o conteúdo com o relatado." |
| Indicadores físicos (câmbio, produção, contencioso) | "Obtenha os dados de fonte independente (ANP / Banco Central / boletins oficiais) e compare com os valores da população do auditado." |
| `BDOC` | "Baixe o arquivo-base da grade de controles no BDOC e compare com o arquivo de upload da extração do auditado." |
| `planilha` / `Excel` | "Solicite a planilha original ao auditado e verifique a rastreabilidade dos dados (fórmulas, fontes, sem alterações manuais)." |
| Texto genérico | "Identifique a fonte primária dos dados descritos na Situação Atual e realize extração independente para comparação." |

**Regra do Guia (pág. 34):** *"Diferença não justificada entre a população do auditado e a extração independente = evidência de ineficácia do controle."*

---

## 12. Tabela de Amostragem SOX

Conforme PCAOB AS 2301 e Guia SOX Petrobras 2025 (págs. 44–48). Usada na seção 8 do RPA e no critério 3 do Módulo 2.

### Tabela Amostral — Risco A

| Frequência | Itens a testar |
|------------|---------------|
| Múltiplas vezes ao dia | 60 |
| Diário | 45 |
| Semanal | 15 |
| Quinzenal | 5 |
| Mensal | 3 |
| Trimestral | 2 |
| Semestral | 2 |
| Anual | 2 |

### Tabela Amostral — Risco B

| Frequência | Itens a testar |
|------------|---------------|
| Múltiplas vezes ao dia | 45 |
| Diário | 25 |
| Semanal | 10 |
| Quinzenal | 4 |
| Mensal | 3 |
| Trimestral | 2 |
| Semestral | 2 |
| Anual | 1 |

### Tabela Amostral — Risco C

| Frequência | Itens a testar |
|------------|---------------|
| Múltiplas vezes ao dia | 25 |
| Diário | 15 |
| Semanal | 5 |
| Quinzenal | 3 |
| Mensal | 2 |
| Trimestral | 2 |
| Semestral | 1 |
| Anual | 1 |

**Controles por evento (sem frequência definida):** anualizar via regra de três: `ocorrências_observadas ÷ meses_observados × 12`, depois consultar a tabela pela frequência equivalente mais próxima.

---

## 13. Tecnologias

| Tecnologia | Versão | Finalidade | Modo |
|-----------|--------|-----------|------|
| HTML5 + CSS3 | — | Interface — identidade visual Petrobras | Inline |
| JavaScript ES5 | — | Lógica dos 3 módulos (sem framework, sem build) | Inline |
| SheetJS (xlsx.js) | 0.18.5 | Leitura do CPT.xlsx | **Embutido** no HTML (~640KB) |
| PDF.js | 3.4.120 | Extração de texto de PDFs (Módulo 2 Opção B) | CDN (opcional) |
| GitHub Pages | — | Hospedagem e entrega do CPT.xlsx | Externo |
| Fetch API | Nativa | Auto-carregamento do CPT.xlsx | Nativa do browser |
| localStorage | Nativa | Persistência das avaliações do Módulo 3 | Nativa do browser |

**Nenhuma dependência de servidor, banco de dados, LLM ou API externa é obrigatória para operação do MVP.**

---

## 14. Como Usar

### Módulo 1 — Gerar RPA + Folha de Teste

1. Acesse `https://suellen1986.github.io/themisia/`
2. CPT carrega automaticamente (`✅ X controles carregados`) via fetch do GitHub
3. Digite o ID ou nome do controle no campo de busca → selecione
4. Clique **⚡ Gerar RPA + Folha de Teste**
5. Navegue pelas abas: **📄 RPA** · **📋 Folha de Teste** · **⚠️ Oportunidades**
6. Na Folha de Teste: preencha SIM/NÃO por atributo, procedimentos executados, caminhos das evidências
7. Na tabela "Resultado por Amostra": preencha por ID de amostra
8. **🖨️ Imprimir** (salvar como PDF para arquivar no BDOC) ou **💾 Exportar HTML**

### Módulo 2 — Revisar Folha de Teste

1. Com o controle selecionado e a FT preenchida no Módulo 1, role até **🔍 Revisor AI SOX**
2. **Opção A (recomendada):** clique **⚡ Revisar Folha de Teste Atual** — lê direto do DOM
3. **Opção B:** carregue o PDF salvo → se sem texto, cole manualmente → **⚡ Revisar pelo Texto**
4. Escolha o modo: 🔵 Padrão (revisão interna) ou 🔴 Crítico ADC (simulação KPMG)
5. O parecer aparece imediatamente com status, score, checklist, pontos e perguntas ao auditor

### Módulo 3 — Avaliar Auditores

1. Role até **🏆 Painel de Avaliação de Metas**
2. **Lançar Avaliação:** preencha auditor, ciclo, controle, revisor → clique nas estrelas (1–5) → **💾 Salvar**
3. **Painel por Auditor:** veja KPIs gerais ("Todos") ou clique no ranking para detalhe individual
4. **Painel do Ciclo:** selecione o ciclo → ranking completo → **📥 Exportar CSV**

---

## 15. Arquitetura Backend para Petrobras (Roadmap)

O MVP atual é 100% client-side e não requer TI para operar. Para integração com os sistemas corporativos da Petrobras, propõem-se três caminhos evolutivos com complexidade crescente.

### Contexto — sistemas corporativos Petrobras envolvidos

| Sistema | Função no processo SOX |
|---------|----------------------|
| **SAP GRC-PC** (IP6) | Catálogo de controles, cadastro de testes, Ponto Crítico |
| **RM SOX** (`rmsox.petrobras.com.br`) | Folha de Teste (FT), fluxo de revisão, workflow auditor-revisor |
| **BDOC** | Arquivo de evidências, população, amostras, Queries SOX |
| **ACL Analytics** | Seleção amostral aleatória (gerador de seed) |
| **Microsoft Teams** | Comunicação, acompanhamento, controle do ciclo |
| **GRC-PC (portal web)** | Cadastro final do resultado após revisão |
| **Power BI** | Dashboards de gestão e resultados do ciclo |
| **M365 / SharePoint** | Repositório de documentos, pastas de controle |

### Opção 1 — Quick Win: Power Platform (M365 — sem infraestrutura nova)

**Prazo estimado:** 2–4 meses | **Custo adicional:** $0 (já licenciado no M365)

```
┌──────────────┐    Power Automate     ┌──────────────────┐
│  THEMISIA    │ ──── webhook ────────► │  SharePoint List  │
│  (HTML)      │                        │  (m3Data remoto)  │
└──────────────┘                        └──────────────────┘
                                                │
                                        Power BI Dashboard
                                        (gestores / líderes)
```

**O que muda:**
- Dados do Módulo 3 (avaliações) salvos em SharePoint List via Power Automate — compartilhados entre todos os coordenadores
- Dashboard Power BI conectado ao SharePoint — visão consolidada por ciclo/diretoria
- Teams Bot (Power Virtual Agents) para notificações de prazo e resultado de testes
- CPT carregado de SharePoint em vez do GitHub — atualização automática diária

**O que permanece igual:** O HTML do THEMISIA não muda. Adiciona-se apenas uma chamada `fetch()` para o endpoint do Power Automate.

### Opção 2 — Enterprise: API Gateway + Microsserviços (recomendada para escala)

**Prazo estimado:** 6–12 meses | **Custo:** Infraestrutura + horas de desenvolvimento

```
┌──────────────┐   REST API   ┌──────────────────────────────────────┐
│  THEMISIA    │ ────────────► │         API Gateway (Kong/APIGEE)    │
│  (frontend)  │              └──────────┬───────────────┬────────────┘
└──────────────┘                         │               │
                                         ▼               ▼
                              ┌──────────────┐  ┌──────────────────┐
                              │ Serviço CPT  │  │ Serviço Avaliação│
                              │ (SAP GRC-PC) │  │ (PostgreSQL/RDS) │
                              └──────────────┘  └──────────────────┘
                                         │               │
                                         ▼               ▼
                              ┌──────────────┐  ┌──────────────────┐
                              │ SAP RFC/BAPI │  │ Power BI Premium │
                              │ (leitura CPT │  │ (relatórios)     │
                              │  em tempo   │  └──────────────────┘
                              │  real)      │
                              └─────────────┘
```

**Microsserviços sugeridos:**

| Serviço | Função | Tecnologia sugerida |
|---------|--------|-------------------|
| `cpt-service` | Leitura do GRC-PC via RFC/OData | Node.js + SAP Cloud SDK |
| `avaliacao-service` | CRUD de avaliações do Módulo 3 | Python FastAPI + PostgreSQL |
| `review-service` | Engine de revisão (regras codificadas + futuro LLM) | Python + Claude API (opcional) |
| `notificacao-service` | Emails e Teams ao finalizar teste | Microsoft Graph API |
| `auth-service` | SSO via AD Petrobras | SAML 2.0 / OAuth 2.0 |

**Banco de dados sugerido:**
```sql
-- Tabela principal de avaliações
CREATE TABLE avaliacoes_sox (
  id UUID PRIMARY KEY,
  auditor VARCHAR(100),
  ciclo VARCHAR(20),
  id_controle VARCHAR(50),
  revisor VARCHAR(100),
  nota_doc INTEGER CHECK (nota_doc BETWEEN 1 AND 5),
  nota_evid INTEGER CHECK (nota_evid BETWEEN 1 AND 5),
  nota_concl INTEGER CHECK (nota_concl BETWEEN 1 AND 5),
  nota_temp INTEGER CHECK (nota_temp BETWEEN 1 AND 5),
  nota_risc INTEGER CHECK (nota_risc BETWEEN 1 AND 5),
  media DECIMAL(3,2),
  obs TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  created_by VARCHAR(100)
);
```

### Opção 3 — Full Agentic: n8n + Claude API (visão de longo prazo)

**Prazo estimado:** 12–18 meses | **Modelo:** Human-in-the-loop com agentes especializados

```
[RAI/GRC-PC] ──webhook──► [Agente 1: Contextualizador]
                                │  (n8n + SAP API)
                                │  Gera RPA + Atributos (JSON)
                                ▼
                    [Revisão Humana — Auditor executa teste]
                                │
                                ▼
                    [Agente 2: Revisor AI]
                                │  (n8n + Claude API claude-opus-4-6)
                                │  Análise semântica da FT
                                │  Verdict: APROVADO/DEVOLVIDO/INEFICAZ
                                ▼
                    [Revisão Humana — Coordenador aprova/devolve]  ← OBRIGATÓRIA
                                │
                                ▼
                    [Agente 3: Avaliador]
                                │  (n8n + Power BI API)
                                │  Registra notas, atualiza painel
                                ▼
                    [GRC-PC atualizado] + [Teams notificado]
```

**Premissa inegociável:** Revisão humana obrigatória em **dois pontos** do fluxo — antes do Agente 2 (auditor executa) e antes do Agente 3 (coordenador aprova). O fluxo agentic não substitui o julgamento profissional do auditor.

**Integração com Claude API:**
```javascript
// Exemplo de chamada ao Agente 2 (review-service)
const response = await anthropic.messages.create({
  model: "claude-opus-4-6",
  max_tokens: 2048,
  system: `Você é um auditor SOX especialista da Petrobras.
           Analise a Folha de Teste e emita parecer estruturado.
           Regras: PCAOB AS 2301, Guia Prático SOX Petrobras 2025.`,
  messages: [{
    role: "user",
    content: `CPT do Controle: ${cpt_json}\n\nFolha de Teste Preenchida:\n${ft_text}`
  }]
});
```

### Comparativo das opções

| Critério | Opção 1 (Power Platform) | Opção 2 (API Gateway) | Opção 3 (n8n + LLM) |
|---------|------------------------|----------------------|---------------------|
| Tempo para produção | 2–4 meses | 6–12 meses | 12–18 meses |
| Custo adicional | Baixo (já licenciado) | Médio (infra + dev) | Alto (dev + API LLM) |
| Integração SAP | Limitada (export manual) | Nativa (RFC/OData) | Nativa (via n8n) |
| Qualidade da revisão | Rule-based (atual) | Rule-based (atual) | Semântica (Claude API) |
| Aprovação TI necessária | Mínima | Sim | Sim + Segurança |
| Risco de adoção | Baixo | Médio | Alto |
| **Recomendação** | ✅ Iniciar agora | 📋 Planejar para 2027 | 🔭 Visão 2028+ |

### Requisitos de segurança para qualquer backend

- **Autenticação:** SSO via AD Petrobras (SAML 2.0 ou OAuth 2.0) — nenhum usuário/senha local
- **Dados sensíveis:** Nenhum dado do CPT trafega para fora da rede Petrobras sem criptografia TLS 1.3
- **LLM (se adotado):** Garantir que dados de controles SOX não sejam usados para treinamento do modelo (Data Processing Agreement com Anthropic / Azure OpenAI)
- **Auditoria de acesso:** Log de todas as ações (quem gerou, quem revisou, quando) com retenção mínima de 7 anos (SOX requirement)
- **CPT:** Acesso ao endpoint de CPT restrito a VPN corporativa

---

## 16. Histórico de Versões

| Versão | Data | Alterações |
|--------|------|-----------|
| 1.0 | Jun/2025 | Módulo 1 inicial — RPA + Folha de Teste + Oportunidades de Melhoria |
| 1.1 | Jun/2026 | Atributos derivados da Situação Atual; CPT auto-carregado; rebatizado THEMISIA |
| 1.2 | Jun/2026 | Validação da População; FT editável; Constatações Críticas; Sugestões de Plano de Ação |
| 1.3 | Jun/2026 | Módulo 2 (Revisor AI — DOM + PDF); Módulo 3 (Painel de Metas v1) |
| 1.4 | Jun/2026 | Módulo 3 corrigido (IDs únicos, storage global, painel "Todos" com KPIs); identidade visual Petrobras |
| 1.5 | Jun/2026 | README técnico completo: lógica do Guia SOX 2025, mapa completo do CPT (40 colunas), arquitetura backend |

---

## 17. Glossário

| Termo | Definição |
|-------|-----------|
| **ACL Analytics** | Ferramenta de seleção amostral aleatória utilizada pela AI Petrobras. Gera Seed (semente) que identifica o algoritmo de seleção para rastreabilidade |
| **ADC** | Auditores De Campo — equipe de auditoria externa (KPMG) |
| **Assertiva** | Critério que o controle precisa atender: Totalidade (T), Validade (VD), Registro (R), Cut-off (C), Valorização (VZ), Apresentação (A) |
| **BDOC** | Banco de Dados de Consolidação — repositório de evidências e papéis de trabalho |
| **CONF/CI** | Gerência de Conformidade de Controles Internos — coordena a certificação SOX |
| **CPT** | Control Performance Test — relatório do GRC-PC com todas as informações do controle SOX, exportado em Excel (118 colunas) |
| **CST** | Control Self-Testing — arquivo com resultados históricos e status dos testes |
| **Deficiência Significativa (DS)** | Deficiência de controle que é menos severa que uma Fraqueza Material, mas importante o suficiente para atenção da gestão |
| **Folha de Teste (FT)** | Documento padronizado para registro dos procedimentos, amostras e resultado do teste |
| **Fraqueza Material (MW)** | Deficiência grave de controle interno que resulta em possibilidade razoável de distorção relevante nas DFs |
| **GRC-PC** | Governance, Risk and Compliance — Process Control. Sistema SAP onde os controles são cadastrados e os testes são registrados |
| **IPE** | Informações Produzidas pela Entidade — relatórios/transações customizadas nos sistemas SAP utilizados nos controles |
| **Nível de Prova 05** | Nível que indica escopo da Auditoria Interna: autoavaliação + revisão CONF/CI + testes AI |
| **NPT** | Não Passível de Testes — controle que não possui quantidade mínima de eventos para amostragem |
| **PCAOB** | Public Company Accounting Oversight Board — regulador americano cujos padrões (AS 2301) fundamentam a metodologia SOX Petrobras |
| **Ponto Crítico** | Falha ou desvio registrado formalmente no RM SOX e no GRC-PC quando um controle é considerado Ineficaz |
| **Query BW** | Query SAP Business Warehouse utilizada para extração de população — deve ser previamente homologada/validada pela TIC como "Query SOX" |
| **RAI** | Sistema de Registro e Acompanhamento Integrado — plataforma SOX interna Petrobras |
| **RM SOX** | Sistema de workflow de auditoria (`rmsox.petrobras.com.br`) — onde as Folhas de Teste são abertas, preenchidas e revisadas |
| **Roll-Forward** | Extensão do período de teste quando as evidências foram coletadas antes do fechamento do exercício (regras PCAOB Alert 11/2013) |
| **RPA** | Relatório Preliminar de Auditoria — documento gerado pelo THEMISIA com base no CPT |
| **Seed (semente)** | Identificador numérico do algoritmo de seleção aleatória ACL — deve ser arquivado junto com as evidências de amostragem |
| **SOX** | Sarbanes-Oxley Act (2002) — lei americana que exige certificação anual de controles internos sobre relatórios financeiros |
| **Tamanho Amostral** | Quantidade de evidências a testar, definida pela Tabela Amostral SOX conforme frequência e categoria de risco (PCAOB AS 2301) |
| **Validação da População** | Extração independente realizada pelo auditor para verificar integridade e completude da população apresentada pelo auditado |

---

*THEMISIA · Auditoria Interna Petrobras · Certificação SOX 2026*
*HTML5 + JavaScript ES5 + SheetJS + PDF.js + GitHub Pages*
*Fundamentado no Guia Prático SOX 2025 e PCAOB AS 2301*
