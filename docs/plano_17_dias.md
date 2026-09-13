# Plano de 17 dias — Projeto Pessoal Um (v1)

Criado em 13/09/2026 (domingo); reformulado no mesmo dia para arquitetura centrada no Databricks.
Início: **seg 14/09/2026**. Entrega: **qua 30/09/2026**. ~44h.
O plano estendido de 12 semanas é o backlog da v2 (`docs/backlog_v2_plano_12_semanas.md`).

## O caso de negócio (fictício)
**Horizonte Investimentos** é uma assessoria que atende clientes de quatro perfis — conservador, moderado, arrojado e agressivo. A diretoria quer parar de recomendar "no feeling" e pediu à área de dados um painel que responda, com dados públicos e histórico desde 2015:

| # | Pergunta de negócio | Onde é respondida |
|---|---|---|
| 1 | Quanto rendeu cada classe nos últimos 12, 36 e 60 meses — nominal e acima da inflação? | SQL (agregação + join com IPCA) · DAX (`CALCULATE` + `DATESINPERIOD`) |
| 2 | R$ 10 mil aplicados em jan/2015 em cada perfil viraram quanto hoje? | SQL (acumulado com window) · DAX (acumulado + parâmetro what-if) |
| 3 | Qual foi o pior momento (drawdown máximo) de cada classe e quantos meses levou para recuperar? | SQL (`MAX OVER`, gaps) · DAX (variáveis) |
| 4 | Em quais anos a renda variável perdeu do CDI? Ranking anual das classes. | SQL (`RANK OVER PARTITION BY ano`, pivot) · DAX (`RANKX`) |
| 5 | Diversificar ajuda? Quando o Ibovespa cai, o dólar (IVVB11) protege? | Python (correlação) · SQL (`CORR`) |
| 6 | Cripto compensou o risco? Retorno por unidade de volatilidade vs CDI. | SQL (`STDDEV OVER`) · DAX (`DIVIDE`) |
| 7 | Qual a expectativa de Selic e IPCA nos próximos 6 meses, e o que isso muda para o cliente conservador? | Python (Prophet) · Power BI (previsto × realizado) |
| 8 | Qual perfil teve o melhor retorno ajustado ao risco, e o que recomendar para cada cliente? | Síntese — página 1 do dashboard + README |

Cada pergunta tem uma resposta escrita em `docs/respostas_negocio.md` (o "relatório para a diretoria") — é isso que vai virar o texto do README e do post.

## Arquitetura
```
┌──────────────────────────── Databricks Free Edition ─────────────────────────────┐
│  APIs públicas ─► notebooks Python ─► bronze (Delta) ─► SQL ─► silver ─► gold      │
│  BCB SGS, yfinance   requests/pandas    dados brutos    tratamento   tabelas fato/dim│
│                                                                          │         │
│                      notebooks Python (pandas/Prophet) ◄─────────────────┤         │
│                      EDA, baselines, previsão ─► gold.fato_previsao ─────┘         │
│  Git folder ◄──────────────────────────────────────────────────── commits ─────────┼─► GitHub
└───────────────────────────────────────────┬───────────────────────────────────────┘
                                            │ conector Databricks (ou CSV via Volume)
                                            ▼
                     Power BI Desktop: modelo estrela + DAX básico → avançado + KPIs
```
Ambiente local (VS Code + `.venv`) fica como apoio: clone do repositório, Git pela linha de comando, README, e backup se o Databricks estiver fora.

## Escada de risco (v1)
| Perfil | Ativo no projeto | Série |
|---|---|---|
| Conservador | CDB pós-fixado / Tesouro Selic | CDI (SGS 12), Selic (SGS 11) |
| Moderado | Fundo imobiliário | HGLG11.SA |
| Arrojado | Ibovespa e S&P 500 em reais | ^BVSP, BOVA11.SA, IVVB11.SA |
| Agressivo | Bitcoin | BTC-USD × PTAX (SGS 1) |
| Referência | Inflação | IPCA (SGS 433) |

## Orçamento de tempo (~44h)
| Período | Horas | Uso |
|---|---|---|
| Seg–qui (14–17/09 e 21–24/09) | 1h manhã + 1h noite | manhã = prática; noite = teoria curta (≤40 min) + diário (10 min) |
| Sex 18 e 25/09 | 2–3h | bloco de prática |
| Sáb 19 e 26/09 | 5h | bloco profundo — a maior entrega da semana |
| Dom 20 e 27/09 | 3–4h | fechar, commitar, revisar |
| Seg–qua 28–30/09 | 2h/dia | acabamento e publicação |

