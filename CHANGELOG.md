# MTA Planner — Changelog de Melhorias

> Documento gerado em 2026-03-27. Descreve todas as evoluções implementadas na ferramenta a partir da versão original.

---

## Visão geral das mudanças

O `index.html` passou por duas rodadas de melhorias organizadas em:

- **Camada 1A** — Lógica condicional enriquecida (sem dependência de infra nova)
- **Camada 1B** — Refinamentos de usabilidade e produtividade
- **Integração Jatoolbox** — Scripts Python atualizados para usar a biblioteca oficial da DP6

---

## Camada 1A — Lógica condicional enriquecida

### 1. Perfil de Complexidade (Etapa 1 — sidebar)

**O que é:** Card novo na sidebar da Etapa 1 que calcula e exibe automaticamente o nível de complexidade do projeto (BAIXA / MÉDIA / ALTA) com base nos campos preenchidos.

**Como funciona:** A função `buildDiagnosis()` é chamada a cada `onInput()`. Ela atribui pontos a cada fator de risco e soma o score total:

| Fator | Pontos | Severidade |
|---|---|---|
| BigQuery não configurado | +3 | 🔴 Alta |
| Web + App simultâneos | +3 | 🔴 Alta |
| App sem MMP | +2 | 🔴 Alta |
| Canal offline / físico | +3 | 🔴 Alta |
| Ambiente não logado | +2 | 🔴 Alta |
| CRM ativo | +2 | 🟡 Média |
| Objetivo omnichannel | +2 | 🔴 Alta |
| Ciclo longo (60–90d) | +1 | 🟡 Média |
| MMP configurado | +1 | 🟢 Baixa |
| 6+ canais ativos | +1 | 🟢 Baixa |
| Marca consolidada | +1 | 🟢 Baixa |
| Objetivo awareness | +1 | 🟢 Baixa |

**Resultado:** Score ≥ 8 = ALTA, Score ≥ 4 = MÉDIA, Score < 4 = BAIXA.

**Helper adicionado:**
```javascript
function getProjectParams() {
  // Lê projeto GCP, dataset GA4/AF e lookback do Validador
  // Usa como fallback: 'SEU_PROJECT', 'analytics_XXXXXXX', 'appsflyer_data'
}
```

---

### 2. Checklist expandido — Etapa 2

**O que era:** 9 itens fixos de validação.

**O que ficou:** 17 itens, sendo a maioria gerada condicionalmente com base na Etapa 1.

**Itens adicionados:**
- BigQuery não configurado → item CRÍTICO aparece no topo quando `chips-bq` = "Não"
- Validação prática de User ID cross-platform (não só checklist teórico)
- App sem MMP → item crítico específico
- Mapeamento de schema do MMP (`advertising_id`, `customer_user_id`)
- Schema de eventos CRM documentado
- Ambiente não logado → item sobre fallback para `user_pseudo_id`
- Confirmação de nomes exatos dos eventos de conversão no BigQuery
- Ausência de mudanças de implementação GA4 no período confirmada

**Exemplo de item condicional:**
```javascript
!loginOk ? {
  t: '(!) Ambiente não logado — user_id limitado, avaliar fallback (user_pseudo_id)',
  auto: true,
  reason: 'Sem login, rastreamento cross-session é impreciso'
} : null
```

---

### 3. Queries BigQuery parametrizadas — Etapa 2 (sidebar DE)

**O que era:** 2 queries fixas com `proj.dataset` genérico.

**O que ficou:** 6–7 queries (variam por configuração), todas com:
- Label de **pilar** (Identidade, Volume, Qualidade, Cobertura)
- Parâmetros preenchidos automaticamente a partir dos campos do MTA Source Validator (`vc-project`, `vc-ga4`, `vc-af`)
- Botão `📋 Copiar` funcional em todas

**Queries adicionadas:**

| Query | Pilar | Condição de aparição |
|---|---|---|
| Cobertura de User ID nas conversões | Identidade | Sempre |
| Volume diário de conversões | Volume | Sempre |
| Deduplicação de eventos de conversão | Qualidade | Sempre |
| Distribuição de source / medium | Cobertura | Sempre |
| Validação User ID cross-platform | Identidade | Web + App |
| Taxa de match MMP ↔ GA4 | Identidade | MMP ativo |
| Taxa de match CRM ↔ GA4 | Identidade | CRM ativo |

