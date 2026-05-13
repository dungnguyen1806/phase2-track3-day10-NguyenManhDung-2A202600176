# Lab Implementation Plan — Reliability Engineering for Production Agents

This plan outlines the steps to complete the Day 10 Lab, focusing on building a reliability layer for an LLM agent gateway.

## Phase 1: Setup and Orientation
- [x] Create virtual environment and install dependencies.
- [x] Run baseline tests (`make test`) and identify xfailing/skipped tests.
- [x] Run baseline chaos simulation (`make run-chaos`) and inspect `reports/metrics.json`.
- [x] Review TODOs in the source code.

## Phase 2: Circuit Breaker + Fallback Routing
### Circuit Breaker (`circuit_breaker.py`)
- [ ] Implement `allow_request()`: Handle OPEN state and HALF_OPEN transition after `reset_timeout`.
- [ ] Implement `record_success()`: Reset failure count and handle HALF_OPEN -> CLOSED transition.
- [ ] Implement `record_failure()`: Increment failure count and handle CLOSED -> OPEN / HALF_OPEN -> OPEN transitions.

### Gateway Routing (`gateway.py`)
- [ ] Improve route reasons to be more descriptive (e.g., `"primary:gpt4"`, `"fallback:backup_provider"`, `"cache_hit:0.95"`).
- [ ] Add timing around the `complete()` method to capture total routing overhead.
- [ ] Ensure `CircuitOpenError` is handled and triggers fallback.

**Verification:**
- Run `make test` to ensure `test_gateway_contract.py` passes.
- Verify circuit transitions in `transition_log`.

## Phase 3: Metrics + Chaos Scenarios
### Chaos Simulation (`chaos.py`)
- [ ] Add at least 3 distinct named scenarios:
    - `primary_timeout_100`: Force all primary requests to fail.
    - `primary_flaky_50`: 50% failure rate for primary.
    - `cache_stale_candidate`: Test for false cache hits with low threshold.
- [ ] Implement `recovery_time_ms` calculation from `transition_log`.
- [ ] Define pass/fail criteria for each scenario.

### Metrics (`metrics.py`)
- [ ] Ensure all required fields in `RunMetrics` are populated correctly.

**Verification:**
- Run `make run-chaos` and verify `metrics.json` contains all required fields and scenarios.

## Phase 4: In-memory Cache + Tuning
### Cache Improvements (`cache.py`)
- [ ] Improve `similarity()`: Implement a better metric than Jaccard (e.g., TF-IDF, character n-gram, or vector similarity).
- [ ] Add false-hit guardrails:
    - Privacy-sensitive keyword check.
    - Risk label check (from sample queries).
- [ ] Tune `similarity_threshold` to pass `test_todo_requirements.py`.

**Verification:**
- Run `pytest tests/test_todo_requirements.py` to ensure it passes.
- Compare metrics with and without cache.

## Phase 5: Redis Shared Cache
- [ ] Start Redis using `make docker-up`.
- [ ] Implement `SharedRedisCache.set()`: Store query/response as Redis Hash with TTL.
- [ ] Implement `SharedRedisCache.get()`: 
    - Exact match lookup.
    - Similarity scan for near-matches.
    - Apply privacy and false-hit guardrails.
- [ ] Ensure graceful degradation if Redis is down.

**Verification:**
- Run `make test` to ensure all Redis tests in `tests/test_redis_cache.py` pass.
- Verify shared state by using two cache instances.

## Phase 6: Load test + Final Report
- [ ] Increase `load_test.requests` in `configs/default.yaml`.
- [ ] (Stretch) Implement concurrency in `run_simulation` using `ThreadPoolExecutor`.
- [ ] Generate the final report using `make report`.
- [ ] Complete `reports/final_report.md` with:
    - Architecture diagram.
    - Configuration table with justifications.
    - Metrics and Chaos scenario tables.
    - Cache comparison and Redis shared cache evidence.
    - Failure analysis and next steps.

## Final Submission Checklist
- [ ] All TODOs in `src/` completed.
- [ ] `make test` passes (0 failures).
- [ ] `make typecheck` and `make lint` pass.
- [ ] `reports/metrics.json` and `reports/final_report.md` are complete.
- [ ] `docker-compose.yml` included.