Regras: **manhã nunca é teoria; noite nunca é código pesado.** Todo dia termina com um commit, mesmo pequeno. Travou por mais de 30 min → anota no diário e me chama.

## Dia a dia

### Semana 1 — Ingestão, tratamento e perguntas em SQL
| Dia | Manhã (prática, 1h) | Noite (teoria, ≤40 min) |
|---|---|---|
| **Seg 14** | Ligar o Databricks ao GitHub (Git folder apontando para o repositório). O que é uma API (10 min). Notebook `notebooks/01_ingestao_bcb` (Python): `requests.get` na Selic → pandas → `spark.createDataFrame` → tabela `bronze.sgs`. | **Git 1 (Xperiun · Git & GitHub para Colaboração):** "Git & GitHub" (21 min) + "Meu Primeiro Commit" (17 min). Commit e push do dia com as próprias mãos — pelo Git folder do Databricks. |
| **Ter 15** | Generalizar em `busca_sgs(codigo, inicio, fim)`; ingerir CDI, IPCA, PTAX; gravar em `bronze.sgs` com coluna `codigo`. | **Git 1b + Python:** Xperiun Git — "Conectando meu repositório local ao GitHub", "Resumo dos novos comandos" (23 min). Xperiun Python — "Funções" (12 min). |
| **Qua 16** | Notebook `02_ingestao_yfinance` (`%pip install yfinance`): ^BVSP, BOVA11, IVVB11, HGLG11, BTC-USD desde 2015 → `bronze.cotacoes`. | **Pandas:** Kaggle *Pandas* lições 1–3. 15 min: pós × pré × inflação × variável. |
| **Qui 17** | Notebook `03_silver` (**SQL básico → intermediário**): `CAST`, `to_date`, `date_trunc`, `last_day`, `GROUP BY`, `CASE WHEN`, `JOIN`; tudo mensal; `silver.serie_mensal` e `silver.dim_ativo`. | **Git 2 (Xperiun):** "Introdução a branches", "Conceitos essenciais", "Pull Request" (41 min). Primeira PR do projeto (branch `silver`) e merge. |
| **Sex 18** (2–3h) | Antes (40 min): Xperiun · Databricks — "Databricks SQL" aulas 1–4, "Delta Lake I". Depois, notebook `04_gold` (**SQL avançado**): CTEs, `LAG`, retorno acumulado com `EXP(SUM(LN(1+r)) OVER (...))`, `STDDEV OVER (ROWS BETWEEN 11 PRECEDING AND CURRENT ROW)`, drawdown com `MAX OVER`, `QUALIFY`. Tabelas `gold.fato_mensal`, `gold.dim_ativo`, `gold.dim_calendario`. **Testar o conector do Power BI Desktop** (Xperiun · "Conectando os seus dados no Power BI"). | — |
| **Sáb 19** (5h) | Notebook `05_perguntas_negocio` (SQL): responder as perguntas 1, 2, 3, 4 e 6 com uma query cada (`RANK`, `PIVOT`, `CORR`, `FIRST_VALUE`). Escrever as respostas em `docs/respostas_negocio.md`. Apoio: Xperiun · Databricks — "Manipulando datas", "Window functions". | — |
| **Dom 20** (3–4h) | Notebook `06_eda` (Python/pandas): R$ 10 mil por perfil, correlação (pergunta 5), gráficos por classe, tabela risco × retorno. Fechar a semana: PR + merge, diário. Apoio: Xperiun · Intro à ML — módulo "Análise Exploratória" (Matplotlib, Seaborn). | — |

