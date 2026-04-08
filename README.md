# 📊 MTA Planner — DP6

Ferramenta interativa para planejamento e organização de projetos de **Multi-Touch Attribution (MTA)**, estruturada em 4 etapas com geração automática de validações, queries BigQuery e tarefas por área (DE / DS / BA). Inclui o **MTA Source Validator**, módulo independente para verificação da qualidade das fontes de dados antes da construção da SOT.

---

## 🚀 Acesso rápido

**Produção:** `https://seu-usuario.github.io/mta-planner`

> Substitua `seu-usuario` pelo seu usuário do GitHub após configurar o Pages.

---

## 📁 Estrutura do projeto

```
mta-planner/
│
├── index.html        # Aplicação principal (single-file)
└── README.md         # Este arquivo
```

---

## ⚙️ Setup local (primeira vez)

### Pré-requisitos
- [Git](https://git-scm.com/downloads) instalado
- Conta no [GitHub](https://github.com)

### 1. Clone o repositório

```bash
git clone https://github.com/seu-usuario/mta-planner.git
cd mta-planner
```

### 2. Abra localmente

Não precisa de servidor — basta abrir o arquivo no navegador:

```bash
# Mac
open index.html

# Windows
start index.html

# Linux
xdg-open index.html
```

---

## 🔄 Fluxo de atualização

Sempre que fizer uma alteração no `index.html`:

```bash
# 1. Adicionar as mudanças
git add index.html

# 2. Commitar com uma descrição clara
git commit -m "feat: descrição do que foi alterado"

# 3. Enviar para o GitHub
git push origin main
```

O GitHub Pages atualiza automaticamente em ~1-2 minutos após o push.

---

## 🌐 Configurando o GitHub Pages (uma vez só)

1. Acesse o repositório no GitHub
2. **Settings** → **Pages** (menu lateral esquerdo)
3. Em **Source**: selecione `Deploy from a branch`
4. Em **Branch**: selecione `main` → pasta `/ (root)`
5. Clique em **Save**

O link ficará disponível no topo da mesma página de configuração.

---

## 📋 Etapas da ferramenta

| Etapa | Nome | Responsável principal | O que faz |
|-------|------|-------------|-----------|
| 1 | Entrevistas | Todos | Captura contexto de negócio, objetivo MTA, mídias, plataformas, conversões e perguntas do cliente. Alimenta todas as etapas seguintes automaticamente. |
| 2 | Mapeamento | DE / BA | Gera checklist de validações, riscos por criticidade e queries BigQuery automaticamente com base na Etapa 1. |
| 3 | Diagnóstico | DE / DS / BA | Define evento de conversão, agrupamento de canais e lookback window. Recomenda modelo de atribuição e gera roteiro de EDA para o DS. |
| 4 | Construção | DE / DS / BA | Tarefas por área geradas automaticamente. Query pré-pronta para construção da base de jornadas. |

> O conteúdo de cada etapa é filtrado pelo **role switcher** (DE / DS / BA) no canto superior direito — cada papel vê apenas o que é relevante para sua atuação.

---

## 🔍 MTA Source Validator

Módulo independente acessível pelo botão no topo da ferramenta, a qualquer momento. Verifica a qualidade das fontes de dados (GA4 e Appsflyer) antes de construir a SOT.

### Fluxo

```
BA preenche a aba Configuracao
         |
Ferramenta gera o CONFIG Python automaticamente
         |
DE copia o CONFIG → cola na Celula 2 do Colab
         |
DE substitui dados mockados pelas queries reais (Celula 3)
         |
DE executa todas as celulas em ordem
         |
Na ultima celula, roda:
  import json
  print(validator.to_dataframe().to_json(orient='records'))
         |
DE copia o JSON → cola na aba Resultados
         |
Ferramenta renderiza o relatorio (PASS / WARN / FAIL)
e propaga alertas para as etapas do planejamento
```

### Dimensões verificadas

| # | Dimensão | Fonte | O que verifica |
|---|----------|-------|----------------|
| 1 | Cobertura Temporal | GA4 + Appsflyer | Dias disponíveis e gaps na série |
| 2 | Cobertura de Identidade | GA4 + Appsflyer | % de user_id preenchido nas conversões |
| 3 | Qualidade de UTMs | GA4 | % de source nulo e canais não mapeados |
| 4 | Duplicatas | GA4 | Taxa de eventos duplicados nas conversões |
| 5 | Volume de Conversões | GA4 | Média de conversões por dia |
| 6 | Consistência entre Fontes | GA4 x Appsflyer | Divergência de volume por dia |

> **Nota:** a conexão com o BigQuery real é feita manualmente pelo DE no Colab. A automação completa via API é uma evolução planejada.

---

## ✨ Lógica de geração automática

A ferramenta gera conteúdo dinamicamente com base nas escolhas da Etapa 1:

- **App + Web selecionados** → validação de User ID cross-platform + query BigQuery + alerta de risco ALTO
- **CRM ativo** → tarefa de cruzamento de User ID + pergunta de EDA sobre papel do CRM na jornada
- **Objetivo: Canibalização** → recomendação de Markov + alerta para DS
- **Objetivo: Awareness** → alerta de lookback estendida + pergunta de EDA sobre canais introdutores
- **Ciclo de decisão** → lookback window sugerida calculada automaticamente
- **MMP ativo (Adjust / Appsflyer)** → checklist de exportação para BigQuery + risco CRITICO se ausente
- **Canal offline** → alerta de identificador necessário para cruzamento
- **Perguntas do cliente** → viram tarefas automáticas do BA na Etapa 4

---

## 🚀 Integração com martech-toolkit

A seção **"Configuração de Ambiente — Pipeline Dataform"** na Etapa 1 conecta o planejamento diretamente ao pipeline de dados da empresa.

Com base nas respostas das entrevistas, a ferramenta gera automaticamente os arquivos do [martech-toolkit](../martech-toolkit-v9-main/) prontos para deploy:

| Arquivo gerado | O que é |
|---|---|
| `workflow_settings.yaml` | Variáveis do Dataform com datasets e parâmetros do cliente |
| `definitions/REFINED/DM_GOLD_MARKETING_PERFORMANCE.sqlx` | Tabela Gold de performance com o **evento de conversão real** do cliente substituído |
| `definitions/REFINED/DM_GOLD_RETAIL_CUBE.sqlx` | Cubo de Retail Media parametrizado |
| `definitions/TRUSTED/FT_SIL_GA4_EVENTS.sqlx` | Tabela Silver de eventos GA4 |
| `definitions/TRUSTED/FT_SIL_ADS_PERFORMANCE.sqlx` | Tabela Silver de performance de Ads |
| `includes/data.js` | Módulo de variáveis com todos os parâmetros do cliente |

### Como usar

1. Preencha a **Etapa 1** (canais, cliente, configuração BigQuery)
2. Na **Etapa 3**, preencha o evento de conversão GA4 (ex: `generate_lead`, `purchase`)
3. Abra o card **"Configuração de Ambiente — Pipeline Dataform"** na Etapa 1
4. Revise os arquivos gerados nos previews
5. Clique em **"Download Pacote ZIP"**
6. Extraia o ZIP na pasta do `martech-toolkit` e execute o `deploy.sh`

> O DE sai da entrevista com o pacote de deploy em mãos, sem precisar editar arquivos manualmente.

---

## 🔧 Referências técnicas

- **Infraestrutura de dados:** Google BigQuery (GA4 export)
- **Modelos suportados:** Markov Chain, Shapley Value
- **Biblioteca de modelagem:** Jatoox
- **Notebook de validação:** MTA_Source_Validator.ipynb (Google Colab)

---

## 💡 Convenções de commit

```
feat: nova funcionalidade
fix: correcao de bug
style: ajuste visual sem mudanca de logica
refactor: refatoracao de codigo
docs: atualizacao de documentacao
```

---

## 👥 Times e responsabilidades

| Role | Cor | Responsabilidade |
|------|-----|-----------------|
| **DE** | Azul | Base de jornada, queries BigQuery, validação de dados, execução do Validador |
| **DS** | Roxo | EDA, execução do modelo (Jatoox), análise de resultados, versionamento de notebooks |
| **BA** | Verde | Configuração do Validador, perguntas de negócio, storytelling, apresentação ao cliente |
