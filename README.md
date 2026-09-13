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

O projeto segue um plano de 12 semanas (14/09 → 06/12/2026) com sete fases:
coleta via API em Python → tratamento em SQL no Databricks → análise exploratória →
modelo preditivo de séries temporais → dashboard em Power BI → portfólio.
Detalhes em [docs/plano_12_semanas.md](docs/plano_12_semanas.md).
