# Plano de 17 dias — Projeto Pessoal Um (v1)

Criado em 13/09/2026 (domingo). Início: **seg 14/09/2026**. Entrega: **qua 30/09/2026**.
Substitui o plano de 12 semanas, que passa a ser o backlog da versão 2 (`docs/backlog_v2_plano_12_semanas.md`).

## Orçamento de tempo (~44h)
| Período | Horas | Uso |
|---|---|---|
| Seg–qui (14–17/09 e 21–24/09) | 1h manhã + 1h noite | manhã = prática; noite = teoria curta (≤40 min) + diário (10 min) |
| Sex 18 e 25/09 | 2–3h | bloco de prática |
| Sáb 19 e 26/09 | 5h | bloco profundo — a maior entrega da semana |
| Dom 20 e 27/09 | 3–4h | fechar, commitar, revisar |
| Seg–qua 28–30/09 | 2h/dia | acabamento e publicação |

Regras: **manhã nunca é teoria; noite nunca é código pesado.** Todo dia termina com um commit, mesmo pequeno. Se algo travar por mais de 30 min, anotar no diário e me chamar — não gastar a hora tentando sozinha.

## O que entra e o que fica para a v2
| Entra na v1 (17 dias) | Fica para a v2 |
|---|---|
| BCB SGS: Selic, CDI, IPCA, PTAX | Tesouro Transparente (CSV de títulos) |
| yfinance: ^BVSP, BOVA11, IVVB11, HGLG11, BTC-USD | small caps, mais FIIs, ETH, CoinGecko |
| Databricks: bronze → silver → gold em SQL | Delta time travel, MERGE incremental |
| Modelo: baseline naive + Prophet, holdout de 12 meses | ARIMA/SARIMA, LightGBM, backtest com janela móvel |
| Power BI via CSV exportado do gold (3 páginas) | conector Databricks nativo, simulador what-if avançado |
| Retorno acumulado, retorno real, volatilidade, drawdown | Sharpe, correlação no Power BI |
| README com a história + post no LinkedIn | GitHub Pages, gif animado do dashboard |

## Escada de risco (v1)
| Perfil | Ativo no projeto | Série |
|---|---|---|
| Conservador | CDB pós-fixado / Tesouro Selic | CDI (SGS 12) e Selic (SGS 11) |
| Moderado | Fundo imobiliário | HGLG11.SA |
| Arrojado | Ibovespa e S&P 500 em reais | ^BVSP, BOVA11.SA, IVVB11.SA |
| Agressivo | Bitcoin | BTC-USD × PTAX (SGS 1) |
| Referência | Inflação | IPCA (SGS 433) |

## Dia a dia

