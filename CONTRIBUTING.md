# Contributing to PredictIQ

Thank you for your interest in contributing! PredictIQ is a research-grade platform and we hold contributions to a high standard. This guide will help you contribute effectively.

## Table of Contents
- [Code of Conduct](#code-of-conduct)
- [Development Setup](#development-setup)
- [Contribution Workflow](#contribution-workflow)
- [Code Standards](#code-standards)
- [ML Model Contributions](#ml-model-contributions)
- [Testing Requirements](#testing-requirements)
- [Documentation](#documentation)
- [Review Process](#review-process)

---

## Code of Conduct

All contributors must follow our [Code of Conduct](CODE_OF_CONDUCT.md). We are committed to maintaining a welcoming, inclusive community for researchers and engineers from all backgrounds.

---

## Development Setup

```bash
# Fork and clone
git clone https://github.com/YOUR_USERNAME/predictive-maintenance-ai-platform.git
cd predictive-maintenance-ai-platform

# Run quickstart (sets up everything)
./scripts/setup/quickstart.sh

# Install pre-commit hooks
pip install pre-commit
pre-commit install
pre-commit install --hook-type commit-msg
```

### Branch Naming Convention

```
feature/short-description          # New features
fix/bug-description                # Bug fixes
research/model-or-experiment-name  # ML experiments
docs/what-you-are-documenting      # Documentation only
refactor/component-name            # Code refactoring
perf/optimization-description      # Performance improvements
```

---

## Contribution Workflow

1. **Open an Issue first** — For features or non-trivial fixes, open an issue to discuss approach before coding. For ML contributions, include a research brief.

2. **Create a branch** from `develop` (not `main`):
   ```bash
   git checkout develop
   git pull upstream develop
   git checkout -b feature/your-feature-name
   ```

3. **Write code** following our standards below.

4. **Write tests** — All contributions must include tests. See [Testing Requirements](#testing-requirements).

5. **Run checks locally**:
   ```bash
   # Linting
   ruff check backend/ ml/
   black backend/ ml/ --check
   mypy backend/app --ignore-missing-imports

   # Tests
   cd backend && pytest tests/ --cov=app --cov-fail-under=90

   # Frontend
   cd frontend && npm run lint && npm run type-check && npm run test:unit
   ```

6. **Commit** with conventional commits format:
   ```
   type(scope): short description

   Longer description if needed.

   Refs: #issue-number
   ```
   Types: `feat`, `fix`, `docs`, `test`, `refactor`, `perf`, `ci`, `chore`, `research`

7. **Open a Pull Request** to `develop` using the PR template.

---

## Code Standards

### Python (Backend + ML)

- **Style**: Black (line length 100) + Ruff (linting) + isort
- **Types**: Full type annotations required (`mypy --strict`)
- **Docstrings**: Google-style with Args/Returns/Raises
- **Error handling**: Use custom exception classes from `app/core/exceptions.py`
- **Logging**: Use `structlog` exclusively, never `print()`
- **Config**: Never hardcode values; use `settings` from `app/core/config.py`

```python
# Good ✓
async def predict_failure(
    machine_id: str,
    sensor_data: list[SensorReading],
    horizon_hours: int = 24,
) -> FailurePrediction:
    """
    Predict failure probability for a machine.

    Args:
        machine_id: Unique asset identifier.
        sensor_data: List of recent sensor readings (chronological).
        horizon_hours: Prediction horizon in hours.

    Returns:
        FailurePrediction with probability, failure modes, and confidence.

    Raises:
        InsufficientDataError: If fewer than 60 readings provided.
        ModelNotReadyError: If model is still warming up.
    """

# Bad ✗
def predict(id, data):  # No types, no docstring
    print("predicting")  # Use structlog
    return model.predict(data)  # No error handling
```

### TypeScript (Frontend)

- **Style**: ESLint + Prettier (2-space indent)
- **Types**: No `any` — use proper types or generics
- **Components**: Functional components with hooks only
- **State**: Zustand for global, `useState` for local
- **Data fetching**: React Query (`@tanstack/react-query`)
- **Naming**: PascalCase components, camelCase functions/variables

### SQL / Database

- All schema changes via Alembic migrations (never raw ALTER TABLE in prod)
- Index all foreign keys and timestamp columns
- Use TimescaleDB hypertables for any time-series table
- Add appropriate compression policies for historical data

---

## ML Model Contributions

Contributing a new model or improving an existing one requires additional steps:

### 1. Research Brief (required for new models)

Open an issue with:
- Paper citation (if applicable)
- Expected improvement over baseline (with metric)
- Dataset used for validation
- Computational requirements

### 2. Implementation Requirements

```
ml/models/your_model/
├── __init__.py
├── model.py          # Model architecture (PyTorch or sklearn)
├── trainer.py        # Training loop with MLflow logging
├── config.py         # Dataclass config with all hyperparameters
└── README.md         # Model card with architecture, benchmarks
```

### 3. Model Card

Every new model must include a `README.md` with:
- Architecture diagram/description
- Hyperparameters and their effects
- Training data requirements
- Benchmark results on standard datasets (C-MAPSS FD001, etc.)
- Known limitations
- Inference latency on CPU/GPU

### 4. Benchmark Standards

New models must meet or exceed these baselines on C-MAPSS FD001:

| Metric | Minimum Threshold |
|--------|-------------------|
| RMSE (RUL) | < 15.0 |
| MAE (RUL) | < 11.0 |
| Score (NASA) | < 400 |
| Inference P99 | < 100ms CPU |

### 5. Integration

After standalone validation, integrate via:
```python
# Register in ml/models/__init__.py
# Add to InferenceService model loading
# Add configuration to ml/configs/production.yaml
# Update API schema if new outputs
```

---

## Testing Requirements

### Coverage Targets

| Component | Minimum Coverage |
|-----------|-----------------|
| Backend API | 95% |
| ML Models | 90% |
| Services | 90% |
| Frontend | 80% |

### Test Categories

**Unit Tests** (`tests/unit/`):
- Test individual functions/classes in isolation
- Mock all external dependencies (DB, Redis, Kafka, models)
- Must be fast (< 1s each)

**Integration Tests** (`tests/integration/`):
- Test service interactions with real dependencies (test containers)
- Database migrations, Kafka producers/consumers
- Model loading and inference pipeline

**API Tests** (`tests/api/`):
- Full HTTP request/response testing via FastAPI TestClient
- Auth flows, rate limiting, error responses
- OpenAPI schema validation

**ML Tests** (`tests/unit/test_ml_models.py`):
- Shape correctness for all inputs/outputs
- Gradient flow (no dead parameters)
- Numerical stability (NaN/Inf checks)
- Determinism in eval mode
- Performance SLOs (latency, accuracy)

**Performance Tests** (`tests/performance/`):
- k6 load tests for API endpoints
- Throughput: ≥ 1000 req/s on /api/v1/predictions/health-score
- P99 latency: < 100ms

### Writing Good Tests

```python
# Good test: descriptive, focused, AAA pattern
def test_ensemble_predicts_higher_probability_for_faulty_machine():
    """Ensemble should assign higher failure prob to machines with fault signatures."""
    # Arrange
    normal_features = create_normal_sensor_features(n_samples=100)
    faulty_features = create_faulty_sensor_features(n_samples=100, fault_type="bearing")
    ensemble = load_trained_ensemble()

    # Act
    normal_proba = ensemble.predict_proba(normal_features)[:, 1]
    faulty_proba = ensemble.predict_proba(faulty_features)[:, 1]

    # Assert
    assert faulty_proba.mean() > normal_proba.mean(), (
        f"Faulty machines should have higher failure probability. "
        f"Got normal={normal_proba.mean():.3f}, faulty={faulty_proba.mean():.3f}"
    )
    assert faulty_proba.mean() > 0.5, "Faulty machines should be majority positive"
```

---

## Documentation

### What Needs Documentation

- All public functions/classes: docstrings
- New API endpoints: OpenAPI description in the router decorator
- New ML models: model card README.md
- Architecture changes: update `docs/ARCHITECTURE.md`
- New configuration options: update `.env.example` with comments
- New datasets: update the datasets table in main README.md

### Research Notes

For significant algorithmic contributions, add a Jupyter notebook in `notebooks/research/` explaining:
- The research question
- Experimental setup
- Results with visualizations
- Conclusions and recommendations

---

## Review Process

### What Reviewers Check

**All PRs:**
- [ ] Tests pass in CI
- [ ] Coverage meets threshold
- [ ] No linting errors
- [ ] Type checking passes
- [ ] Commit messages follow convention
- [ ] PR description is complete

**ML PRs additionally:**
- [ ] Model card present
- [ ] Benchmark results documented
- [ ] Inference latency measured
- [ ] No data leakage in validation
- [ ] Uncertainty quantification included

### Review Timeline

- **Bug fixes**: 1–2 business days
- **Features**: 3–5 business days
- **ML models**: 5–10 business days (includes benchmark verification)
- **Research contributions**: 2–4 weeks (full peer review)

### Becoming a Reviewer

After 5+ accepted PRs, you can apply to become a reviewer by opening an issue tagged `[reviewer-application]`.

---

## Questions?

- 💬 **Discussions**: [GitHub Discussions](https://github.com/your-org/predictive-maintenance-ai-platform/discussions)
- 🐛 **Bugs**: [Issue tracker](https://github.com/your-org/predictive-maintenance-ai-platform/issues)
- 📧 **Security**: security@predictiq.ai (see [SECURITY.md](SECURITY.md))
- 🔬 **Research**: research@predictiq.ai

We look forward to your contributions!