### Semana 2 — Previsão e Power BI
| Dia | Manhã (prática, 1h) | Noite (teoria, ≤40 min) |
|---|---|---|
| **Seg 21** | Notebook `07_baselines`: naive e média móvel 12m para Selic, IPCA e Ibovespa mensais; holdout = últimos 12 meses; MAE e MAPE. | **Séries 1:** Kaggle *Time Series* lições 1–2. Xperiun · Intro à ML — "Data Leakage" (nunca embaralhar o tempo). |
| **Ter 22** | Notebook `08_prophet` (`%pip install prophet`): mesmas 3 séries, comparar com baseline. Se o Prophet não bater o naive no Ibovespa, isso **é** o resultado. | **Séries 2:** quickstart do Prophet; intervalo de confiança. |
| **Qua 23** | Previsão 6 meses com intervalo → `gold.fato_previsao` (data, ativo, modelo, previsto, inf, sup). Pergunta 7 em `docs/respostas_negocio.md` + `docs/resultados_modelo.md`. | **Power BI 1 (Xperiun · Modelagem de Dados Avançado):** "Star Schema ou Snowflake?", tabela calendário, relacionamentos. |
| **Qui 24** | Power BI: conectar ao Databricks (ou importar CSVs); modelo estrela `dim_calendario` ← `fato_mensal` → `dim_ativo`, `fato_previsao`. **DAX básico:** medidas de `SUM`/`AVERAGE`, `DIVIDE`, formatação. | **Power BI 2 (Xperiun · DAX Avançado):** "Revisando Contexto de Filtro", "Filtrando com a CALCULATE", "Removendo filtros com ALL" (32 min). |
| **Sex 25** (2–3h) | Antes (25 min): Xperiun · DAX — "Acumulado infinito com ALL/ALLSELECTED", "Médias móveis com DATESINPERIOD". Depois, **DAX intermediário → avançado:** retorno 12/36/60m (`DATESINPERIOD`), acumulado (`PRODUCTX`), retorno real, volatilidade 12m, drawdown (variáveis), `RANKX` anual. | — |
| **Sáb 26** (5h) | Páginas: **(1)** Diretoria — KPIs por perfil, R$ 10 mil com parâmetro what-if, ranking; **(2)** Risco — dispersão vol × retorno, drawdown e recuperação; **(3)** Previsão — previsto × realizado no holdout, projeção 6 meses com faixa. Apoio: Xperiun · DAX — "Usando DAX em Visuais". | — |
| **Dom 27** (3–4h) | Acabamento (tema, títulos, tooltips, bookmarks), capturas para o README, `dashboard/horizonte_investimentos.pbix` commitado. Pergunta 8 respondida. | — |

### Reta final — Portfólio
| Dia | 2h |
|---|---|
| **Seg 28** | README final: o caso Horizonte → dados → arquitetura → perguntas e respostas → modelo → dashboard, com diagrama e prints. Limpar notebooks. |
| **Ter 29** | Rodar o pipeline do zero no Databricks (01 → 08) para provar reprodutibilidade. `docs/aprendizados.md`. Rascunho do post. |
| **Qua 30** | Tag `v1.0` (release). Post no LinkedIn com o link. Projeto na seção *Projetos* do perfil. **Entregue.** |

## Escada de SQL — do básico ao avançado, na ordem em que aparece
| Nível | Funções / recursos | Notebook |
|---|---|---|
| Básico | `SELECT`, `WHERE`, `ORDER BY`, `LIMIT`, `DISTINCT`, `CAST`, `COALESCE` | 03_silver |
| Datas | `to_date`, `date_trunc`, `last_day`, `year`, `month`, `datediff`, `add_months` | 03_silver, 04_gold |
| Agregação | `GROUP BY`, `HAVING`, `COUNT`, `SUM`, `AVG`, `MIN/MAX`, `CASE WHEN` | 03_silver |
| Junções | `INNER/LEFT JOIN`, chaves compostas, join com calendário | 03_silver, 05 |
| Estrutura | `WITH` (CTEs), subqueries, `CREATE OR REPLACE TABLE/VIEW` | 04_gold |
| Window | `LAG/LEAD`, `SUM/AVG OVER`, `ROWS BETWEEN`, `RANK/DENSE_RANK/ROW_NUMBER`, `FIRST_VALUE`, `QUALIFY` | 04_gold, 05 |
| Estatística | `STDDEV`, `CORR`, `EXP(SUM(LN()))` para produto acumulado, `PERCENTILE` | 04_gold, 05 |
| Reshape | `PIVOT`, `UNPIVOT`, `COLLECT_LIST` | 05 |
| Delta | `MERGE INTO`, `DESCRIBE HISTORY`, `OPTIMIZE` | 04_gold (v2: ingestão incremental) |

