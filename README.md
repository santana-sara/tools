# 📊 MTA Planner — DP6

Ferramenta interativa para planejamento e organização de projetos de **Multi-Touch Attribution (MTA)**, estruturada em 4 etapas com geração automática de validações, queries BigQuery e tarefas por área (DE / DS / BA).

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

O GitHub Pages atualiza automaticamente em ~1–2 minutos após o push.

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

| Etapa | Nome | Responsável | O que faz |
|-------|------|-------------|-----------|
| 1 | Entrevistas | Todos | Captura contexto de negócio, mídias, plataformas e perguntas do cliente |
| 2 | Mapeamento | DE | Gera checklist de validações e riscos automaticamente |
| 3 | Diagnóstico | DE / DS / BA | Agrupamento de canais, lookback window e perguntas da EDA |
| 4 | Construção | DE / DS / BA | Tarefas por área geradas com base em tudo que foi preenchido |

---

## ✨ Lógica de geração automática

A ferramenta gera conteúdo dinamicamente com base nas escolhas da Etapa 1:

- **App + Web selecionados** → validação de User ID cross-platform + query BigQuery
- **CRM ativo** → tarefa de cruzamento de User ID na base
- **Objetivo: Canibalização** → recomendação de Markov + alerta para DS
- **Ciclo de decisão** → lookback window sugerida calculada automaticamente
- **Perguntas do cliente** → viram tarefas automáticas do BA na Etapa 4

---

## 🔧 Referências técnicas

- **Biblioteca de modelos:** [Jatoox](https://github.com/) — criada por Jaimeira
- **Infraestrutura de dados:** Google BigQuery (GA4 export)
- **Modelos suportados:** Markov Chain, Shapley Value
- **Projetos de referência:** OLX (Zap), Banco Inter

---

## 💡 Convenções de commit

```
feat: nova funcionalidade
fix: correção de bug
style: ajuste visual sem mudança de lógica
refactor: refatoração de código
docs: atualização de documentação
```

---

## 👥 Times

| Role | Cor | Responsabilidade |
|------|-----|-----------------|
| **DE** | Azul | Base de jornada, queries BigQuery, validação de dados |
| **DS** | Roxo | EDA, execução do modelo (Jatoox), análise de resultados |
| **BA** | Verde | Perguntas de negócio, storytelling, apresentação ao cliente |