**Exemplo de parametrização:**
```javascript
// Lê do Validador; usa placeholder se não preenchido
const p = getProjectParams();
const tbl = `\`${p.proj}.${p.ga4}.events_*\``;
// Resultado: `meu-projeto.analytics_123456789.events_*`
```

---

### 4. Banco de perguntas EDA por pilar — Etapa 3

**O que era:** ~10 perguntas sem organização.

**O que ficou:** 22–26 perguntas (variam por configuração), organizadas em 6 pilares com separadores visuais:

| Pilar | Perguntas fixas | Perguntas condicionais |
|---|---|---|
| 🪪 Identidade | 2 | +1 Web+App, +1 CRM, +1 MMP |
| 📡 Cobertura | 3 | +1 Offline, +1 Afiliados |
| ✅ Qualidade | 3 | +1 CRM |
| ⏱ Janela | 3 | +1 Awareness |
| 🎯 Conversão | 3 | — |
| 🤖 Atribuição | 4 | +1 Sobreposição |

**Exemplo de pergunta condicional:**
```javascript
if (hasCRM) edaQ.push({
  pilar: '🪪 Identidade',
  t: 'Qual % dos disparos de CRM tem match com sessões no GA4 no mesmo user_id? Há muitos disparos sem match?',
  auto: true,
  role: 'ds',
  reason: 'CRM ativo — validar cruzamento'
});
```

---

### 5. Scripts Python para DS — Etapa 3 (sidebar DS)

**O que era:** Não existia.

**O que ficou:** Painel novo visível no role DS com scripts prontos para copiar no Google Colab, integrados com a biblioteca **Jatoolbox** da DP6.

> Ver seção **Integração Jatoolbox** para detalhes completos.

---

## Camada 1B — Refinamentos de usabilidade

### 6. Templates por vertical — Etapa 1

**O que é:** Card novo no topo da Etapa 1 com 5 botões de atalho que pré-preenchem toda a configuração com um clique.

**Templates disponíveis:**

| Template | Ciclo | Plataforma | Canais principais | MMP |
|---|---|---|---|---|
| 🏦 Financeiro | Longo | Web + App | Google, Meta, CRM | AppsFlyer |
| 🛒 E-commerce | Curto | Web | Google, Meta, Afiliados, CRM | Não usa |
| 📱 App-only | Curto | App | Google, Meta | AppsFlyer |
| 📰 Portal / Publisher | Médio | Web | Google, Display, YouTube, CRM | Não usa |
| 🏢 SaaS / B2B | Muito longo | Web | Google, CRM | Não usa |

**Como funciona:**
```javascript
function applyTemplate('financeiro') {
  // 1. Seta f-ciclo e f-marca via .value
  // 2. Liga/desliga chips de cada grupo por texto exato
  // 3. Marca o botão como tpl-active visualmente
  // 4. Chama onInput() para regenerar tudo
}
```

**Comportamento importante:** O nome do cliente (`f-cliente`) **não é alterado** ao aplicar um template.

---

### 7. Persistência local com localStorage

**O que é:** O estado completo do formulário é salvo automaticamente no `localStorage` do navegador a cada interação. Ao recarregar a página, tudo é restaurado.

**O que é salvo:**

```javascript
{
  cliente, produto, ciclo, marca,     // campos de texto
  obj, conv, canal, plat,             // chips on/off
  login, mmp, bq,                     // chips exclusivos
  convCust, canalCust,                // chips customizados adicionados pelo usuário
  questions,                          // perguntas do cliente (Etapa 1)
  vcProject, vcGa4, vcAf,            // config do MTA Source Validator
  activeTpl                           // template ativo (para restaurar destaque visual)
}
```

**Funções implementadas:**
- `saveState()` — chamada automaticamente dentro de `onInput()`
- `restoreState()` — chamada uma vez no carregamento da página
- `addChipRaw(val, chipsId)` — helper para recriar chips customizados na restauração

**Chave usada:** `mta_planner_v2`

---

### 8. Exportação do planejamento (.md)

**O que é:** Botão `📄 Exportar .md` na barra de ações da Etapa 1. Gera e faz download de um arquivo Markdown com todo o planejamento do projeto.

**Conteúdo do arquivo gerado:**

```
# MTA Planner — [Nome do cliente]
> Exportado em DD/MM/AAAA via MTA Planner DP6

