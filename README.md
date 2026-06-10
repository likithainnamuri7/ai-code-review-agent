# Production-code-review-agent
Production-code-review-agent
Overview CodeSentinel is an end-to-end ML system that automates code review at the pull request level. The core pipeline combines a three-stage hybrid retrieval system with a QLoRA fine-tuned CodeLlama-7B model served via vLLM, orchestrated through a LangGraph ReAct agent that can execute code, run tests, check CVEs, and apply static analysis — all within a bounded 30-second execution budget. Reviews are posted natively to GitHub as structured review objects with inline comments, severity badges (critical / warning / info), and generated fix patches. A CI-gated evaluation harness enforces F1, false positive rate, and latency thresholds before any model update ships to production.

Key Features

1.Hybrid RAG retrieval — BM25 sparse search (Elasticsearch) + dense FAISS retrieval (CodeBERT embeddings) + cross-encoder reranking, producing a 4k-token context window of the most relevant codebase snippets.

2.QLoRA fine-tuned model — CodeLlama-7B trained on 50,000 real GitHub PR + review pairs using 4-bit NF4 quantization; served via vLLM with speculative decoding for ~3× throughput.

3.Multi-head analysis — four parallel inference calls covering bug detection, performance, security (OWASP Top 10), and code quality; all outputs schema-constrained via outlines

4.Agentic tool-use — LangGraph ReAct loop with E2B sandboxed code execution, pytest/jest test running, CVE lookup, semgrep static analysis, and type checking

5.Structured GitHub reviews — posts native review objects with APPROVE, COMMENT, or REQUEST_CHANGES events determined automatically by finding severity

6.CI-gated eval pipeline — F1 ≥ 0.82, FPR ≤ 8%, and p95 latency ≤ 45s enforced as merge gates; Prometheus time-series for regression trend detection

6.Full observability — per-trace LLM observability via Langfuse, Grafana dashboards, and PagerDuty SLA alerts

Tech Stack LayerTechnologyAPI serverFastAPI · uvicorn Webhook securityHMAC SHA-256 signature validation Code parsingtree-sitter 0.22 · unidiff · gitpython Sparse retrievalElasticsearch 8.x Dense retrievalfaiss-gpu · microsoft/codebert-base Rerankingcross-encoder/ms-marco-MiniLM-L-6-v2 Base modelcodellama/CodeLlama-7b-Instruct-hf Fine-tuningtrl · peft · bitsandbytes Training accelerationDeepSpeed ZeRO-2 Inference servervllm 0.4+ with PagedAttention Structured outputoutlines Agentic frameworkLangGraph 0.1+ Code sandboxE2B Static analysissemgrep · bandit · eslint Type checkingmypy · tsc LLM observabilityLangfuse MetricsPrometheus · Grafana Alerting PagerDuty Containerization Docker · Docker Compose Orchestration Kubernetes

Prerequisites: Python 3.11+ Docker and Docker Compose NVIDIA GPU with CUDA 12.1+ (CPU-only mode is not supported at production throughput targets) A GitHub App with pull_requests: write and contents: read permissions API credentials: GITHUB_APP_PRIVATE_KEY, E2B_API_KEY, BRAVE_SEARCH_API_KEY, LANGFUSE_SECRET_KEY

Contributing

Fork the repository and create a feature branch: git checkout -b feat/your-feature Make your changes and write tests for new behaviour Run the test suite: pytest tests/ -v Run a quick eval to confirm no regressions: python eval/harness.py --quick Open a pull request — CodeSentinel will review its own PR

License This project is licensed under the MIT License. See LICENSE for details.

Built with CodeLlama · HuggingFace · LangGraph · FAISS · FastAPI · vLLM · Langfuse · Grafana
