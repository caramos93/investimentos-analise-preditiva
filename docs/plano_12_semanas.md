# Plano de 12 semanas — Projeto Pessoal Um

Criado em 13/09/2026 (domingo). Início: **segunda, 14/09/2026**. Término previsto: **domingo, 06/12/2026**.

## Ponto de partida
| Pilar | Hoje | Meta ao fim das 12 semanas |
|---|---|---|
| Python | básico, pouca prática | coletar dados via API, tratar com pandas, treinar e avaliar modelo de séries temporais |
| SQL Databricks | forte, uso diário | arquitetura bronze/silver/gold própria, window functions, Delta, tabelas prontas para BI |
| Power BI | manutenção do que já existe | modelo estrela e dashboard criados do zero, com DAX próprio |
| Finanças | conhecimento e interesse fortes | domínio conceitual das classes (conservador → agressivo) e das métricas de risco/retorno |
| Git/GitHub | conta existente, sem projetos próprios | repositório público, commits semanais, README que conta a história |

## Orçamento de tempo (~14h/semana)
- **Seg–qui, manhã (1h, cabeça fresca)** → mão na massa: a tarefa prática da semana (código, SQL ou Power BI). Nunca teoria.
- **Seg–qui, noite (1h, cansada)** → teoria leve: leitura/vídeo do tema da semana + 10 min no diário de aprendizado (`docs/diario.md`: o que fiz, o que travou, o que aprendi). Nunca código pesado.
- **Sexta** → folga ou revisão leve da semana. Não iniciar tarefa nova.
- **Sábado (3–4h)** → bloco de trabalho profundo: a entrega da semana, integração, o que exige concentração contínua.
- **Domingo (1–2h)** → revisar, commitar, atualizar o README com o resumo da semana, planejar a próxima.

Regra de ouro: **cada semana termina com um commit no GitHub**. Se a semana apertar, corta-se teoria, não a entrega.

## Pipeline do projeto (ponta a ponta)
```
APIs públicas ──► Python (coleta) ──► data/raw ──► Databricks Free Edition
(BCB, Tesouro,     requests/yfinance    parquet/csv    bronze → silver → gold (SQL)
 B3, cripto)                                                  │
                                                              ▼
Power BI Desktop ◄── tabelas gold + previsões ◄── Python (modelo preditivo)
(modelo estrela,        (conector Databricks           statsmodels / Prophet
 KPIs, DAX)              ou export csv/parquet)         backtesting
```

## Escopo de investimentos (escada de risco)
| Perfil | Ativos no projeto | Fonte de dados |
|---|---|---|
| Conservador | Poupança, Tesouro Selic, CDB pós-fixado (CDI) | BCB SGS: Selic (11), CDI (12), Poupança (conferir código no catálogo SGS) |
| Moderado | Tesouro IPCA+, Tesouro Prefixado, FIIs | Tesouro Transparente (preços e taxas); yfinance: HGLG11.SA, XPML11.SA |
| Arrojado | Ações BR, ETFs (Ibovespa, S&P 500 em reais), small caps | yfinance: ^BVSP, BOVA11.SA, IVVB11.SA, SMAL11.SA, PETR4.SA, VALE3.SA |
| Agressivo | Cripto (BTC, ETH) | yfinance: BTC-USD, ETH-USD (+ dólar PTAX, SGS 1) ou CoinGecko |
| Referência | IPCA (inflação), dólar | BCB SGS: IPCA (433), PTAX venda (1), meta Selic (432) |

API do BCB (sem cadastro, sem chave):
`https://api.bcb.gov.br/dados/serie/bcdata.sgs.{codigo}/dados?formato=json&dataInicial=01/01/2015&dataFinal=31/12/2026`

## Fases e semanas

### Fase 0 — Fundação · Semana 1 (14–20/09)
**Prática:** criar repositório no GitHub; instalar Python (Miniconda ou uv) + VS Code; criar ambiente `venv` com pandas, requests, yfinance, matplotlib, jupyter; criar conta no Databricks Free Edition; instalar Power BI Desktop; primeiro `requests.get` na API do BCB trazendo a Selic e salvando um CSV.
**Teoria:** Git básico (clone, add, commit, push, .gitignore); panorama das classes de investimento (renda fixa × variável, pós × pré × inflação).
**Entrega:** repositório público com README, `.gitignore`, ambiente reproduzível (`requirements.txt`) e `src/coleta_bcb.py` funcionando.

### Fase 1 — Coleta de dados · Semanas 2–3 (21/09–04/10)
**Prática (S2):** função genérica `busca_sgs(codigo, inicio, fim)`; coletar Selic, CDI, IPCA, PTAX; salvar em `data/raw/` como parquet; script único `src/coleta.py` que roda tudo.
**Prática (S3):** yfinance para índices, ETFs, FIIs, ações e cripto; baixar CSV do Tesouro Transparente e ler com pandas; padronizar todas as fontes num mesmo formato (`data`, `ativo`, `valor`, `fonte`).
**Teoria:** pandas (DataFrame, index de datas, `resample`, `merge`); renda fixa a fundo: Selic × CDI, taxa nominal × real, marcação a mercado, por que Tesouro IPCA+ oscila.
**Entrega:** `python src/coleta.py` recria toda a `data/raw/` do zero. Documentar cada fonte em `docs/fontes_de_dados.md`.