### Semana 1 — Coleta em Python, Git e Databricks
| Dia | Manhã (prática, 1h) | Noite (teoria, ≤40 min) |
|---|---|---|
| **Seg 14** | O que é uma API (10 min de explicação). Escrever `src/coleta_bcb.py`: `requests.get` na Selic (SGS 11), transformar em DataFrame, salvar CSV. | **Git 1 (Xperiun · Git & GitHub para Colaboração):** "Git & GitHub" (21 min) + "Meu Primeiro Commit" (17 min). Fazer o commit e push do dia com as próprias mãos. |
| **Ter 15** | Generalizar em `busca_sgs(codigo, inicio, fim)`; coletar CDI, IPCA, PTAX; salvar em parquet em `data/raw/`. | **Git 1b + Python:** Xperiun Git — "Conectando meu repositório local ao GitHub" (12 min) e "Resumo dos novos comandos" (11 min). Xperiun Python — "Funções" teoria (12 min). |
| **Qua 16** | `src/coleta_yf.py` com yfinance: ^BVSP, BOVA11.SA, IVVB11.SA, HGLG11.SA, BTC-USD desde 2015; salvar parquet. | **Pandas:** Kaggle *Pandas* lições 1–3 (não há pandas no Xperiun). 15 min: pós × pré × inflação × variável. |
| **Qui 17** | `src/coleta.py` que roda tudo e gera uma tabela longa única: `data, ativo, valor, fonte`. Escrever `docs/fontes_de_dados.md`. | **Git 2 (Xperiun):** "Introdução a branches" (12 min), "Conceitos essenciais" (24 min), "Pull Request" (5 min). Abrir a primeira PR do projeto (branch `coleta`) e fazer o merge. |
| **Sex 18** (2–3h) | Antes (40 min, Xperiun · Databricks com Spark): módulo "Databricks SQL" aulas 1–4 e "Delta Lake — Parte I". Depois: criar catálogo/schema, subir os parquets para um Volume, criar tabelas **bronze** (dados como chegaram). | — |
| **Sáb 19** (5h) | **Silver:** tipos corrigidos, datas, deduplicação, tudo em frequência mensal (último valor do mês; CDI/Selic acumulados no mês), `dim_ativo` (ativo, classe, perfil). **Gold:** retorno mensal (`LAG`), retorno acumulado, retorno real (descontado IPCA), volatilidade 12m (`STDDEV OVER`), drawdown máximo. Exportar gold em CSV para `data/processed/`. Notebooks SQL salvos em `src/sql/`. Apoio: Xperiun Databricks — "Manipulando datas" (10 min) e "Window functions" (9 min). | — |
| **Dom 20** (3–4h) | `notebooks/01_exploracao.ipynb`: "R$ 10 mil em jan/2015 em cada perfil", gráfico por classe, heatmap de correlação, tabela risco × retorno, **5 conclusões escritas**. Commit. Apoio: Xperiun · Introdução à ML — módulo "Análise Exploratória", aulas de Matplotlib (estrutura, paradigmas) e Seaborn. | — |

### Semana 2 — Modelo preditivo e Power BI
| Dia | Manhã (prática, 1h) | Noite (teoria, ≤40 min) |
|---|---|---|
| **Seg 21** | Baselines para Selic, IPCA e Ibovespa mensais: *naive* (repete o último valor) e média móvel 12m. Holdout: treinar até ago/2025, testar os 12 meses seguintes. MAE e MAPE. | **Séries temporais 1:** Kaggle *Time Series* lições 1–2 (tendência, sazonalidade). Xperiun · Introdução à ML — "Data Leakage" (por que nunca embaralhar o tempo). |
| **Ter 22** | Prophet nas mesmas 3 séries; comparar com baseline na tabela. Se Prophet não bater o naive no Ibovespa, isso **é** o resultado — e vai no README. | **Séries temporais 2:** quickstart do Prophet; intervalo de confiança. |
| **Qua 23** | Previsão de 6 meses à frente com intervalo; gerar `data/processed/fato_previsao.csv` (data, ativo, modelo, previsto, inf, sup). `docs/resultados_modelo.md`. | **Power BI 1 (Xperiun · Modelagem de Dados Avançado):** "Star Schema ou Snowflake?" e a aula de tabela calendário / relacionamentos bidirecionais. |
| **Qui 24** | Power BI: importar CSVs gold + previsão; `dim_calendario`, `dim_ativo`, `fato_mensal`, `fato_previsao`; relacionamentos. | **Power BI 2 (Xperiun · DAX Avançado):** "Revisando Contexto de Filtro" (10 min), "Filtrando expressões com a CALCULATE" (10 min), "Removendo filtros com ALL" (12 min). |
| **Sex 25** (2–3h) | Antes (30 min, Xperiun · DAX Avançado): "Acumulado infinito com ALL/ALLSELECTED" (9 min) e "Totais e médias móveis com DATESINPERIOD" (14 min). Depois: medidas de retorno acumulado, retorno real, volatilidade 12m, drawdown, retorno 12m/36m. | — |
| **Sáb 26** (5h) | Páginas: **(1)** visão geral — KPIs por perfil e evolução de R$ 10 mil; **(2)** risco × retorno — dispersão vol × retorno, drawdown; **(3)** previsões — previsto × realizado no holdout e projeção 6 meses com faixa. | — |
| **Dom 27** (3–4h) | Acabamento visual (tema, títulos, tooltips), capturas de tela para o README, `dashboard/investimentos.pbix` commitado. | — |

