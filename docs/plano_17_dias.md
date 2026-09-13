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
| **Seg 14** | O que é uma API (10 min de explicação). Escrever `src/coleta_bcb.py`: `requests.get` na Selic (SGS 11), transformar em DataFrame, salvar CSV. | **Git 1:** GitHub Skills *Introduction to GitHub* (1h, em inglês, prático). Fazer o commit e push do dia com as próprias mãos. |
| **Ter 15** | Generalizar em `busca_sgs(codigo, inicio, fim)`; coletar CDI, IPCA, PTAX; salvar em parquet em `data/raw/`. | **Pandas 1:** Kaggle *Pandas* lições 1–3 (criar, ler, selecionar, resumir). |
| **Qua 16** | `src/coleta_yf.py` com yfinance: ^BVSP, BOVA11.SA, IVVB11.SA, HGLG11.SA, BTC-USD desde 2015; salvar parquet. | **Pandas 2:** `resample` mensal e `merge`. Classes de investimento: pós × pré × inflação × variável (20 min). |
| **Qui 17** | `src/coleta.py` que roda tudo e gera uma tabela longa única: `data, ativo, valor, fonte`. Escrever `docs/fontes_de_dados.md`. | **Git 2:** branches e pull request — GitHub Skills *Review pull requests*. Abrir a primeira PR do projeto (branch `coleta`) e fazer o merge. |
| **Sex 18** (2–3h) | Databricks: criar catálogo/schema, subir os parquets para um Volume, criar tabelas **bronze** (dados como chegaram). | — |
| **Sáb 19** (5h) | **Silver:** tipos corrigidos, datas, deduplicação, tudo em frequência mensal (último valor do mês; CDI/Selic acumulados no mês), `dim_ativo` (ativo, classe, perfil). **Gold:** retorno mensal (`LAG`), retorno acumulado, retorno real (descontado IPCA), volatilidade 12m (`STDDEV OVER`), drawdown máximo. Exportar gold em CSV para `data/processed/`. Notebooks SQL salvos em `src/sql/`. | — |
| **Dom 20** (3–4h) | `notebooks/01_exploracao.ipynb`: "R$ 10 mil em jan/2015 em cada perfil", gráfico por classe, heatmap de correlação, tabela risco × retorno, **5 conclusões escritas**. Commit. | — |

### Semana 2 — Modelo preditivo e Power BI
| Dia | Manhã (prática, 1h) | Noite (teoria, ≤40 min) |
|---|---|---|
| **Seg 21** | Baselines para Selic, IPCA e Ibovespa mensais: *naive* (repete o último valor) e média móvel 12m. Holdout: treinar até ago/2025, testar os 12 meses seguintes. MAE e MAPE. | **Séries temporais 1:** Kaggle *Time Series* lições 1–2 (tendência, sazonalidade). Por que nunca embaralhar o tempo. |
| **Ter 22** | Prophet nas mesmas 3 séries; comparar com baseline na tabela. Se Prophet não bater o naive no Ibovespa, isso **é** o resultado — e vai no README. | **Séries temporais 2:** quickstart do Prophet; intervalo de confiança. |
| **Qua 23** | Previsão de 6 meses à frente com intervalo; gerar `data/processed/fato_previsao.csv` (data, ativo, modelo, previsto, inf, sup). `docs/resultados_modelo.md`. | **Power BI 1 (Xperiun):** modelo estrela, relacionamentos, tabela calendário. |
| **Qui 24** | Power BI: importar CSVs gold + previsão; `dim_calendario`, `dim_ativo`, `fato_mensal`, `fato_previsao`; relacionamentos. | **Power BI 2 (Xperiun):** DAX básico — medidas, `CALCULATE`, contexto de filtro. |
| **Sex 25** (2–3h) | Medidas DAX: retorno acumulado, retorno real, volatilidade 12m, drawdown, retorno 12m/36m. | — |
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

| Recurso | Formato | Quando |
|---|---|---|
| GitHub Skills — *Introduction to GitHub* (skills.github.com) | prático, ~1h, inglês, dentro do próprio GitHub | seg 14 |
| GitHub Skills — *Review pull requests* | prático, ~1h | qui 17 |
| Curso em Vídeo — *Git e GitHub* (Gustavo Guanabara, YouTube) | vídeo, português, gratuito | reserva, se quiser ver em português |
| *Pro Git*, caps. 1–3 (git-scm.com/book/pt-br) | leitura, português | referência quando surgir dúvida |
| Xperiun — verificar no catálogo se o plano Ultra tem módulo de Git/GitHub | vídeo | se existir, substitui os itens acima |

## Trilha teórica mínima (só o que o dia exige)
- **Python/pandas:** Kaggle *Pandas* (lições 1–4) e Kaggle *Time Series* (lições 1–2). Livro de referência: *Python for Data Analysis* (Wes McKinney, gratuito em wesmckinney.com/book), só para consulta.
- **Power BI / DAX:** **Xperiun Ultra** como fonte principal — módulos de modelagem (estrela, calendário) e DAX (medidas, `CALCULATE`, contexto). Complemento: SQLBI *DAX Guide* para consultar uma função específica.
- **SQL / Databricks:** você já domina; só a documentação do Free Edition (Volumes, Unity Catalog) e, se o Xperiun tiver módulo de Databricks, a aula de ingestão de arquivos.
- **Finanças:** 20 min sobre pós × pré × inflação × renda variável (site do Tesouro Direto ou B3 Educação). O resto é o próprio dado ensinando.
- **Previsão:** quickstart do Prophet + Kaggle *Time Series* já cobrem o que a v1 usa.

## Como usar a IA sem perder o aprendizado
1. Pedir explicação **antes** do código: "me explique como fazer X, depois me mostre".
2. Escrever a primeira versão sozinha; a IA revisa, aponta erros e sugere a forma idiomática.
3. Toda função gerada com IA é reescrita com as próprias palavras nos comentários.
4. O diário (`docs/diario.md`) registra o que a IA fez que você ainda não saberia fazer — isso vira a pauta da v2.
