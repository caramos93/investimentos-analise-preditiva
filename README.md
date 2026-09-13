# Horizonte Investimentos — análise e previsão de investimentos

Caso de dados de ponta a ponta para uma assessoria de investimentos fictícia: ingestão de dados públicos por API,
tratamento em SQL e análise preditiva em Python dentro do Databricks, e dashboard em Power BI com modelagem
relacional e DAX. Oito perguntas de negócio guiam o projeto (ver `docs/plano_17_dias.md`).

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
├── notebooks/         # notebooks do Databricks (Python e SQL), numerados 01–08
├── src/               # scripts de apoio para rodar localmente
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

A v1 é construída em 17 dias (14/09 → 30/09/2026), cobrindo o pipeline inteiro no Databricks:
ingestão via API (Python) → bronze → silver → gold (SQL) → análise e previsão (pandas, Prophet) →
Power BI (modelo estrela + DAX) → publicação.
Dia a dia em [docs/plano_17_dias.md](docs/plano_17_dias.md).
O que ficou de fora está em [docs/backlog_v2_plano_12_semanas.md](docs/backlog_v2_plano_12_semanas.md).
