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

## Como rodar

Requisitos: [uv](https://docs.astral.sh/uv/) (gerencia o Python e o ambiente virtual).

```bash
uv venv --python 3.12 .venv
uv pip install --python .venv -r requirements.txt
.venv\Scriptsctivate        # Windows
```

## Plano e cronograma

A v1 é construída em 17 dias (14/09 → 30/09/2026), cobrindo o pipeline inteiro:
coleta via API em Python → tratamento em SQL no Databricks → análise exploratória →
modelo preditivo (baseline + Prophet) → dashboard em Power BI → publicação.
Dia a dia em [docs/plano_17_dias.md](docs/plano_17_dias.md).
O que ficou de fora está em [docs/backlog_v2_plano_12_semanas.md](docs/backlog_v2_plano_12_semanas.md).