## Etapa 1 — Entrevistas
| Campo | Valor |
| Cliente | ... |
| Objetivos | ... |
...

### Perguntas do cliente
1. ...

## Etapa 2 — Mapeamento
### Validações de maturidade de dados
- [x] Item concluído
- [ ] Item pendente

## Etapa 3 — Diagnóstico
| Evento de conversão | ... |
### Agrupamento de canais
### Perguntas da EDA (com status de conclusão)
### Modelo selecionado

## Etapa 4 — Construção
### DE — Base de Jornada
### DS — EDA e Modelo
### BA — Insights e Apresentação

## Riscos identificados
- Risco 1
```

**Nome do arquivo:** `MTA_[Cliente]_[YYYY-MM-DD].md`

---

### 9. Query SOT completa e parametrizada — Etapa 4 (sidebar DE)

**O que era:** Uma query estática e simplificada de montagem de jornadas.

**O que ficou:** Query completa de 6 CTEs, gerada dinamicamente com base na configuração das Etapas 1–3, pronta para rodar diretamente no BigQuery.

**Parâmetros preenchidos automaticamente:**
- Projeto GCP e dataset GA4 (do MTA Source Validator)
- Evento de conversão (do campo `f-conv-evt` da Etapa 3)
- Lookback window calculada da Etapa 1
- CASE WHEN de canais gerado a partir dos canais ativos selecionados

**Estrutura da query (6 CTEs):**

```sql
WITH
-- 1. eventos       → todos os eventos com canal agrupado (CASE WHEN dinâmico)
-- 2. conversoes    → 1ª conversão por usuário no período
-- 3. touchpoints   → eventos dentro da lookback window antes da conversão
-- 4. jornadas_conv → STRING_AGG por usuário → jornadas conversoras
-- 5. jornadas_nao_conv → mesma estrutura para não conversores
-- 6. SELECT final  → UNION ALL com ordenação

SELECT * FROM jornadas_conv
UNION ALL
SELECT * FROM jornadas_nao_conv WHERE n_touchpoints > 0
ORDER BY converteu DESC, dt_conversao DESC
```

**Colunas do resultado:**

| Coluna | Tipo | Descrição |
|---|---|---|
| `uid` | STRING | COALESCE(user_id, user_pseudo_id) |
| `jornada` | STRING | "Canal1 > Canal2 > Canal3" |
| `n_touchpoints` | INT64 | Número de touchpoints na jornada |
| `dt_primeiro_tp` | DATE | Data do primeiro touchpoint |
| `dt_conversao` | DATE | Data da conversão (NULL se não converteu) |
| `duracao_dias` | INT64 | Dias entre primeiro TP e conversão |
| `converteu` | INT64 | 1 = converteu, 0 = não converteu |

---

## Integração Jatoolbox

**Biblioteca:** [`Jatoolbox`](https://dp6.github.io/customer-journey-analysis-toolbox/) — lib Python da DP6 para análise de jornadas no formato `"A > B > C"`.

**Instalação:**
```bash
pip install Jatoolbox ChannelAttribution pandas
```

**Inicialização:**
```python
from jatoolbox import jatoolbox as jt
an = jt.JAToolbox()
```

### Scripts disponíveis no painel DS (Etapa 3)

Os scripts aparecem no role **DS**, na sidebar da Etapa 3, após clicar em "Avançar para Diagnóstico". Alguns são condicionais.

---

#### Script 0 — Setup e estrutura base
> Sempre disponível

Instala dependências, inicializa o `JAToolbox`, adiciona colunas derivadas ao DataFrame e prepara `df_grp` (todas as jornadas agrupadas) e `df_conv` (apenas conversoras) — estrutura exigida pelos métodos `channels_by_tp` e `tps_by_channel`.

```python
from jatoolbox import jatoolbox as jt
an = jt.JAToolbox()

df['n_tps']      = df['jornada'].apply(an.get_size)
df['primeiro_tp'] = df['jornada'].apply(an.get_first_tp)
df['ultimo_tp']   = df['jornada'].apply(an.get_last_tp)

df_grp = (df.groupby('jornada')
            .agg(ocurrencies=('uid','count'), conversoes=('converteu','sum'))
            .reset_index())
