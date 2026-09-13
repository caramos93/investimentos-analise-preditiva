# Projeto Pessoal Um

Projeto pessoal de análise e previsão de investimentos com Python + IA, pensado como peça de portfólio (GitHub/LinkedIn).

Ver [requisitos.md](requisitos.md) para objetivo, motivação e focos.

## Estrutura proposta

```
Projeto Pessoal Um/
├── requisitos.md      # brief do projeto (fonte da verdade)
├── README.md          # este arquivo
├── docs/              # estudo de tipos de investimento, decisões, anotações
├── data/
│   ├── raw/           # dados públicos brutos, como baixados
│   └── processed/     # dados tratados prontos para análise
├── notebooks/         # exploração e experimentos em Python
├── src/               # scripts reutilizáveis (coleta, tratamento, modelos)
└── dashboard/         # produto final
```

## Próximos passos (a definir com a Caroline)
- [ ] Escolher fontes de dados públicas (ex.: Banco Central/SGS, B3, Tesouro Direto, Yahoo Finance, CVM)
- [ ] Definir quais classes de investimento entram no escopo (do conservador ao agressivo)
- [ ] Definir a stack do dashboard (Streamlit, Power BI, Dash, HTML)
- [ ] Definir o tipo de previsão (séries temporais: ARIMA/Prophet/ML)
- [ ] Criar o repositório no GitHub
