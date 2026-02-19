# MTA Planner — Próximos Passos e Oportunidades

## Visão geral

Evoluir o MTA Planner de uma ferramenta de checklist interativo para um **assistente inteligente com memória institucional**, alimentado pelos projetos reais do time. A IA usa o histórico de projetos executados como base de conhecimento para gerar diagnósticos, sugestões e melhores práticas contextualizadas para cada novo projeto.

---

## O que temos hoje

- Interface interativa com 4 etapas estruturadas (Entrevistas → Mapeamento → Diagnóstico → Construção)
- Geração automática de validações, riscos e queries BigQuery via lógica condicional
- Visão por role (DE / DS / BA)
- Hospedado via GitHub Pages

---

## Próximos passos

### 1. Construir a base de conhecimento (Knowledge Base)

Reunir e estruturar os projetos de MTA já realizados como fonte de verdade para a IA:

- **Por projeto, capturar:**
  - Prazo real de execução por etapa vs. estimado
  - Ferramentas utilizadas (BigQuery, GA4, MMPs, etc.)
  - Configuração de canais e agrupamentos adotados
  - Lookback window escolhida e justificativa
  - Modelo selecionado (Markov, Shapley) e por quê
  - Principais dificuldades e como foram resolvidas
  - Pontos cegos encontrados e soluções
  - Lições aprendidas e o que faria diferente
- **Padrões por indústria:** financeiro, e-commerce, portais, SaaS — cada vertical tem comportamentos recorrentes que podem virar sugestões automáticas

---

### 2. Explorar Vertex AI (Google Cloud) como plataforma de IA

Dado que a empresa é parceira Google, vale avaliar o ecossistema antes de escolher a stack:

- **Vertex AI Search & Conversation:** ideal para RAG — indexa os documentos dos projetos e responde perguntas com base neles
- **Vertex AI Agent Builder:** cria agentes conversacionais com grounding nos documentos da knowledge base
- **Gemini na Vertex AI:** modelo de linguagem com janela de contexto grande, bom para processar documentação extensa de projetos
- **Document AI:** para processar automaticamente PDFs, apresentações e notebooks dos projetos anteriores
- **BigQuery ML:** possibilidade de cruzar dados reais de execução dos projetos com previsões (prazo, complexidade, risco)
- **Vantagem parceria:** créditos Google Cloud, suporte técnico e acesso antecipado a features podem acelerar o desenvolvimento

> **Alternativa paralela a avaliar:** API do Claude (Anthropic) — mais simples para um MVP rápido, pode ser substituída ou combinada com Vertex AI depois

---

### 3. Arquitetura proposta — RAG com memória institucional

```
Documentos dos projetos anteriores
(PDFs, notebooks, apresentações, planilhas)
              ↓
     Processamento e indexação
     (Vertex AI Search ou embeddings)
              ↓
         Knowledge Base
    "memória do time de MTA"
              ↓
    Inputs do MTA Planner (Etapa 1)
    (cliente, canais, objetivo, plataformas...)
              ↓
         Agente / LLM
    raciocina sobre o novo projeto
    consultando a knowledge base
              ↓
     Saídas enriquecidas:
     - Diagnóstico narrativo
     - Alertas contextualizados
     - Lookback recomendada com base em projetos similares
     - Queries BigQuery customizadas
     - Estimativa de prazo por etapa
     - Riscos com histórico de ocorrência
     - Melhores práticas por vertical/indústria
```

---

### 4. Funcionalidades que a IA desbloquearia no MTA Planner

- **Diagnóstico narrativo ao final da Etapa 1:** parágrafo em linguagem natural explicando os riscos, complexidade e recomendações do projeto
- **Queries BigQuery customizadas:** geradas com os nomes reais das tabelas, datasets e campos do cliente
- **Estimativa de prazo:** "projetos similares ao deste cliente levaram em média X semanas na Etapa 2"
- **Sugestão de lookback baseada em histórico:** "para financeiro com ciclo médio, a lookback de 30 dias foi validada em 3 dos 4 projetos anteriores"
- **Alertas de risco com precedente:** "esse padrão de App sem MMP gerou retrabalho no projeto OLX — ação recomendada: ..."
- **Perguntas de EDA personalizadas:** geradas com base no objetivo + vertical + canais ativos
- **Documento de planejamento automático:** exportar o `.docx` preenchido com todas as decisões do projeto ao final da Etapa 3

---

### 5. Estrutura de dados para alimentar a IA

Para cada projeto histórico, criar um documento padronizado com:

```
projeto: [nome do cliente]
vertical: [financeiro | e-commerce | portal | etc]
periodo_execucao: [data início → data fim]
prazo_por_etapa:
  entrevistas: X dias
  mapeamento: X dias
  diagnostico: X dias
  construcao: X dias

configuracao:
  plataformas: [web | app | físico]
  canais: [lista]
  modelo: [Markov | Shapley | ambos]
  lookback_window: X dias
  conversao: [evento]
  mmp: [sim/não | qual]

dificuldades:
  - [descrição + como foi resolvido]

licoes_aprendidas:
  - [o que faria diferente]

resultado:
  - [principais insights entregues ao cliente]
```

---

### 6. MVP sugerido — o que implementar primeiro

Antes de Vertex AI, validar o conceito com menor custo:

1. **Estruturar 3–5 projetos históricos** no formato padronizado acima
2. **Implementar o botão "Analisar com IA"** no MTA Planner atual usando API do Claude com os projetos como contexto
3. **Validar com o time** se as sugestões geradas são melhores do que a lógica condicional atual
4. **Se validado:** migrar para Vertex AI com RAG completo e indexação de toda a documentação

---

### 7. Outras oportunidades de enriquecimento do MTA Planner

- **Exportação automática:** gerar o `.docx` de planejamento preenchido ao clicar em "Concluir Etapa 3"
- **Histórico de projetos:** salvar projetos anteriores na ferramenta para consulta e comparação
- **Colaboração em tempo real:** múltiplos usuários (DE, DS, BA) editando simultaneamente — Firestore (Google Cloud) viabiliza isso
- **Integração com Jira / Notion:** exportar as tarefas da Etapa 4 diretamente para onde o time já gerencia o trabalho
- **Dashboard de projetos:** visão consolidada de todos os projetos MTA em andamento e concluídos
- **Templates por vertical:** configurações pré-definidas para financeiro, e-commerce, portais — acelera a Etapa 1

---

## Resumo de decisões a tomar

| Decisão | Opções | Recomendação inicial |
|---------|--------|---------------------|
| Plataforma de IA | Vertex AI vs. Claude API | Claude API para MVP → Vertex AI para escala |
| Onde hospedar | GitHub Pages vs. Google Cloud Run | GitHub Pages por agora → Cloud Run quando tiver backend |
| Por onde começar | Interface vs. Knowledge Base | Montar a knowledge base primeiro — a IA só é boa com dados bons |
| Quem lidera | DE, DS ou BA | DE (Sara) para infra de dados + DS para integração com modelos |