df_conv = df_grp[df_grp['conversoes'] > 0].copy()
```

**Métodos Jatoolbox:** `get_size`, `get_first_tp`, `get_last_tp`

---

#### Script 1 — Distribuição de touchpoints
> Sempre disponível

Quantos touchpoints têm as jornadas conversoras? Qual a taxa de conversão por tamanho de jornada? Quais são os canais mais frequentes como primeiro e último touchpoint?

```python
dist = df.groupby('n_tps')['converteu'].agg(total='count', conversoes='sum').reset_index()
dist['tx_conv'] = (dist['conversoes'] / dist['total'] * 100).round(1)

print(df[df['converteu']==1]['primeiro_tp'].value_counts().head(8))  # Introdutores
print(df[df['converteu']==1]['ultimo_tp'].value_counts().head(8))    # Conversores
```

**Métodos Jatoolbox:** usa colunas calculadas no Setup

---

#### Script 2 — Canal por posição na jornada
> Sempre disponível

Usa `channels_by_tp` para mostrar qual canal aparece em cada posição da jornada (1º touchpoint = Introdutor, últimas posições = Conversor). Usa `tps_by_channel` para frequência geral.

```python
result = an.channels_by_tp(
    df=df_conv, j='jornada',
    ocurrencies='ocurrencies',
    max_journey_size=6, separator=' > '
)

freq = an.tps_by_channel(df=df_conv, j='jornada', ocurrencies='ocurrencies')
```

**Métodos Jatoolbox:** `channels_by_tp`, `tps_by_channel`

---

#### Script 3 — Matriz de transição entre canais
> Sempre disponível

Usa `get_transitions` em cada jornada e agrega com `Counter` para montar a matriz completa de transições entre canais. Mostra os pares mais frequentes.

```python
transitions = Counter()
for jornada in df[df['converteu']==1]['jornada']:
    trans = an.get_transitions(jornada, count=True, norm=False)
    if hasattr(trans, 'items'):
        for k, v in trans.items():
            transitions[k] += v

matrix = trans_df.pivot(index='origem', columns='destino', values='freq').fillna(0)
```

**Métodos Jatoolbox:** `get_transitions`

---

#### Script 4 — Calibragem da lookback window
> Sempre disponível | lookback pré-preenchida com o valor calculado na Etapa 1

Valida empiricamente se a lookback window sugerida é adequada. Meta: > 90% das jornadas conversoras devem se encerrar dentro da janela.

```python
LW = 30  # preenchido automaticamente com o valor da Etapa 1

for p in [50, 75, 90, 95, 99]:
    print(f"P{p:02d}: {conv.quantile(p/100):.0f} dias")

pct = (conv <= LW).mean()
print(f"% jornadas encerradas em <= {LW} dias: {pct:.1%}")
```

Também demonstra uso de `get_nth_tp` para inspecionar jornadas com 3+ touchpoints.

**Métodos Jatoolbox:** `get_nth_tp`, `get_last_tp`

---

#### Script 5 — Last Click vs. Markov (ChannelAttribution)
> Sempre disponível

Executa o modelo Markov via `ChannelAttribution`, compara com Last Click e exibe o efeito de remoção por canal.

```python
from ChannelAttribution import markov_model

markov_out, removal_eff, _ = markov_model(
    df, var_path='jornada', var_conv='converteu',
    var_value=None, order=1, out_more=True
)

# Last Click usa coluna 'ultimo_tp' calculada no Setup
lc = df[df['converteu']==1]['ultimo_tp'].value_counts().rename('last_click')

result['delta_pct'] = ((result['markov'] - result['last_click'])
                       / result['last_click'].clip(1) * 100).round(1)
```

---

#### Script 6 — Análise cross-platform web vs. app
> Condicional: aparece apenas quando **Web + App** estão ativos na Etapa 1

Usa `translate_tp` para mapear cada canal para sua plataforma (web / app / ambos), depois identifica jornadas que cruzaram as duas plataformas.

```python
plat_dict = {
    'Mídia Performance': 'web', 'Orgânico': 'web',
    'CRM': 'web',               'Direto': 'web',
    'Mídia Social Paga': 'ambos',
}

