# THEMISIA – Auditoria Interna SOX · Petrobras
## Documentação Técnica do Projeto

> *THEMISIA — nome inspirado em Themis, deusa grega da justiça e conformidade. IA integrada para aumentar produtividade e tempestividade na Certificação SOX da Petrobras.*

> **Projeto:** Automação da Auditoria Interna SOX – Certificação Petrobras  
> **Módulo:** Gerador de Relatório Preliminar de Auditoria (RPA) + Folha de Teste  
> **Versão:** 1.0  
> **Responsável:** Auditoria Interna / CONF/CI  
> **Data:** Junho de 2025  

---

## Índice

1. [Visão Geral do Projeto](#1-visão-geral-do-projeto)
2. [Contexto e Motivação](#2-contexto-e-motivação)
3. [Arquitetura da Solução](#3-arquitetura-da-solução)
4. [Módulo 1 – Gerador de RPA (Este arquivo)](#4-módulo-1--gerador-de-rpa)
5. [Mapeamento do CPT](#5-mapeamento-do-cpt)
6. [Lógica de Geração do RPA](#6-lógica-de-geração-do-rpa)
7. [Lógica de Geração da Folha de Teste](#7-lógica-de-geração-da-folha-de-teste)
8. [Atributos do Teste – Geração Inteligente](#8-atributos-do-teste--geração-inteligente)
9. [Tabela de Amostragem SOX](#9-tabela-de-amostragem-sox)
10. [Validação de Campos Críticos](#10-validação-de-campos-críticos)
11. [Oportunidades de Melhoria](#11-oportunidades-de-melhoria)
12. [Tecnologias Utilizadas](#12-tecnologias-utilizadas)
13. [Estrutura de Arquivos](#13-estrutura-de-arquivos)
14. [Como Usar](#14-como-usar)
15. [Roadmap – Próximos Módulos](#15-roadmap--próximos-módulos)
16. [Limitações Conhecidas](#16-limitações-conhecidas)
17. [Glossário](#17-glossário)

---

## 1. Visão Geral do Projeto

Este projeto tem como objetivo automatizar o processo de certificação SOX da Petrobras, aumentando a produtividade da equipe de auditoria interna e padronizando a qualidade dos trabalhos executados.

A iniciativa está organizada em **três módulos de automação**:

| Módulo | Nome | Status |
|--------|------|--------|
| 1 | **Gerador de RPA + Folha de Teste** | ✅ Desenvolvido (v1.0) |
| 2 | **Revisor AI SOX** | 🔄 Planejado |
| 3 | **Painel de Avaliação de Metas** | 🔄 Planejado |

O Módulo 1 — objeto desta documentação — lê automaticamente os dados do **CPT (Controle de Processo e Tecnologia)** exportado do sistema RAI e gera, sem necessidade de API externa ou conexão com internet:

- Relatório Preliminar de Auditoria (RPA) estruturado em 9 seções
- Folha de Teste com atributos específicos do controle em formato de perguntas SIM/NÃO
- Diagnóstico de oportunidades de melhoria para campos críticos não preenchidos no CPT

---

## 2. Contexto e Motivação

### Problema

O processo atual de certificação SOX exige que cada auditor:

1. Leia manualmente o CPT para entender o controle a ser testado
2. Elabore um Relatório Preliminar de Auditoria (RPA) do zero
3. Construa a Folha de Teste com atributos adequados
4. Identifique manualmente campos críticos não preenchidos no CPT

Esse processo é **repetitivo, demorado e sujeito a variação de qualidade** entre auditores, gerando risco de gaps que podem ser questionados pela auditoria externa (ADC).

### Solução

Automatizar as etapas 1 a 4 usando os dados já existentes no CPT, aplicando:

- **Lógica baseada no Guia SOX Petrobras** para estrutura do RPA
- **Análise de texto da Abordagem e Situação Atual** para geração de atributos específicos do controle
- **Tabela de amostragem oficial SOX/PCAOB** para cálculo do tamanho amostral
- **Verificação automatizada de completude** dos campos críticos

### Benefícios Esperados

| Benefício | Impacto |
|-----------|---------|
| Redução do tempo de elaboração do RPA | ~70% |
| Padronização dos atributos de teste | Alta |
| Identificação proativa de lacunas no CPT | Antes do início do teste |
| Guia para auditores menos experientes | Reduz curva de aprendizado |
| Rastreabilidade e documentação | Melhora qualidade para revisão |

---

## 3. Arquitetura da Solução

```
┌─────────────────────────────────────────────────────┐
│               GERADOR DE RPA SOX                    │
│           Arquivo: Gerador_RPA_SOX.html             │
│              (Single-file application)              │
└────────────────────────┬────────────────────────────┘
                         │
          ┌──────────────┴──────────────┐
          │                             │
   ┌──────▼──────┐               ┌──────▼──────┐
   │  CPT.xlsx   │               │  SheetJS    │
   │  (entrada)  │──────────────▶│  (parser)   │
   └─────────────┘               └──────┬──────┘
                                        │
                          ┌─────────────▼─────────────┐
                          │      ENGINE DE REGRAS      │
                          │                            │
                          │  • Mapeamento de colunas   │
                          │  • Análise de texto        │
                          │  • Tabela amostral SOX     │
                          │  • Geração de perguntas    │
                          │  • Validação de campos     │
                          └──────┬──────────┬──────────┘
                                 │          │
                    ┌────────────▼──┐  ┌────▼────────────┐
                    │   RPA (HTML)  │  │ Folha de Teste  │
                    │   9 seções    │  │ + Atributos     │
                    └───────────────┘  └─────────────────┘
```

### Princípios de Design

- **Zero dependências externas em execução**: toda a lógica está contida no arquivo HTML
- **Offline-first**: funciona sem internet, sem API, sem servidor
- **Single-file**: distribuição por e-mail ou pasta compartilhada sem instalação
- **Baseado em dados reais**: usa exclusivamente campos do CPT — não inventa informações

---

## 4. Módulo 1 – Gerador de RPA

### Arquivo Principal

```
Gerador_RPA_SOX.html   (~670 KB, autocontido)
```

### Fluxo de Uso

```
1. Abrir o HTML no navegador (Chrome recomendado)
       ↓
2. Carregar o arquivo CPT.xlsx exportado do RAI
       ↓
3. Buscar o controle por ID numérico ou nome
       ↓
4. Verificar diagnóstico de campos críticos
       ↓
5. Clicar em "⚡ Gerar RPA + Folha de Teste"
       ↓
6. Revisar as 3 abas: RPA | Folha de Teste | Oportunidades de Melhoria
       ↓
7. Exportar como HTML ou imprimir
```

### Compatibilidade

| Navegador | Status |
|-----------|--------|
| Google Chrome | ✅ Recomendado |
| Microsoft Edge | ✅ Compatível |
| Mozilla Firefox | ✅ Compatível |
| Internet Explorer | ❌ Não suportado |

---

## 5. Mapeamento do CPT

O CPT exportado do RAI contém **118 colunas**. A ferramenta utiliza as seguintes colunas-chave (índice base 0):

| Índice | Nome da Coluna | Uso na Ferramenta |
|--------|---------------|-------------------|
| 1 | Unidade Organizacional | Identificação, cabeçalho |
| 6 | Macroprocesso | Identificação |
| 8 | Processo | Identificação |
| 11 | Subprocesso | Identificação |
| 13 | Risco Subprocesso | Análise de riscos |
| 19 | **ID do Controle** | Busca, identificação |
| 20 | **Controle** (nome) | Busca, identificação |
| 21 | **Sugestão de Abordagem** | ⭐ Geração de atributos |
| 25 | Categoria de Controle | RPA seção 3 |
| 26 | Nível de Prova | RPA seção 3 |
| 27 | Importância do Controle | Análise, badge |
| 28 | Risco do Controle | Análise, badge |
| 29 | Relevância | RPA |
| 31 | **Evidência Sugerida** | RPA seção 6, Folha seção 5 |
| 32 | Natureza do Controle | RPA seção 3 |
| 34 | **Frequência** | Tabela amostral SOX |
| 36 | Finalidade (Prev/Det) | Análise de desenho, atributos |
| 37 | Automação | Análise de desenho, atributos |
| 39 | Procedimento de Teste | Técnica de teste |
| 42 | **Objetivo do Controle** | RPA seção 2 — campo crítico |
| 44 | Desc. Objetivo Controle | RPA seção 2 |
| 45 | Categoria de Risco | Tabela de riscos |
| 46 | **Risco(s)** | Tabela de riscos — campo crítico |
| 47 | Descrição do Risco | Tabela de riscos |
| 56 | Status Design | RPA seção 5 |
| 60 | Classificação Design | RPA seção 5, badge |
| 61 | **Situação Atual** | ⭐ Geração de atributos |
| 62 | Desc. Extração de Dados | Atributos — base de dados |
| 63 | Finalidade do Controle (2) | Fallback finalidade |
| 64 | Automatização (2) | Fallback automação |
| 65 | Frequência (2) | Fallback frequência |
| 67 | Início da Operação | Identificação |
| 69 | Responsável – Nome | Identificação, questionamentos |
| 74 | Período | Cabeçalho |
| 75 | Ano | Cabeçalho |
| 82 | Ponto Crítico | Alertas na conclusão |
| 89 | Descrição Ponto Crítico | Alertas |
| 90 | Recomendação | RPA seção 5 |
| 106 | Empresa | Identificação |
| 107 | Diretoria | Identificação |
| 108 | Gerência Executiva | Identificação |
| **117** | **Contas / Assertivas** | RPA seção 2 — campo crítico |

> ⭐ **Campos mais importantes para qualidade dos atributos gerados:** Sugestão de Abordagem (col. 21) e Situação Atual (col. 61)

---

## 6. Lógica de Geração do RPA

O RPA é gerado em **9 seções fixas** conforme o Guia SOX Petrobras:

### Seção 1 — Identificação do Controle
Campos extraídos diretamente do CPT: ID, nome, empresa, diretoria, GE, unidade, macroprocesso, processo, subprocesso, responsável, grupo, período.

### Seção 2 — Objetivo do Controle
Extrai o campo Objetivo (col. 42) e Desc. Objetivo (col. 44), ambos podendo conter múltiplos valores separados por `|`.  
Exibe as Contas e Assertivas (col. 117) em lista separada.  
Campos ausentes geram alertas visuais em vermelho.

### Seção 3 — Descrição e Entendimento do Controle
Exibe: Categoria, Natureza, Finalidade, Automação, Frequência, Importância, Nível de Prova, Risco do Controle (com badge colorido), Sugestão de Abordagem e Situação Atual.

### Seção 4 — Riscos Associados
Monta tabela com os riscos do controle (col. 46), categorias (col. 45) e descrições (col. 47), todos separados por `|` no CPT.

### Seção 5 — Análise do Desenho do Controle
Avalia o desenho com base em: Classificação do Design, natureza preventiva/detectiva, grau de automação, segregação de funções. Destaca Pontos Críticos e Recomendações existentes.

### Seção 6 — Evidências Necessárias
Lista as evidências sugeridas no CPT (col. 31), separadas por `;`.

### Seção 7 — Questionamentos ao Responsável
Gera lista de perguntas-chave com base na frequência, finalidade, automação e riscos do controle. Inclui perguntas específicas para controles automatizados e detectivos.

### Seção 8 — Procedimento de Teste
Define:
- Técnica de teste (inferida do campo Procedimento de Teste col. 39)
- Tamanho amostral (calculado conforme Tabela SOX — ver seção 9)
- Passos detalhados do teste

### Seção 9 — Conclusão Preliminar
Alertas automáticos para: controle Chave, risco Alto, pontos críticos existentes e classificação do design.

---

## 7. Lógica de Geração da Folha de Teste

A Folha de Teste segue o modelo padrão Petrobras e contém **10 seções**:

| # | Seção | Conteúdo |
|---|-------|----------|
| Cabeçalho | Identificação | ID, nome, unidade, responsável, frequência, técnica, tamanho amostral, data |
| 1 | Objetivo do Teste | Derivado do DESC_OBJ / OBJETIVO + lista de contas/assertivas |
| 2 | Descrição do Controle | Campo SITUACAO_ATUAL (ou ABORDAGEM como fallback) |
| 3 | Riscos Mitigados | Lista dos riscos do CPT |
| 4 | Critério de Aprovação | EFICAZ ou INEFICAZ — sem graduações intermediárias |
| 5 | Evidências a Solicitar | Campo EVIDENCIA_SUG do CPT |
| 6 | **Atributos do Teste** | Perguntas SIM/NÃO geradas automaticamente (ver seção 8) |
| 7 | Resultado por Amostra | Tabela dinâmica com colunas por atributo |
| 8 | Conclusão do Teste | Checkboxes EFICAZ / INEFICAZ + campo justificativa |
| 9 | Pontos Críticos | Campo aberto |
| 10 | Recomendações | Campo aberto |

### Critério de Aprovação

```
EFICAZ   = TODOS os atributos de TODAS as amostras respondidos SIM
INEFICAZ = QUALQUER atributo respondido NÃO em qualquer amostra
```

Não existe conclusão "Eficaz Parcialmente". Toda falha é uma exceção que deve ser comunicada ao revisor e registrada como Ponto Crítico.

---

## 8. Atributos do Teste – Geração Inteligente

Esta é a funcionalidade mais sofisticada da ferramenta. Os atributos são **perguntas objetivas em SIM/NÃO** construídas a partir da análise textual dos campos **Sugestão de Abordagem** (col. 21) e **Situação Atual** (col. 61) do CPT.

### Por que perguntas e não descrições?

Os atributos representam verificações executáveis que o auditor faz para **cada amostra**. Uma pergunta SIM/NÃO é diretamente respondível com base na evidência, eliminando ambiguidade.

**Exemplo correto:** "AS ANÁLISES FORAM EXECUTADAS ATÉ O FIM DO MÊS SUBSEQUENTE ÀS APROPRIAÇÕES?"  
**Exemplo incorreto:** "Verificar se o controle mitiga o Risco RF01" *(abstrato demais)*

### Lógica de Geração (sem LLM)

A função `buildAtributos()` aplica **análise de padrões de texto (regex)** sobre os campos Abordagem e Situação Atual para gerar até 5 perguntas específicas:

#### Atributo 1 — Tempestividade / Prazo

Detecta referências de prazo no texto e gera a pergunta correspondente:

| Padrão detectado | Pergunta gerada |
|-----------------|-----------------|
| "mês subsequente" / "mês seguinte" | "A EXECUÇÃO FOI REALIZADA ATÉ O FIM DO MÊS SUBSEQUENTE AO PERÍODO?" |
| "fim do mês" / "final do mês" | "O CONTROLE FOI EXECUTADO DENTRO DO PRAZO (ATÉ O FIM DO MÊS)?" |
| "X dias após/antes" | "O CONTROLE FOI EXECUTADO DENTRO DO PRAZO DE X DIAS [APÓS/ANTES] O EVENTO?" |
| Nenhum prazo específico | Usa frequência do CPT: "...DENTRO DO PRAZO PREVISTO PARA A FREQUÊNCIA 'TRIMESTRAL'?" |

#### Atributo 2 — Escopo / Completude da Análise

Detecta o objeto da análise e gera pergunta de completude:

| Padrão detectado | Pergunta gerada |
|-----------------|-----------------|
| DCC + variação | "TODAS AS VARIAÇÕES RELEVANTES DAS DCC (DRE E BP) FORAM ANALISADAS E JUSTIFICADAS?" |
| Saldos contábeis | "TODAS AS VARIAÇÕES RELEVANTES DOS SALDOS CONTÁBEIS FORAM ANALISADAS?" |
| Imobilizado/Intangível | "TODAS AS MOVIMENTAÇÕES DO ATIVO IMOBILIZADO E INTANGÍVEL FORAM ANALISADAS?" |
| Notas explicativas | "AS INFORMAÇÕES DAS NOTAS EXPLICATIVAS FORAM REVISADAS EM SUA TOTALIDADE?" |
| Conciliação | "A CONCILIAÇÃO COBRIU TODAS AS CONTAS SEM LACUNAS E AS DIFERENÇAS FORAM TRATADAS?" |
| Genérico | Usa início da Abordagem como objeto da pergunta |

#### Atributo 3 — Evidência de Revisão / Aprovação

Detecta o sistema ou tipo de formalização esperada:

| Padrão detectado | Pergunta gerada |
|-----------------|-----------------|
| Blackline + revisão | "A REVISÃO FOI REGISTRADA NO BLACKLINE COM IDENTIFICAÇÃO DO RESPONSÁVEL E DATA?" |
| Sistema + aprovação | "A APROVAÇÃO FOI REGISTRADA NO SISTEMA COM APROVADOR E DATA ANTERIOR À EXECUÇÃO?" |
| Aprovação sem sistema | "A APROVAÇÃO FOI FORMALIZADA COM DATA ANTERIOR AO EVENTO POR PESSOA COM ALÇADA?" |
| Genérico | "A EXECUÇÃO ESTÁ DOCUMENTADA COM RESPONSÁVEL, DATA E CONTEÚDO RASTREÁVEIS?" |

#### Atributo 4 — Tratamento de Exceções *(condicional)*

Gerado quando o controle é **detectivo** ou o texto menciona "exceções", "desvios", "variações relevantes":

> "EVENTUAIS EXCEÇÕES OU VARIAÇÕES RELEVANTES IDENTIFICADAS FORAM FORMALMENTE DOCUMENTADAS E ENCAMINHADAS PARA TRATAMENTO?"

#### Atributo 5 — Fonte de Dados / Sistema *(condicional)*

Gerado quando há menção a sistemas específicos (Blackline, SAP) ou descrição de extração de dados:

> "OS DADOS UTILIZADOS FORAM EXTRAÍDOS DO SISTEMA [BLACKLINE/SAP] E CORRESPONDEM AO PERÍODO CORRETO?"

### Tabela Dinâmica de Amostras

As colunas da tabela de resultados são geradas dinamicamente com base nos atributos. Cada coluna recebe a pergunta como cabeçalho e células `[ ] SIM [ ] NÃO` para preenchimento pelo auditor.

---

## 9. Tabela de Amostragem SOX

A ferramenta implementa a tabela oficial de amostragem SOX conforme **PCAOB AS 2301** e o **Guia SOX Petrobras**:

| Frequência do Controle | Tamanho Amostral |
|------------------------|-----------------|
| Múltiplas vezes ao dia | **45 amostras** |
| Diário | **25 amostras** |
| Semanal | **5 amostras** |
| Quinzenal | **3 amostras** |
| Mensal | **2 amostras** |
| Trimestral | **2 amostras** |
| Semestral | **2 amostras** |
| Anual | **1 amostra** |
| Não definido | **2 amostras** (padrão conservador) |

A frequência é lida do campo col. 34 do CPT, com fallback para col. 65.

---

## 10. Validação de Campos Críticos

Ao selecionar um controle, a ferramenta verifica automaticamente **12 campos críticos** do CPT:

| Campo | Coluna CPT | Criticidade |
|-------|-----------|-------------|
| Objetivo do Controle | 42 | 🔴 Obrigatório — Guia SOX |
| Desc. Objetivo | 44 | 🟡 Importante |
| Contas / Assertivas | 117 | 🔴 Obrigatório — análise financeira |
| Risco(s) | 46 | 🔴 Obrigatório — fundamento do controle |
| Categoria de Risco | 45 | 🟡 Importante |
| Descrição do Risco | 47 | 🟡 Importante |
| Evidência Sugerida | 31 | 🟡 Importante |
| Procedimento de Teste | 39 | 🟡 Importante |
| Sugestão de Abordagem | 21 | 🔴 Crítico para atributos |
| Situação Atual | 61 | 🔴 Crítico para atributos |
| Frequência | 34 | 🔴 Obrigatório — tamanho amostral |
| Natureza do Controle | 32 | 🟡 Importante |

Campos não preenchidos são exibidos com badge ❌ e alimentam a aba **Oportunidades de Melhoria**.

---

## 11. Oportunidades de Melhoria

Para cada campo crítico não preenchido no CPT, a ferramenta gera automaticamente um cartão com:

- **Nome do campo** em falta
- **Impacto** da ausência (por que é importante para a auditoria SOX)
- **Ação recomendada** (solicitar preenchimento ao responsável)
- **Responsável** identificado no próprio CPT

Esta funcionalidade permite ao auditor, antes de iniciar o teste, identificar lacunas no CPT e comunicar ao gestor do controle, evitando retrabalho durante ou após a auditoria.

---

## 12. Tecnologias Utilizadas

| Tecnologia | Versão | Finalidade |
|-----------|--------|-----------|
| HTML5 | — | Interface e estrutura |
| CSS3 | — | Estilização (design Petrobras) |
| JavaScript ES5+ | — | Lógica da aplicação |
| **SheetJS (xlsx.js)** | 0.18.5 | Leitura do arquivo CPT.xlsx |
| — | — | Embutido no HTML (offline) |

### Por que JavaScript puro (sem framework)?

A escolha por JavaScript puro (sem React, Vue, Angular etc.) garante:
- **Zero instalação** — abre no navegador sem setup
- **Distribuição simples** — um único arquivo HTML
- **Funcionamento offline** — sem dependência de CDN ou servidor
- **Compatibilidade** — funciona em qualquer navegador moderno

### Biblioteca SheetJS

O SheetJS é a única dependência externa, **embutida diretamente no HTML** (~640 KB minificado). Ele é responsável por:
- Ler o arquivo `.xlsx` binário via FileReader API
- Converter para array JavaScript bidimensional
- Preservar valores de células multi-linha e com pipe `|`

---

## 13. Estrutura de Arquivos

```
SOX/
├── Gerador_RPA_SOX.html        ← Aplicação principal (autocontida)
├── CPT.xlsx                    ← Base de controles SOX exportada do RAI
├── xlsx.full.min.js            ← Biblioteca SheetJS (backup — não obrigatória)
├── README.md                   ← Esta documentação
└── Guia Prático SOx-2025.pdf  ← Guia SOX de referência
```

> **Nota:** O `xlsx.full.min.js` está embutido no HTML. O arquivo separado serve apenas como backup caso o HTML precise ser reconstruído.

---

## 14. Como Usar

### Pré-requisitos

- Navegador moderno (Chrome recomendado)
- Arquivo CPT.xlsx exportado do RAI (todos os controles do ciclo)

### Passo a Passo

**1. Abrir a ferramenta**
```
Clicar duas vezes em: Gerador_RPA_SOX.html
```

**2. Carregar o CPT**
- Clicar em "Escolher Arquivo"
- Selecionar o arquivo `CPT.xlsx`
- O arquivo carrega automaticamente ao ser selecionado
- Aguardar a mensagem: `✅ X controles carregados`

**3. Selecionar o controle**
- Digitar o ID numérico (ex: `50418865`) ou parte do nome (ex: `CCC08`)
- Selecionar o controle na lista de resultados
- Verificar o diagnóstico de campos críticos exibido abaixo

**4. Gerar os documentos**
- Clicar em `⚡ Gerar RPA + Folha de Teste`
- Navegar pelas abas:
  - `📄 Relatório RPA` — 9 seções do relatório preliminar
  - `📋 Folha de Teste` — modelo completo com atributos
  - `⚠️ Oportunidades de Melhoria` — lacunas do CPT

**5. Exportar / Imprimir**
- `🖨️ Imprimir` — abre diálogo de impressão (use "Salvar como PDF")
- `💾 Exportar HTML` — salva cópia independente com o ID do controle no nome

### Dicas de Uso

- Para **controles chave** de alto risco: revisar cuidadosamente os atributos gerados e ajustá-los se necessário antes de iniciar o teste
- Os **campos de texto** na Folha de Teste (conclusão, pontos críticos, recomendações) são editáveis diretamente no navegador antes de imprimir
- Para **múltiplos controles** do mesmo ciclo: a ferramenta pode ser usada repetidamente sem recarregar a página — basta limpar a seleção e buscar outro controle

---

## 15. Roadmap – Próximos Módulos

### Módulo 2 — Revisor AI SOX

**Objetivo:** Automatizar a revisão dos trabalhos de auditoria, auxiliando coordenadores e revisores.

**Funcionalidades previstas:**
- Carregar os dados do controle do CPT (mesma base do Módulo 1)
- Permitir o upload do detalhamento preenchido pelo auditor no sistema
- Analisar automaticamente se:
  - As evidências relatadas são aderentes ao objetivo do controle
  - O desenho do controle mitiga os riscos identificados
  - O trabalho está suficientemente documentado
- Gerar lista de questionamentos e pontos que precisam de esclarecimento
- Emitir parecer preliminar: **Eficaz / Ineficaz / Necessita esclarecimentos**
- Configuração de **modo crítico** (mais rigoroso) para simular postura da auditoria externa

**Diferencial:** O revisor será configurado para ser mais exigente que o auditor executor, mitigando riscos de gaps que poderiam ser questionados pela ADC.

---

### Módulo 3 — Painel de Avaliação de Metas

**Objetivo:** Substituir controles manuais (planilhas) de avaliação de performance da equipe.

**Funcionalidades previstas:**
- Interface para revisores (coordenadores) registrarem notas de 1 a 5 por controle revisado
- Campos avaliados: qualidade da documentação, suficiência de evidências, clareza da conclusão, tempestividade, identificação de riscos
- Notas abaixo de 2 exigem justificativa obrigatória
- Cálculo automático de médias ponderadas por auditor e por período
- Painel consolidado com visão por auditor, unidade e ciclo
- Campo de observações do revisor para cada avaliação
- Geração automática de resumo por auditor ao final do ciclo (sumarização LLM opcional)
- Exportação do relatório de metas em Excel ou PDF

---

## 16. Limitações Conhecidas

| Limitação | Descrição | Mitigação |
|-----------|-----------|-----------|
| Atributos baseados em regras | Sem LLM, os atributos são gerados por análise de padrões — podem não capturar nuances específicas de controles muito particulares | Auditor deve revisar e ajustar os atributos antes de usar |
| CPT com campos em branco | Campos críticos não preenchidos limitam a qualidade do RPA gerado | Ferramenta identifica e alerta para os campos ausentes |
| Separador `|` nos campos | Múltiplos valores em um campo são separados por `|` no CPT — a ferramenta faz o split correto, mas valores com `|` literal no texto podem ser quebrados incorretamente | Situação rara no CPT atual |
| Tamanho do arquivo | O HTML (~670KB) inclui a biblioteca SheetJS embutida | Tamanho aceitável; não impacta performance |
| Campos de texto não persistem | Os campos de conclusão na Folha de Teste não são salvos ao fechar o navegador | Exportar o HTML antes de fechar usando o botão "Exportar HTML" |

---

## 17. Glossário

| Termo | Definição |
|-------|-----------|
| **ADC** | Auditores De Campo — auditoria externa da Petrobras |
| **Atributo de Teste** | Pergunta SIM/NÃO que o auditor responde para cada amostra durante o teste |
| **CPT** | Controle de Processo e Tecnologia — documento que descreve cada controle SOX no sistema RAI |
| **DCC** | Demonstrações Contábeis Consolidadas |
| **Folha de Teste** | Documento padronizado utilizado pelo auditor para registrar os resultados do teste de controle |
| **Gap de Controle** | Falha ou lacuna no controle que pode resultar em distorção nas demonstrações financeiras |
| **Ponto Crítico** | Observação registrada no sistema quando o controle apresenta falha ou oportunidade de melhoria significativa |
| **RAI** | Sistema de Registro e Acompanhamento Integrado — plataforma SOX da Petrobras |
| **RPA** | Relatório Preliminar de Auditoria — documento elaborado antes do início do teste para guiar o auditor |
| **SOX** | Sarbanes-Oxley Act — lei americana que exige certificação de controles internos sobre relatórios financeiros |
| **Tamanho Amostral** | Quantidade de evidências a serem testadas, determinada conforme a frequência do controle pela tabela PCAOB |

---

## Histórico de Versões

| Versão | Data | Alterações |
|--------|------|-----------|
| 1.0 | Jun/2025 | Versão inicial — RPA + Folha de Teste + Oportunidades de Melhoria |

---

*Desenvolvido para a Auditoria Interna da Petrobras — Projeto de Melhoria da Certificação SOX 2025*  
*Tecnologia: HTML5 + JavaScript + SheetJS — Sem dependências externas em execução*