## Escada de DAX — do básico ao avançado
| Nível | Medidas do projeto | Funções |
|---|---|---|
| Básico | Valor médio, contagem de meses, retorno médio mensal | `SUM`, `AVERAGE`, `COUNTROWS`, `DIVIDE` |
| Contexto | Retorno por perfil vs total, % sobre subtotal | `CALCULATE`, `ALL`, `ALLSELECTED`, `KEEPFILTERS` |
| Tempo | Retorno 12 / 36 / 60 meses, retorno no ano | `DATESINPERIOD`, `DATEADD`, `SAMEPERIODLASTYEAR`, `TOTALYTD` |
| Acumulado | R$ 10 mil → hoje; índice base 100 | `PRODUCTX`, `FILTER`, `MAX` de data |
| Variáveis | Drawdown máximo, meses até recuperar | `VAR`, `MAXX`, `CALCULATETABLE` |
| Ranking | Melhor classe do ano, segmentação por faixa de risco | `RANKX`, `SWITCH`, `TOPN` |
| What-if | Valor inicial ajustável pelo usuário | parâmetro what-if, `SELECTEDVALUE` |
| Visual | Destacar maior/menor valor, cor condicional por medida | medidas de formatação condicional |

## Git e GitHub — o que o mercado pede e onde aprender
O necessário para vaga de dados: clonar, `add/commit/push/pull`, mensagens de commit claras, `.gitignore`, branches, abrir e revisar *pull request*, README bem escrito, tags/releases. Neste projeto você pratica de dois lugares: o Git folder do Databricks (commit pela interface) e o terminal local (VS Code).

Fonte principal: **Xperiun · Git & GitHub para Colaboração**.

| Aulas | Duração | Quando |
|---|---|---|
| "Git & GitHub"; "Meu Primeiro Commit" | ~38 min | seg 14 |
| "Conectando meu repositório local ao GitHub", "Resumo dos novos comandos Git" | ~23 min | ter 15 |
| "Introdução a branches", "Conceitos essenciais", "Pull Request" | ~41 min | qui 17 |
| "Mão na massa com Branch" | 35 min, opcional | fds 19–20 |
| Setup profissional para Eng. de Dados: "venv e uv", "Subindo no GitHub", "colega clonando", "PR e merge" | ~64 min, opcional | fds 26–27 |

Reserva: GitHub Skills *Introduction to GitHub* (inglês); *Pro Git* caps. 1–3 (git-scm.com/book/pt-br).

## Trilha teórica mínima — Xperiun primeiro
| Pilar | Xperiun | Complemento |
|---|---|---|
| Git/GitHub | *Git & GitHub para Colaboração* | GitHub Skills (opcional) |
| Python | *Python para Análise de Dados* — "Funções", "Arquivo CSV" | **pandas não está no Xperiun:** Kaggle *Pandas* 1–4 |
| Databricks/SQL | *Databricks com Spark* — "Databricks SQL" (5 aulas), "Manipulando datas", "Window functions", "Delta Lake I", "Conectando os seus dados no Power BI" (4), bônus "Lendo dados da API no Databricks"; *Banco de Dados e Linguagem SQL* (em andamento) para funções específicas | docs do Free Edition (Volumes, Unity Catalog, Git folders) |
| Previsão | *Introdução à ML* — "Data Leakage" | Kaggle *Time Series* 1–2; quickstart do Prophet |
| EDA | *Introdução à ML* — módulo "Análise Exploratória" | — |
| Power BI | *Modelagem de Dados Avançado*; *DAX Avançado* (CALCULATE, inteligência de tempo, acumulado, médias móveis, visuais) | SQLBI *DAX Guide* |
| KPIs | *Business Analytics: Indicadores e KPIs* (em andamento) | — |
| Finanças | — | 15 min no Tesouro Direto ou B3 Educação |

## Como usar a IA sem perder o aprendizado
1. Pedir explicação **antes** do código: "me explique como fazer X, depois me mostre".
2. Escrever a primeira versão sozinha; a IA revisa, aponta erros e sugere a forma idiomática.
3. Toda função gerada com IA é reescrita com as próprias palavras nos comentários.
4. Quando travar num conceito, a IA aponta a aula e o minuto no Xperiun em vez de explicar por cima do professor.
5. O diário (`docs/diario.md`) registra o que a IA fez que você ainda não saberia fazer — isso vira a pauta da v2.