### Reta final — Portfólio
| Dia | 2h |
|---|---|
| **Seg 28** | README final: problema → dados → pipeline → modelo → dashboard → conclusões, com diagrama e prints. Limpar notebooks. |
| **Ter 29** | Rodar tudo do zero (`coleta.py` → SQL → notebook) para garantir reprodutibilidade. `docs/aprendizados.md`. Rascunho do post. |
| **Qua 30** | Tag `v1.0` no GitHub (release). Post no LinkedIn com link. Projeto adicionado à seção *Projetos* do perfil. **Entregue.** |

## Git e GitHub — o que o mercado pede e onde aprender
O necessário para vaga de dados: clonar, `add/commit/push/pull`, escrever mensagens de commit claras, `.gitignore`, branches, abrir e revisar *pull request*, README bem escrito, tags/releases. Não precisa de rebase, submodules ou CI nesta fase.

Fonte principal: **Xperiun · Git & GitHub para Colaboração** (35 aulas). O que assistir e quando:

| Aulas | Duração | Quando |
|---|---|---|
| Introdução Versionamento → "Git & GitHub"; Praticando o GIT → "Meu Primeiro Commit" | ~38 min | seg 14 |
| Praticando o GIT → "Conectando meu repositório local ao GitHub", "Resumo dos novos comandos Git" | ~23 min | ter 15 |
| Branches → "Introdução a branches", "Conceitos essenciais", "Pull Request" | ~41 min | qui 17 |
| Branches → "Mão na massa com Branch" (35 min) | opcional | fim de semana 19–20 |
| Setup profissional para Eng. de Dados → "Instalando o venv e uv", "Subindo no GitHub", "Simulando um colega clonando", "Iniciando o PR e fazendo o merge" | ~64 min | opcional, fim de semana 26–27 — é exatamente o ambiente que montamos (uv + .venv) |

Reserva em inglês, se quiser praticar dentro do GitHub: GitHub Skills *Introduction to GitHub* (skills.github.com). Referência para dúvidas: *Pro Git* caps. 1–3 (git-scm.com/book/pt-br).

## Trilha teórica mínima (só o que o dia exige)
Fonte principal: **Xperiun Ultra** (já conectado). Complementos externos só onde o catálogo não cobre.

| Pilar | Xperiun | Complemento |
|---|---|---|
| Git/GitHub | *Git & GitHub para Colaboração* (tabela acima) | GitHub Skills (inglês, opcional) |
| Python | *Python para Análise de Dados* — "Funções", "Arquivo CSV" | **pandas não está no Xperiun:** Kaggle *Pandas* lições 1–4 |
| SQL/Databricks | *Databricks com Spark* — "Databricks SQL" (5 aulas), "Manipulando datas", "Window functions", "Delta Lake I", "Conectando os seus dados no Power BI" (4 aulas), bônus "Lendo dados da API no Databricks" | documentação do Free Edition (Volumes, Unity Catalog) |
| Previsão | *Introdução à Machine Learning* — "Data Leakage"; *Estatística para Análise de Dados* (visão geral) | Kaggle *Time Series* lições 1–2; quickstart do Prophet |
| EDA | *Introdução à Machine Learning* — módulo "Análise Exploratória de Dados" (Matplotlib, Seaborn) | — |
| Power BI | *Modelagem de Dados Avançado* (star schema, calendário); *DAX Avançado* (CALCULATE, inteligência de tempo, acumulado, médias móveis, "Usando DAX em Visuais") | SQLBI *DAX Guide* para consultar uma função |
| KPIs | *Business Analytics: Indicadores e KPIs* (já em andamento, 14/58) — só para desenhar a página 1 | — |
| Finanças | — | 15 min no site do Tesouro Direto ou B3 Educação |

## Como usar a IA sem perder o aprendizado
1. Pedir explicação **antes** do código: "me explique como fazer X, depois me mostre".
2. Escrever a primeira versão sozinha; a IA revisa, aponta erros e sugere a forma idiomática.
3. Toda função gerada com IA é reescrita com as próprias palavras nos comentários.
4. O diário (`docs/diario.md`) registra o que a IA fez que você ainda não saberia fazer — isso vira a pauta da v2.
