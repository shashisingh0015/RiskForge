# RiskForge

**A mini AI & ML platform for real-time fraud scoring.** It covers training and a model registry, a feature store, model serving, an LLM gateway, and measurement, all runnable with one command.

> 🚧 Work in progress. This README will grow as each component lands.

## Why

Product teams want to ship ML and LLM features without each team building its own infrastructure. RiskForge is a small, end-to-end platform that shows how a platform team can make that fast and reliable. Its example use case is scoring mobile-money transactions for fraud.

## Architecture (planned)

| Component | Tools | Folder |
|---|---|---|
| Training & registry | Python, LightGBM, MLflow, ONNX | `training/` |
| Feature store | Feast, Parquet, Redis | `features/` |
| Model serving | FastAPI, ONNX Runtime, k6 | `serving/` |
| LLM gateway | ASP.NET Core, Anthropic C# SDK, Ollama | `gateway/` |
| Evaluation | Evidently, LLM eval set | `evals/` |
| Infra & delivery | Docker Compose, GitHub Actions, OpenTelemetry, Prometheus, Grafana | `infra/` |
| Decisions | Architecture Decision Records | `docs/adr/` |

## Data

[PaySim](https://www.kaggle.com/datasets/ealaxi/paysim1) synthetic mobile-money transactions. The data is not committed; see [docs/data.md](docs/data.md).

## Quickstart

_Coming soon: `docker compose up`._

## License

[MIT](LICENSE)
