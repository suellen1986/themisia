# THEMISIA – Auditoria Interna SOX · Empresa X
## Documentação Técnica do Projeto
 
> *THEMISIA — nome inspirado em Themis, deusa grega da justiça e conformidade. IA integrada para aumentar produtividade e tempestividade na Certificação SOX da Empresa X.*
 
> **Projeto:** THEMISIA – Automação da Auditoria Interna SOX  
> **Versão:** 1.4  
> **Responsável:** Auditoria Interna / CONF/CI  
> **Publicado em:** https://suellen1986.github.io/themisia/  
> **Data:** Junho de 2026
 
---
 
## Índice
 
1. [Visão Geral](#1-visão-geral)
2. [Arquitetura](#2-arquitetura)
3. [Módulo 1 — Gerador de RPA + Folha de Teste](#3-módulo-1--gerador-de-rpa--folha-de-teste)
4. [Módulo 2 — Revisor AI SOX](#4-módulo-2--revisor-ai-sox)
5. [Módulo 3 — Painel de Avaliação de Metas](#5-módulo-3--painel-de-avaliação-de-metas)
6. [Mapeamento do CPT](#6-mapeamento-do-cpt)
7. [Atributos do Teste](#7-atributos-do-teste)
8. [Validação da População](#8-validação-da-população)
9. [Tabela de Amostragem SOX](#9-tabela-de-amostragem-sox)
10. [Tecnologias](#10-tecnologias)
11. [Estrutura de Arquivos](#11-estrutura-de-arquivos)
12. [Como Usar](#12-como-usar)
13. [Histórico de Versões](#13-histórico-de-versões)
14. [Glossário](#14-glossário)
---
 
## 1. Visão Geral
 
THEMISIA automatiza o ciclo completo de certificação SOX da Empresa X em três módulos integrados:
 
| Módulo | Nome | Status |
|--------|------|--------|
| 1 | **Gerador de RPA + Folha de Teste** | ✅ v1.4 |
| 2 | **Revisor AI SOX** | ✅ v1.4 |
| 3 | **Painel de Avaliação de Metas** | ✅ v1.4 |
 
**Princípios:**
- Zero instalação — abre direto no navegador
- CPT carregado automaticamente do GitHub
- Sem API externa obrigatória
- Identidade visual Empresa X (azul `#003087`, amarelo `#F5A800`, verde `#009B3A`)
---
 
## 2. Arquitetura
 
```
┌─────────────────────────────────────────────────────────┐
│                   THEMISIA  v1.4                        │
│         https://suellen1986.github.io/themisia          │
│              Single-file · GitHub Pages                 │
└──────────────┬──────────────┬──────────────┬────────────┘
               │              │              │
        MÓDULO 1        MÓDULO 2       MÓDULO 3
     Gerador RPA      Revisor AI     Painel Metas
     Folha Teste      Rule-based     localStorage
     CPT auto-load    DOM + PDF      CSV export
```
 
**Fluxo do ciclo:**
```
CPT auto-carregado → Selecionar controle → Gerar RPA + FT
        → Preencher Folha de Teste → Revisor analisa
                → Coordenador avalia → Painel de metas
```
 
---
 
## 3. Módulo 1 — Gerador de RPA + Folha de Teste
 
### RPA — 10 Seções
 
| # | Seção | Fonte |
|---|-------|-------|
| 1 | Identificação do Controle | CPT |
| 2 | Objetivo do Controle | CPT |
| 3 | Descrição e Entendimento | CPT |
| 4 | Riscos Associados | CPT |
| 5 | Análise do Desenho | CPT + regras |
| 6 | Evidências Necessárias | CPT |
| 7 | Questionamentos ao Responsável | Gerado |
| 8 | Procedimento de Teste | CPT + tabela SOX |
| 🔍 | **Validação da População** | CPT campo BK |
| 9 | Conclusão Preliminar | Gerado |
 
### Folha de Teste — 10 Seções
 
| # | Seção | Detalhe |
|---|-------|---------|
| — | Cabeçalho | ID, controle, frequência, data |
| 1 | Objetivo do Teste | CPT |
| 2 | Descrição do Controle | Situação Atual |
| 3 | Riscos Mitigados | CPT |
| 4 | Critério de Aprovação | EFICAZ / INEFICAZ |
| 5 | Evidências a Solicitar | CPT |
| 6 | **Atributos do Teste** | SIM/NÃO + procedimentos + caminho de papéis |
| 7 | **Resultado por Amostra** | Tabela editável com radio buttons |
| 8 | Conclusão | Checkbox + justificativa |
| 9 | Constatações Críticas Identificadas | Texto livre |
| 10 | Sugestões de Plano de Ação | Texto livre |
 
### Atributos do Teste
 
Gerados por análise textual de **Situação Atual** (col. 61) e **Sugestão de Abordagem** (col. 21):
 
| Atributo | O que extrai | Exemplo |
|----------|-------------|---------|
| 1 — Prazo | Padrões de tempo no texto | "A EXECUÇÃO FOI REALIZADA ATÉ O FIM DO MÊS SUBSEQUENTE?" |
| 2 — Escopo | Objetos reais (BP, DRE, Imobilizado...) | "TODAS AS VARIAÇÕES DE BALANÇO PATRIMONIAL (BP), DRE FORAM ANALISADAS?" |
| 3 — Sistema | Sistema + transação SAP / módulo Blackline | "A CONSULTA FOI REALIZADA VIA TRANSAÇÃO VFX3 NO SAP?" |
| 4 — Evidência | Tipo de registro (Blackline, aprovação...) | "A EXECUÇÃO ESTÁ REGISTRADA NO BLACKLINE COM STATUS CONCLUÍDO?" |
| 5 — Exceções | Pendentes, desvios, variações | "OS ITENS PENDENTES DE CONTABILIZAÇÃO FORAM AVALIADOS?" |
 
Cada atributo exibe: pergunta → SIM/NÃO → "Como verificar" → campo de procedimentos executados → campo de caminho dos papéis de suporte.
 
---
 
## 4. Módulo 2 — Revisor AI SOX
 
Analisa a Folha de Teste preenchida e emite parecer estruturado **sem necessidade de API**.
 
### Duas formas de entrada
 
**Opção A (recomendada):** Revisar a Folha de Teste já aberta no THEMISIA — lê diretamente os radio buttons, textareas e checkboxes do DOM.
 
**Opção B:** Upload de PDF — extrai texto via PDF.js. Se o PDF não preservar texto, permite colar manualmente.
 
### Modos de Revisão
 
| Modo | Threshold aprovação | Postura |
|------|-------------------|---------|
| 🔵 Padrão | Score ≥ 65% | Revisor interno equilibrado |
| 🔴 Crítico (ADC) | Score ≥ 80% | Simula auditoria externa (Big 4) |
 
### Critérios Analisados
 
1. Conclusão declarada (EFICAZ/INEFICAZ)
2. Coerência entre atributos e conclusão
3. Quantidade amostral suficiente (tabela SOX)
4. Procedimentos executados descritos
5. Nível de detalhe dos procedimentos *(modo crítico)*
6. Caminhos dos papéis de suporte informados
7. Período das amostras identificado
8. Validação de integridade da população
9. Constatações críticas documentadas *(se há NÃO)*
10. Sugestões de Plano de Ação *(se há NÃO)*
11. Campos críticos CPT preenchidos
### Output do Parecer
 
- **Status:** 🟢 APROVADO / 🟡 APROVADO COM RESSALVAS / 🟠 DEVOLVIDO / 🔴 INEFICAZ
- **Score de qualidade** (0–100%)
- **Checklist** ✅/❌ por critério
- **Pontos para complementação** numerados
- **Perguntas ao auditor** geradas automaticamente
- **Contagem SIM/NÃO** vs. esperado
- **Conclusão fundamentada** do revisor
---
 
## 5. Módulo 3 — Painel de Avaliação de Metas
 
Coordenador/revisor avalia o trabalho do auditor após a revisão. Dados persistem em `localStorage` + variável global.
 
### Dimensões Avaliadas (1–5)
 
| Dimensão | Descrição |
|----------|-----------|
| Qualidade da Documentação | Clareza, organização e completude dos papéis |
| Suficiência das Evidências | As evidências suportam a conclusão |
| Clareza da Conclusão | Objetiva, fundamentada e consistente |
| Tempestividade | Entregue dentro do prazo do ciclo |
| Identificação de Riscos/Exceções | Desvios e pontos críticos identificados |
 
Nota abaixo de 3 exige justificativa obrigatória no campo Observações.
 
### Aba — Lançar Avaliação
 
- Campos: Auditor, Ciclo, Controle, Revisor, Observações
- Botão **↩ Usar controle selecionado** importa do Módulo 1
- 5 estrelas clicáveis por dimensão com badge colorido
- Média calculada automaticamente ao salvar
### Aba — Painel por Auditor
 
**Visão "Todos os auditores"** (default):
- KPIs gerais: Média do ciclo, Nº auditores, Nº avaliações
- Melhor dimensão do grupo / Dimensão com atenção
- Cards de média por dimensão com barra de progresso
- Ranking dos auditores clicável
**Visão individual** (ao selecionar auditor ou clicar no ranking):
- Cards de resumo por dimensão
- Tabela completa de controles avaliados
- Botão "← Todos os auditores"
### Aba — Painel do Ciclo
 
- Filtro por ciclo (ex: 2026-T2)
- Tabela ranking com todos os auditores × médias por dimensão
- Badges coloridos: 🟢 ≥4 · 🔵 ≥3 · 🟠 ≥2 · 🔴 <2
### Exportação
 
Botão **📥 Exportar CSV** disponível no painel por auditor (individual ou todos) e no painel do ciclo.
 
---
 
## 6. Mapeamento do CPT
 
Colunas-chave do CPT (118 colunas, índice base 0):
 
| Índice | Coluna | Uso |
|--------|--------|-----|
| 19 | ID do Controle | Busca |
| 20 | Nome do Controle | Busca |
| 21 | **Sugestão de Abordagem** | ⭐ Atributos |
| 31 | Evidência Sugerida | FT seção 5 |
| 34 | Frequência | Tabela amostral |
| 36 | Finalidade | Análise desenho |
| 37 | Automação | Análise desenho |
| 42 | Objetivo do Controle | RPA seção 2 |
| 46 | Risco(s) | Tabela riscos |
| 61 | **Situação Atual** | ⭐ Atributos |
| 62 | **Desc. Extração de Dados** | ⭐ Validação população |
| 82 | Ponto Crítico | Alertas |
| 117 | Contas / Assertivas | RPA seção 2 |
 
---
 
## 7. Atributos do Teste
 
A função `buildAtributos()` extrai termos literais da Situação Atual:
- **Sistemas:** Blackline (com módulo), SAP (com transação), Oracle, Excel
- **Objetos contábeis:** BP, DRE, Ativo Imobilizado, Intangível, Notas Explicativas, Faturamento, GEE, etc.
- **Ações:** variação, movimentação, conciliação, carregamento, pendentes
- **Comparativos:** período anterior, mês subsequente
---
 
## 8. Validação da População
 
A função `buildValidacaoPopulacao()` lê **Desc. Extração** (col. 62) e gera instruções específicas:
 
| Detectado no texto | Instrução gerada |
|-------------------|-----------------|
| Blackline – Módulo X (Entidade Y) | Acesse Blackline – Módulo X para Y, extraia relatório do período |
| Transação VFX3 no SAP | Execute VFX3 com mesmos parâmetros e compare |
| Por e-mail (demais empresas) | Solicite os e-mails, verifique remetente e data |
| Indicadores físicos (câmbio, produção...) | Compare com ANP, Banco Central, boletins |
| BDOC | Baixe arquivo base da grade e compare com upload |
| Planilha/Excel | Solicite planilha e verifique rastreabilidade |
 
**Diferença não justificada entre população do auditado e extração independente = INEFICAZ.**
 
---
 
## 9. Tabela de Amostragem SOX
 
PCAOB AS 2301 / Guia SOX Empresa X:
 
| Frequência | Amostras |
|------------|---------|
| Múltiplas vezes ao dia | 45 |
| Diário | 25 |
| Semanal | 5 |
| Quinzenal | 3 |
| Mensal | 2 |
| Trimestral | 2 |
| Semestral | 2 |
| Anual | 1 |
 
---
 
## 10. Tecnologias
 
| Tecnologia | Finalidade |
|-----------|-----------|
| HTML5 + CSS3 | Interface — identidade visual Empresa X |
| JavaScript ES5 | Lógica da aplicação (sem framework) |
| SheetJS (xlsx.js) 0.18.5 | Leitura do CPT.xlsx (embutido no HTML) |
| PDF.js 3.4.120 | Extração de texto de PDFs (CDN, opcional) |
| GitHub Pages | Publicação e hospedagem |
| Fetch API | Auto-carregamento do CPT.xlsx |
| localStorage | Persistência das avaliações do Módulo 3 |
 
---
 
## 11. Estrutura de Arquivos
 
```
themisia/  (GitHub: suellen1986/themisia)
├── index.html          ← Aplicação completa (3 módulos, autocontida)
├── CPT.xlsx            ← Base de controles SOX (anonimizada)
└── README.md           ← Esta documentação
```
 
---
 
## 12. Como Usar
 
### Módulo 1 — Gerar RPA
 
1. Acesse `https://suellen1986.github.io/themisia/`
2. CPT carrega automaticamente (`✅ X controles carregados`)
3. Digite o ID ou nome do controle → selecione
4. Clique **⚡ Gerar RPA + Folha de Teste**
5. Navegue pelas abas: **📄 RPA** · **📋 Folha de Teste** · **⚠️ Oportunidades**
6. Preencha na Folha de Teste: SIM/NÃO por atributo, procedimentos, caminhos das evidências
7. **🖨️ Imprimir** (salvar como PDF) ou **💾 Exportar HTML**
### Módulo 2 — Revisar
 
1. Com o controle selecionado no Módulo 1, role até **🔍 Revisor AI SOX**
2. **Opção A:** preencha a FT na aba 📋 e clique **⚡ Revisar Folha de Teste Atual**
3. **Opção B:** carregue o PDF ou cole o texto → **⚡ Revisar pelo Texto**
4. Escolha o modo (Padrão ou Crítico ADC)
5. O parecer aparece imediatamente com status, checklist, pontos e perguntas
### Módulo 3 — Avaliar
 
1. Role até **🏆 Painel de Avaliação de Metas**
2. **Lançar Avaliação:** preencha auditor, ciclo, controle → clique nas estrelas → **💾 Salvar**
3. **Painel por Auditor:** veja KPIs gerais ("Todos") ou detalhe individual
4. **Painel do Ciclo:** ranking completo por ciclo → **📥 Exportar CSV**
---
 
## 13. Histórico de Versões
 
| Versão | Data | Alterações |
|--------|------|-----------|
| 1.0 | Jun/2025 | Módulo 1 inicial — RPA + Folha de Teste + Oportunidades de Melhoria |
| 1.1 | Jun/2026 | Atributos derivados da Situação Atual; CPT auto-carregado; rebatizado THEMISIA |
| 1.2 | Jun/2026 | Validação da População; FT editável; Constatações Críticas; Sugestões de Plano de Ação |
| 1.3 | Jun/2026 | Módulo 2 (Revisor AI — DOM + PDF); Módulo 3 (Painel de Metas v1) |
| 1.4 | Jun/2026 | Módulo 3 corrigido (IDs únicos, storage global, painel "Todos" com KPIs); identidade visual Empresa X (logo BR, azul #003087, amarelo #F5A800) |
 
---
 
## 14. Glossário
 
| Termo | Definição |
|-------|-----------|
| **ADC** | Auditores De Campo — auditoria externa da Empresa X |
| **Atributo de Teste** | Pergunta SIM/NÃO respondível por evidência, derivada da Situação Atual |
| **BDOC** | Banco de Dados de Consolidação |
| **CPT** | Controle de Processo e Tecnologia — documento do controle SOX no sistema RAI |
| **DCC** | Demonstrações Contábeis Consolidadas |
| **Folha de Teste** | Documento padronizado para registro dos resultados do teste |
| **Constatação Crítica** | Falha ou oportunidade de melhoria registrada formalmente |
| **RAI** | Sistema de Registro e Acompanhamento Integrado — plataforma SOX Empresa X |
| **RPA** | Relatório Preliminar de Auditoria |
| **SOX** | Sarbanes-Oxley Act |
| **Tamanho Amostral** | Quantidade de evidências a testar (PCAOB AS 2301) |
| **Validação da População** | Extração independente para verificar integridade da população do auditado |
 
---
 
*THEMISIA · Auditoria Interna Empresa X · Certificação SOX 2026*  
*HTML5 + JavaScript ES5 + SheetJS + PDF.js + GitHub Pages*