### Fase 2 — Tratamento em SQL (Databricks) · Semanas 4–5 (05–18/10)
**Prática (S4):** subir os arquivos para um Volume no Unity Catalog; criar schema `bronze` (dados como chegaram); `silver` com tipos corrigidos, datas padronizadas, deduplicação e calendário de dias úteis; tabela `dim_ativo` (ativo, classe, perfil de risco).
**Prática (S5):** camada `gold`: retornos diários e mensais (`LAG`, window functions), retorno acumulado, retorno real (descontado IPCA), volatilidade móvel (`STDDEV OVER`), drawdown máximo, correlação mensal entre classes. Tudo em SQL.
**Teoria:** arquitetura medalhão, Delta Lake (time travel, `MERGE`), window functions avançadas; métricas financeiras: volatilidade anualizada, Sharpe (excedente sobre o CDI), drawdown, correlação e diversificação.
**Entrega:** notebooks SQL versionados em `src/sql/` (bronze → silver → gold) e `docs/dicionario_gold.md` explicando cada coluna.

### Fase 3 — Análise exploratória · Semana 6 (19–25/10)
**Prática:** trazer as tabelas gold para o Python (pandas), gráficos de série temporal por classe, "R$ 10 mil investidos em 2015 em cada perfil", heatmap de correlação, tabela resumo risco × retorno. Escrever as 5 conclusões principais.
**Teoria:** matplotlib/seaborn; componentes de uma série temporal (tendência, sazonalidade, ruído); estacionariedade.
**Entrega:** `notebooks/01_exploracao.ipynb` limpo e comentado. **Checkpoint:** post no LinkedIn mostrando o progresso (build in public).

### Fase 4 — Modelo preditivo · Semanas 7–9 (26/10–15/11)
**Prática (S7):** baselines (naive, média móvel, sazonal naive) para Selic, IPCA e Ibovespa; backtest com janela móvel; métricas MAE, RMSE, MAPE. Sem baseline não há como saber se o modelo é bom.
**Prática (S8):** ARIMA/SARIMA com `statsmodels` (ACF/PACF, auto-seleção de ordem) e Prophet; previsão 6 e 12 meses com intervalo de confiança.
**Prática (S9):** modelo de ML (features de lags + LightGBM ou RandomForest) para comparar; escolher o melhor por série; gerar tabela `fato_previsao` (data, ativo, modelo, previsto, limite inferior/superior) e gravar no Databricks.
**Teoria:** *Forecasting: Principles and Practice* (caps. 1–5, 8–9); Kaggle *Time Series*; validação temporal (nunca embaralhar o tempo); por que prever preço de ação não funciona e prever tendência/volatilidade é o realista.
**Entrega:** `notebooks/02_modelagem.ipynb` + `src/modelo.py` reproduzível + `docs/resultados_modelo.md` com a comparação dos modelos.

### Fase 5 — Power BI · Semanas 10–11 (16–29/11)
**Prática (S10):** conectar ao Databricks (conector nativo; alternativa: exportar gold em parquet/csv); modelo estrela: `dim_calendario`, `dim_ativo`, `fato_cotacoes`, `fato_previsao`; medidas DAX: retorno acumulado, CAGR, retorno real, volatilidade, drawdown, Sharpe.
**Prática (S11):** páginas: (1) visão geral com KPIs por perfil, (2) risco × retorno, (3) previsões com intervalo de confiança vs. realizado, (4) simulador "quanto virariam R$ 10 mil". Tema visual próprio, tooltips, bookmarks.
**Teoria:** Microsoft Learn (trilha PL-300: modelagem e DAX); SQLBI (*Introducing DAX*, contexto de filtro × contexto de linha); boas práticas de design de dashboard.
**Entrega:** `dashboard/investimentos.pbix` + capturas de tela/gif no README.

### Fase 6 — Portfólio · Semana 12 (30/11–06/12)
**Prática:** README final contando a história (problema → dados → pipeline → modelo → dashboard → conclusões), com arquitetura e prints; limpar notebooks; `docs/aprendizados.md`; post no LinkedIn com link do repositório; adicionar o projeto na seção Projetos do LinkedIn.
**Entrega:** repositório apresentável e post publicado.

## Trilha de estudos por pilar (todos gratuitos)
- **Python** — Kaggle Learn: *Python* → *Pandas* → *Time Series*; livro *Python for Data Analysis* (Wes McKinney, 3ª ed., gratuito em wesmckinney.com/book). Hábito: toda vez que abriria o Excel, tentar no pandas.
- **SQL / Databricks** — Databricks Academy (cursos gratuitos: *Lakehouse Fundamentals*, trilha *Data Analyst*); documentação do Free Edition; docs de Delta Lake e window functions.
- **Power BI** — Microsoft Learn, trilha *Power BI Data Analyst (PL-300)*; SQLBI: *DAX Guide* e curso gratuito *Introducing DAX*; conceito de modelo estrela.
- **Finanças** — B3 Educação (cursos gratuitos); site do Tesouro Direto (simuladores e explicações por título); Banco Central (Cidadania Financeira); programa da CPA-10 (ANBIMA) como checklist de conceitos.
- **Previsão** — *Forecasting: Principles and Practice* (otexts.com/fpp3, gratuito; exemplos em R, mas a teoria é a que importa); Kaggle *Time Series*; documentação de statsmodels e Prophet.
- **Git** — GitHub Skills (skills.github.com); *Pro Git* (git-scm.com/book, em português).

## Como usar a IA neste projeto (sem perder o aprendizado)
- Pedir explicação **antes** do código: "me explique como fazer X, depois me mostre".
- Escrever a primeira versão sozinha; usar a IA para revisar, apontar erros e sugerir a forma idiomática.
- Toda função gerada com IA: reescrever com as próprias palavras nos comentários.
- Registrar no diário o que a IA fez que você ainda não saberia fazer — essa lista é a sua pauta de estudo.
