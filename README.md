# THEMISIA – Auditoria Interna SOX · Petrobras
## Documentação Técnica do Projeto

> *THEMISIA — nome inspirado em Themis, deusa grega da justiça e conformidade. IA integrada para aumentar produtividade e tempestividade na Certificação SOX da Petrobras.*

> **Projeto:** THEMISIA – Automação da Auditoria Interna SOX – Certificação Petrobras  
> **Módulo:** Gerador de RPA + Folha de Teste  
> **Versão:** 1.2  
> **Responsável:** Auditoria Interna / CONF/CI  
> **Data:** Junho de 2026  

---

## Índice

1. [Visão Geral do Projeto](#1-visão-geral-do-projeto)
2. [Contexto e Motivação](#2-contexto-e-motivação)
3. [Arquitetura da Solução](#3-arquitetura-da-solução)
4. [Módulo 1 – Gerador de RPA](#4-módulo-1--gerador-de-rpa)
5. [Mapeamento do CPT](#5-mapeamento-do-cpt)
6. [Lógica de Geração do RPA](#6-lógica-de-geração-do-rpa)
7. [Lógica de Geração da Folha de Teste](#7-lógica-de-geração-da-folha-de-teste)
8. [Atributos do Teste – Geração Inteligente](#8-atributos-do-teste--geração-inteligente)
9. [Validação da População](#9-validação-da-população)
10. [Tabela de Amostragem SOX](#10-tabela-de-amostragem-sox)
11. [Validação de Campos Críticos](#11-validação-de-campos-críticos)
12. [Oportunidades de Melhoria](#12-oportunidades-de-melhoria)
13. [Tecnologias Utilizadas](#13-tecnologias-utilizadas)
14. [Estrutura de Arquivos](#14-estrutura-de-arquivos)
15. [Como Usar](#15-como-usar)
16. [Roadmap – Próximos Módulos](#16-roadmap--próximos-módulos)
17. [Limitações Conhecidas](#17-limitações-conhecidas)
18. [Histórico de Versões](#18-histórico-de-versões)
19. [Glossário](#19-glossário)

---

## 1. Visão Geral do Projeto

Este projeto tem como objetivo automatizar o processo de certificação SOX da Petrobras, aumentando a produtividade da equipe de auditoria interna e padronizando a qualidade dos trabalhos executados.

A iniciativa está organizada em **três módulos de automação**:

| Módulo | Nome | Status |
|--------|------|--------|
| 1 | **Gerador de RPA + Folha de Teste** | ✅ Desenvolvido (v1.2) |
| 2 | **Revisor AI SOX** | 🔄 Planejado |
| 3 | **Painel de Avaliação de Metas** | 🔄 Planejado |

O Módulo 1 lê automaticamente os dados do **CPT (Controle de Processo e Tecnologia)** exportado do sistema RAI e gera, sem necessidade de API externa:

- Relatório Preliminar de Auditoria (RPA) estruturado em 10 seções
- Seção de **Sugestão de Validação da População** com instruções específicas por sistema
- Folha de Teste com atributos derivados da Situação Atual do controle
- Tabela de resultados por amostra completamente editável
- Diagnóstico de oportunidades de melhoria para campos críticos não preenchidos

A ferramenta está publicada via **GitHub Pages** em: `https://suellen1986.github.io/themisia/`  
O CPT é carregado **automaticamente** ao abrir a página — sem necessidade de upload manual.

---

## 2. Contexto e Motivação

### Problema

O processo atual de certificação SOX exige que cada auditor:

1. Leia manualmente o CPT para entender o controle a ser testado
2. Elabore o RPA do zero
3. Construa a Folha de Teste com atributos adequados
4. Defina como validar a integridade da população utilizada pelo auditado
5. Identifique campos críticos não preenchidos no CPT

Esse processo é **repetitivo, demorado e sujeito a variação de qualidade** entre auditores.

### Solução

Automatizar as etapas 1 a 5 usando os dados já existentes no CPT, aplicando:

- **Lógica baseada no Guia SOX Petrobras** para estrutura do RPA
- **Extração literal de termos** da Situação Atual e Desc. Extração para gerar atributos e orientações de validação de população específicos por controle
- **Tabela de amostragem oficial SOX/PCAOB AS 2301** para cálculo do tamanho amostral
- **Verificação automatizada de completude** dos campos críticos

### Benefícios

| Benefício | Impacto |
|-----------|---------|
| Redução do tempo de elaboração do RPA | ~70% |
| Atributos de teste derivados da realidade do controle | Alta especificidade |
| Orientação de validação de população por sistema (SAP, Blackline, BDOC, e-mail) | Reduz risco de população inválida |
| Identificação proativa de lacunas no CPT | Antes do início do teste |
| Folha de Teste editável e estruturada | Pronta para preenchimento e impressão |

---

## 3. Arquitetura da Solução

```
┌─────────────────────────────────────────────────────┐
│                    THEMISIA                         │
│            Arquivo: index.html                      │
│     Publicado em: suellen1986.github.io/themisia    │
│           (Single-file application)                 │
└────────────────────────┬────────────────────────────┘
                         │
          ┌──────────────┴──────────────┐
          │                             │
   ┌──────▼──────┐               ┌──────▼──────┐
   │  CPT.xlsx   │               │  SheetJS    │
   │  (auto-     │──────────────▶│  (parser)   │
   │  carregado) │               └──────┬──────┘
   └─────────────┘                      │
                          ┌─────────────▼─────────────┐
                          │      ENGINE DE REGRAS      │
                          │                            │
                          │  • Mapeamento de colunas   │
                          │  • Extração literal de     │
                          │    termos (Sit. Atual +    │
                          │    Desc. Extração)         │
                          │  • Tabela amostral SOX     │
                          │  • Validação de população  │
                          │  • Validação de campos     │
                          └──────┬──────────┬──────────┘
                                 │          │
                    ┌────────────▼──┐  ┌────▼────────────┐
                    │  RPA (HTML)   │  │  Folha de Teste  │
                    │  10 seções    │  │  Editável        │
                    └───────────────┘  └──────────────────┘
```

### Princípios de Design

- **Zero dependências externas em execução**: toda a lógica está contida no arquivo HTML
- **Publicado via GitHub Pages**: acesso por URL, sem instalação
- **CPT auto-carregado**: o CPT.xlsx é buscado automaticamente do repositório ao abrir a página
- **Baseado em dados reais**: usa exclusivamente campos do CPT — não inventa informações

---

## 4. Módulo 1 – Gerador de RPA

### Arquivo Principal

```
index.html   (autocontido — SheetJS embutido)
```

### Fluxo de Uso

```
1. Acessar https://suellen1986.github.io/themisia/
       ↓
2. CPT.xlsx carrega automaticamente
       ↓
3. Buscar o controle por ID ou nome
       ↓
4. Verificar diagnóstico de campos críticos
       ↓
5. Clicar em "⚡ Gerar RPA + Folha de Teste"
       ↓
6. Revisar as 3 abas: RPA | Folha de Teste | Oportunidades de Melhoria
       ↓
7. Preencher procedimentos, caminhos de papéis e conclusões na Folha de Teste
       ↓
8. Exportar como HTML ou imprimir
```

---

## 5. Mapeamento do CPT

O CPT exportado do RAI contém **118 colunas**. Colunas-chave utilizadas (índice base 0):

| Índice | Nome da Coluna | Uso |
|--------|---------------|-----|
| 19 | **ID do Controle** | Busca, identificação |
| 20 | **Controle** (nome) | Busca, identificação |
| 21 | **Sugestão de Abordagem** | ⭐ Geração de atributos |
| 31 | Evidência Sugerida | RPA seção 6, Folha seção 5 |
| 34 | **Frequência** | Tabela amostral SOX |
| 36 | Finalidade (Prev/Det) | Análise de desenho |
| 37 | Automação | Análise de desenho |
| 39 | Procedimento de Teste | Técnica de teste |
| 42 | **Objetivo do Controle** | RPA seção 2 |
| 46 | **Risco(s)** | Tabela de riscos |
| 61 | **Situação Atual** | ⭐ Geração de atributos |
| 62 | **Desc. Extração de Dados** | ⭐ Validação de população |
| 82 | Ponto Crítico | Alertas |
| 90 | Recomendação | RPA seção 5 |
| 117 | **Contas / Assertivas** | RPA seção 2 |

> ⭐ **Campos mais importantes:** Situação Atual (61), Sugestão de Abordagem (21) e Desc. Extração (62)

---

## 6. Lógica de Geração do RPA

O RPA é gerado em **10 seções** conforme o Guia SOX Petrobras:

| # | Seção | Conteúdo |
|---|-------|----------|
| 1 | Identificação do Controle | ID, nome, empresa, responsável, período |
| 2 | Objetivo do Controle | Objetivo, desc. objetivo, contas/assertivas |
| 3 | Descrição e Entendimento | Categoria, natureza, frequência, automação, abordagem, situação atual |
| 4 | Riscos Associados | Tabela de riscos com categoria e descrição |
| 5 | Análise do Desenho | Design, preventivo/detectivo, automação, segregação |
| 6 | Evidências Necessárias | Lista de evidências sugeridas no CPT |
| 7 | Questionamentos ao Responsável | Perguntas geradas automaticamente |
| 8 | Procedimento de Teste | Técnica, tamanho amostral SOX, passos |
| 🔍 | **Sugestão de Validação da População** | Instruções específicas por sistema (ver seção 9) |
| 9 | Conclusão Preliminar | Alertas automáticos de risco, design e pontos críticos |

---

## 7. Lógica de Geração da Folha de Teste

A Folha de Teste contém **10 seções** e é totalmente editável no navegador:

| # | Seção | Conteúdo |
|---|-------|----------|
| Cabeçalho | Identificação | ID, controle, unidade, responsável, frequência, data |
| 1 | Objetivo do Teste | Derivado do CPT + contas/assertivas |
| 2 | Descrição do Controle | Situação Atual (fallback: Abordagem) |
| 3 | Riscos Mitigados | Lista dos riscos do CPT |
| 4 | Critério de Aprovação | EFICAZ ou INEFICAZ — sem graduações |
| 5 | Evidências a Solicitar | Campo EVIDENCIA_SUG do CPT |
| 6 | **Atributos do Teste** | Perguntas SIM/NÃO + procedimentos + caminho dos papéis |
| 7 | **Resultado por Amostra** | Tabela editável com inputs e radio buttons |
| 8 | Conclusão do Teste | Checkboxes + campo justificativa |
| 9 | **Constatações Críticas Identificadas** | Campo de texto livre |
| 10 | **Sugestões de Plano de Ação** | Campo de texto livre |

### Seção 6 — Atributos (detalhe por atributo)

Cada atributo exibe:
1. **Pergunta** (CAPS) — respondível SIM / NÃO com radio buttons
2. **Como verificar** — dica de auditoria específica ao atributo
3. **📝 Procedimentos executados** — textarea para o auditor descrever o que foi feito
4. **📁 Caminho dos papéis de suporte** — campo para caminho de rede ou SharePoint

### Seção 7 — Resultado por Amostra (detalhe)

Cada linha da tabela contém campos editáveis:
- **Período/Data** — input texto (`MM/AAAA`)
- **Identificação da Amostra** — input texto
- **Por atributo** — radio buttons SIM / NÃO independentes
- **Resultado Final** — radio buttons EFICAZ / INEFICAZ
- **Observações** — input texto

### Critério de Aprovação

```
EFICAZ   = TODOS os atributos de TODAS as amostras respondidos SIM
INEFICAZ = QUALQUER atributo respondido NÃO em qualquer amostra
```

---

## 8. Atributos do Teste – Geração Inteligente

A função `buildAtributos()` extrai termos literais dos campos **Situação Atual** (col. 61) e **Sugestão de Abordagem** (col. 21) para gerar até **5 perguntas específicas** do controle.

### Por que perguntas e não descrições?

Perguntas SIM/NÃO são diretamente respondíveis com base na evidência, eliminando ambiguidade.

**Exemplo:** "AS VARIAÇÕES RELEVANTES DE BALANÇO PATRIMONIAL (BP), DRE FORAM ANALISADAS E DEVIDAMENTE JUSTIFICADAS NO PERÍODO?"

### Os 5 Atributos Gerados

#### Atributo 1 — Tempestividade / Prazo

| Padrão detectado na Situação Atual | Pergunta gerada |
|-------------------------------------|-----------------|
| "mês subsequente" / "mês seguinte" | "A EXECUÇÃO FOI REALIZADA ATÉ O FIM DO MÊS SUBSEQUENTE AO PERÍODO?" |
| "fim do mês" | "O CONTROLE FOI EXECUTADO DENTRO DO PRAZO PREVISTO (ATÉ O FIM DO MÊS)?" |
| "X dias após/antes" | "O CONTROLE FOI EXECUTADO DENTRO DO PRAZO DE X DIAS APÓS O PERÍODO?" |
| Nenhum prazo específico | Usa frequência do CPT: "...DENTRO DO PRAZO ESPERADO PARA A FREQUÊNCIA 'TRIMESTRAL'?" |

> ⚠️ O nome do responsável **não é incluído** na pergunta — o atributo valida o prazo, não o executor.

#### Atributo 2 — Escopo / Completude

Extrai objetos reais da Situação Atual (BP, DRE, Ativo Imobilizado, Notas Explicativas, etc.) e combina com o tipo de ação (variação, movimentação, conciliação, integridade).

**Exemplo CTR01:** "TODAS AS VARIAÇÕES RELEVANTES DE BALANÇO PATRIMONIAL (BP), DRE FORAM ANALISADAS E DEVIDAMENTE JUSTIFICADAS NO PERÍODO?"

**Exemplo Imobilizado:** "TODAS AS MOVIMENTAÇÕES DE ATIVO IMOBILIZADO, INTANGÍVEL, NOTAS EXPLICATIVAS (ADIÇÃO, BAIXA, TRANSFERÊNCIA, DEPRECIAÇÃO) FORAM ANALISADAS NO PERÍODO SEM OMISSÕES?"

#### Atributo 3 — Sistema / Fonte de Dados

Extrai sistema (Blackline, SAP), módulo e transação SAP diretamente do texto:

**Exemplo Blackline:** "O CARREGAMENTO DOS DADOS NO BLACKLINE – MÓDULO VARIÂNCIA FOI EXECUTADO CORRETAMENTE, COM DADOS DO PERÍODO CORRETO E SEM ERROS?"

**Exemplo SAP VFX3:** "A CONSULTA FOI REALIZADA VIA TRANSAÇÃO VFX3 NO SAP, COBRINDO TODOS OS DOCUMENTOS DO PERÍODO SEM FILTROS INCORRETOS?"

#### Atributo 4 — Evidência / Registro

Verifica se o sistema e o tipo de formalização estão refletidos na pergunta:

**Exemplo Blackline:** "A EXECUÇÃO DO CONTROLE ESTÁ REGISTRADA NO BLACKLINE COM STATUS CONCLUÍDO, IDENTIFICAÇÃO DO RESPONSÁVEL E DATA DO PERÍODO?"

#### Atributo 5 — Tratamento de Exceções

Gerado com base no tipo de exceção detectada na Situação Atual (pendentes, desvios, variações):

**Exemplo faturamento:** "OS ITENS DE CONTAS A RECEBER, FATURAMENTO IDENTIFICADOS COMO PENDENTES FORAM AVALIADOS E, QUANDO NECESSÁRIO, O REGISTRO MANUAL FOI REALIZADO?"

---

## 9. Validação da População

A seção **🔍 Sugestão de Validação da População** é gerada no RPA entre o Procedimento de Teste e a Conclusão Preliminar.

### Por que validar a população?

A integridade da população é pré-requisito para a validade do teste. Se a população utilizada pelo auditado na época da execução for diferente da extração independente realizada pelo auditor, **o resultado do controle é INEFICAZ**.

### Lógica de Geração

A função `buildValidacaoPopulacao()` lê o campo **Desc. Extração de Dados** (col. 62 — campo BK do CPT) e extrai literalmente:

| Elemento detectado | Instrução gerada |
|-------------------|-----------------|
| Blackline – Módulo X (Entidade Y) | "Acesse o Blackline – Módulo X para as entidades Y e extraia o relatório de execuções do período..." |
| Transação SAP (ex: VFX3) | "No SAP, execute a transação VFX3 com os mesmos parâmetros do auditado: empresa, período e filtros..." |
| Por e-mail (demais empresas) | "Solicite ao auditado os e-mails do período. Verifique: remetente, data e consistência com as justificativas..." |
| Indicadores físicos (câmbio, produção...) | "Solicite as fontes primárias: ANP para produção, Banco Central para câmbio, boletins internos para Brent..." |
| BDOC | "Acesse o BDOC e baixe o arquivo base da grade de consolidação. Compare com o arquivo do upload..." |
| Planilha/Excel | "Solicite a planilha e verifique rastreabilidade: origem, data, filtros e período coberto..." |

### Os 5 Passos da Validação

1. **Identificar e acessar a(s) fonte(s) primária(s)** — instruções específicas por sistema
2. **Verificar completude do período** — sem lacunas de datas ou registros excluídos
3. **Comparar totais** — quantidade e valores da extração independente vs. população do auditado
4. **Investigar diferenças** — diferença não justificada = INEFICAZ
5. **Documentar** — registrar parâmetros, filtros, data e total da extração como evidência

---

## 10. Tabela de Amostragem SOX

Implementada conforme **PCAOB AS 2301** e o **Guia SOX Petrobras**:

| Frequência | Tamanho Amostral |
|------------|-----------------|
| Múltiplas vezes ao dia | **45 amostras** |
| Diário | **25 amostras** |
| Semanal | **5 amostras** |
| Quinzenal | **3 amostras** |
| Mensal | **2 amostras** |
| Trimestral | **2 amostras** |
| Semestral | **2 amostras** |
| Anual | **1 amostra** |
| Não definido | **2 amostras** (conservador) |

---

## 11. Validação de Campos Críticos

Ao selecionar um controle, a ferramenta verifica automaticamente campos críticos do CPT:

| Campo | Coluna | Criticidade |
|-------|--------|-------------|
| Objetivo do Controle | 42 | 🔴 Obrigatório |
| Contas / Assertivas | 117 | 🔴 Obrigatório |
| Risco(s) | 46 | 🔴 Obrigatório |
| Sugestão de Abordagem | 21 | 🔴 Crítico para atributos |
| Situação Atual | 61 | 🔴 Crítico para atributos |
| Desc. Extração de Dados | 62 | 🔴 Crítico para validação de população |
| Frequência | 34 | 🔴 Obrigatório — tamanho amostral |
| Evidência Sugerida | 31 | 🟡 Importante |
| Procedimento de Teste | 39 | 🟡 Importante |

---

## 12. Oportunidades de Melhoria

Para cada campo crítico não preenchido, a ferramenta gera um cartão com:

- Nome do campo em falta
- Impacto da ausência na auditoria SOX
- Ação recomendada
- Responsável identificado no CPT

Permite ao auditor comunicar lacunas ao gestor do controle **antes** de iniciar o teste.

---

## 13. Tecnologias Utilizadas

| Tecnologia | Finalidade |
|-----------|-----------|
| HTML5 + CSS3 | Interface e estilização |
| JavaScript ES5 | Lógica da aplicação (sem framework) |
| **SheetJS (xlsx.js) 0.18.5** | Leitura do CPT.xlsx (embutido no HTML) |
| **GitHub Pages** | Publicação e hospedagem do CPT |
| Fetch API | Auto-carregamento do CPT.xlsx ao abrir a página |

---

## 14. Estrutura de Arquivos

```
themisia/  (repositório GitHub)
├── index.html              ← Aplicação principal (autocontida)
├── CPT.xlsx                ← Base de controles SOX (anonimizada)
├── README.md               ← Esta documentação
└── Guia Prático SOx-2025.pdf
```

---

## 15. Como Usar

**1. Acessar a ferramenta**
```
https://suellen1986.github.io/themisia/
```
O CPT carrega automaticamente. Aguardar: `✅ X controles carregados automaticamente`

**2. Buscar o controle**
- Digitar ID ou parte do nome (ex: `CCC08`, `Revisar variações`)
- Selecionar o controle na lista

**3. Gerar os documentos**
- Clicar em `⚡ Gerar RPA + Folha de Teste`
- Navegar pelas abas: `📄 RPA` | `📋 Folha de Teste` | `⚠️ Oportunidades de Melhoria`

**4. Preencher a Folha de Teste**
- Em cada atributo: marcar SIM/NÃO, descrever os procedimentos executados e informar o caminho dos papéis de suporte
- Na tabela de resultados: preencher período, identificação da amostra e marcar SIM/NÃO por atributo
- Seção 9: registrar constatações críticas
- Seção 10: registrar sugestões de plano de ação

**5. Exportar**
- `🖨️ Imprimir` — salvar como PDF
- `💾 Exportar HTML` — salva cópia independente

---

## 16. Roadmap – Próximos Módulos

### Módulo 2 — Revisor AI SOX

Automatizar a revisão dos trabalhos, auxiliando coordenadores e revisores:
- Analisar se evidências são aderentes ao objetivo do controle
- Verificar se o desenho mitiga os riscos identificados
- Emitir parecer: Eficaz / Ineficaz / Necessita esclarecimentos
- Modo crítico para simular postura da auditoria externa (ADC)

### Módulo 3 — Painel de Avaliação de Metas

Substituir planilhas manuais de avaliação de performance:
- Notas de 1 a 5 por controle revisado (qualidade, suficiência, clareza, tempestividade)
- Médias ponderadas por auditor e período
- Painel consolidado por auditor, unidade e ciclo
- Exportação em Excel ou PDF

---

## 17. Limitações Conhecidas

| Limitação | Mitigação |
|-----------|-----------|
| Atributos baseados em regras — podem não capturar nuances de controles muito específicos | Auditor deve revisar os atributos gerados antes de usar |
| CPT com campos em branco limita qualidade do RPA | Ferramenta identifica e alerta os campos ausentes |
| Campos preenchidos na Folha de Teste não persistem ao fechar o navegador | Exportar o HTML antes de fechar |
| CPT atualizado exige novo upload para o GitHub | Processo manual — substituir o arquivo no repositório |

---

## 18. Histórico de Versões

| Versão | Data | Alterações |
|--------|------|-----------|
| 1.0 | Jun/2025 | Versão inicial — RPA + Folha de Teste + Oportunidades de Melhoria |
| 1.1 | Jun/2026 | Atributos derivados literalmente da Situação Atual; remoção do nome do responsável dos atributos; CPT auto-carregado via GitHub Pages; rebatizado como THEMISIA |
| 1.2 | Jun/2026 | Seção de Validação da População com instruções por sistema (SAP/transação, Blackline/módulo, e-mail, indicadores físicos, BDOC); Folha de Teste com campos editáveis por amostra; campos de procedimentos e caminho de papéis por atributo; seção 9 renomeada para Constatações Críticas; seção 10 renomeada para Sugestões de Plano de Ação |

---

## 19. Glossário

| Termo | Definição |
|-------|-----------|
| **ADC** | Auditores De Campo — auditoria externa da Petrobras |
| **Atributo de Teste** | Pergunta SIM/NÃO que o auditor responde para cada amostra |
| **BDOC** | Banco de Dados de Consolidação — fonte dos dados de consolidação contábil |
| **CPT** | Controle de Processo e Tecnologia — documento que descreve cada controle SOX no sistema RAI |
| **DCC** | Demonstrações Contábeis Consolidadas |
| **Folha de Teste** | Documento padronizado para registro dos resultados do teste de controle |
| **Constatação Crítica** | Observação registrada quando o controle apresenta falha ou oportunidade de melhoria significativa |
| **RAI** | Sistema de Registro e Acompanhamento Integrado — plataforma SOX da Petrobras |
| **RPA** | Relatório Preliminar de Auditoria — documento elaborado antes do teste |
| **SOX** | Sarbanes-Oxley Act — lei que exige certificação de controles internos sobre relatórios financeiros |
| **Tamanho Amostral** | Quantidade de evidências a testar, conforme frequência e tabela PCAOB AS 2301 |
| **Validação da População** | Procedimento de extração independente para confirmar que a população usada pelo auditado é íntegra |

---

*Desenvolvido para a Auditoria Interna da Petrobras — Projeto THEMISIA · Certificação SOX 2026*  
*Tecnologia: HTML5 + JavaScript ES5 + SheetJS + GitHub Pages — Sem dependências externas em execução*