df['jornada_plat'] = df['jornada'].apply(
    lambda j: an.translate_tp(j, translation_dict=plat_dict, separator=' > ')
)
```

**Métodos Jatoolbox:** `translate_tp`

---

#### Script 7 — Análise de sobreposição de audiência
> Condicional: aparece apenas quando o objetivo **"Há sobreposição de audiência entre canais?"** está selecionado

Usa `check_tp` para identificar se brand search aparece em cada jornada e com que frequência é o last touch.

```python
df['tem_brand'] = df['jornada'].apply(
    lambda j: an.check_tp(j, tp_to_check='Mídia Performance', separator=' > ')
)
df['brand_last'] = df['ultimo_tp'] == 'Mídia Performance'
```

**Métodos Jatoolbox:** `check_tp`

---

## Resumo de funções JavaScript adicionadas

| Função | Onde é chamada | O que faz |
|---|---|---|
| `getProjectParams()` | `buildStage2()`, `buildSotQuery()` | Lê projeto/dataset do Validador, retorna com fallbacks |
| `buildDiagnosis()` | `onInput()` | Gera perfil de complexidade na sidebar da Etapa 1 |
| `applyTemplate(key)` | Botões da Etapa 1 | Pré-preenche formulário por vertical |
| `saveState()` | `onInput()` | Persiste estado no localStorage |
| `restoreState()` | Inicialização da página | Restaura estado do localStorage |
| `addChipRaw(val, id)` | `restoreState()` | Recria chips customizados |
| `exportPlan()` | Botão Etapa 1 | Gera e baixa arquivo .md do planejamento |
| `buildSotQuery()` | `buildStage4()` | Gera query SOT parametrizada no sidebar DE da Etapa 4 |
| `buildPythonScripts()` | `buildStage3()` | Gera scripts Jatoolbox no sidebar DS da Etapa 3 |
| `copyQuery(btn)` | Botões 📋 Copiar | Copia código limpo (sem HTML) para clipboard |

---

## Fluxo completo atualizado

```
[Etapa 1] Entrevistas
  ├── Templates por vertical (atalho de configuração)
  ├── Campos de contexto (cliente, produto, ciclo, marca)
  ├── Chips de objetivos, canais, plataformas, conversões, MMP, BQ
  ├── Perguntas do cliente (BA)
  ├── Sidebar: Lookback sugerida + mini cards de resumo
  ├── Sidebar: Perfil de complexidade (AUTO) ← NOVO
  ├── Sidebar: Alertas automáticos
  ├── Botão: 💾 Salvar no Sheets
  └── Botão: 📄 Exportar .md ← NOVO

[Etapa 2] Mapeamento (gerado automaticamente da Etapa 1)
  ├── Checklist de validações (17 itens, condicionais) ← EXPANDIDO
  ├── Riscos identificados com ação recomendada ← EXPANDIDO
  └── Sidebar DE: Queries BigQuery parametrizadas (6–7 queries) ← EXPANDIDO

[Etapa 3] Diagnóstico (gerado automaticamente da Etapa 1)
  ├── Definição de conversão
  ├── Agrupamento de canais (CASE WHEN dinâmico)
  ├── Lookback window e períodos
  ├── Perguntas da EDA (22–26, organizadas por pilar) ← EXPANDIDO
  ├── Modelo de atribuição com recomendação
  └── Sidebar DS: Scripts Python com Jatoolbox (5–7 scripts) ← NOVO

[Etapa 4] Construção (gerado automaticamente das Etapas 1–3)
  ├── Tarefas DE, DS e BA com badge AUTO e CRÍTICO
  ├── Barra de progresso
  └── Sidebar DE: Query SOT completa (6 CTEs, parametrizada) ← NOVO

[MTA Source Validator] (acessível em qualquer etapa)
  ├── Aba 1: Configuração (BA) — pré-preenchida da Etapa 1
  ├── Aba 2: Colab (DE) — CONFIG Python gerado automaticamente
  └── Aba 3: Resultados — importação de JSON do Colab
```

---

## Dependências externas

| Biblioteca | Instalação | Onde é usada |
|---|---|---|
| [Jatoolbox](https://dp6.github.io/customer-journey-analysis-toolbox/) | `pip install Jatoolbox` | Scripts EDA — DS (Etapa 3) |
| [ChannelAttribution](https://cran.r-project.org/web/packages/ChannelAttribution/) | `pip install ChannelAttribution` | Modelo Markov (Script 5) |
| pandas | `pip install pandas` | Todos os scripts Python |
| Google Fonts | CDN (Plus Jakarta Sans, JetBrains Mono) | Interface da ferramenta |

---

*Documento mantido pela equipe de MTA — DP6*
