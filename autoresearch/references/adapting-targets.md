# Adapting autoresearch to Non-LLM Targets

autoresearch works for any optimization target where you can express "better" as a number from
a command. This guide shows how to adapt the framework for common use cases beyond LLM training.

---

## Table of Contents

1. [Pattern Overview](#1-pattern-overview)
2. [JavaScript Bundle Size](#2-javascript-bundle-size)
3. [Test Suite Speed](#3-test-suite-speed)
4. [Web Performance / Lighthouse](#4-web-performance--lighthouse)
5. [API Latency](#5-api-latency)
6. [Compiler / Build Time](#6-compiler--build-time)
7. [General Checklist](#7-general-checklist)

---

## 1. Pattern Overview

For any target, you need four things:

| Component | LLM Training | General |
|-----------|-------------|---------|
| **Target file** | `train.py` | Your config, source, or build file |
| **Frozen eval** | `evaluate_bpb()` | Your benchmark script or command |
| **Metric** | `val_bpb` (lower = better) | Any scalar number |
| **Budget** | 5-min time window | Fixed iterations, fixed dataset, fixed test suite |

The budget concept is critical: the benchmark must be **fixed** so every experiment is comparable.
If you benchmark test speed but add tests between experiments, results are incomparable.

---

## 2. JavaScript Bundle Size

**Goal**: Minimize final bundle size in bytes (lower = better).

**Target file**: `vite.config.ts`, `webpack.config.js`, `rollup.config.ts`, or `tsconfig.json`

**Benchmark command**:
```bash
bun run build 2>&1
du -sb dist/ | awk '{print "bundle_bytes: "$1}'
```

Or for specific files:
```bash
bun run build 2>&1
stat -f "%z" dist/index.js | awk '{print "bundle_bytes: "$1}'
# On Linux: stat -c "%s" dist/index.js
```

**What can be changed**:
- Bundler config: minification options, tree-shaking settings, chunk splitting strategy
- `tsconfig.json` target and lib settings
- Import paths (barrel file elimination, direct imports)
- Dynamic imports to split code
- Dependency substitutions (lighter alternatives for heavy deps)

**What cannot be changed**:
- The source application code (functional behavior must be preserved)
- The list of features/exports (public API surface)
- The build output directory structure (if downstream depends on it)

**Correctness check**: run your test suite after each improvement:
```bash
#!/bin/bash
# checks.sh
bun test
```

**Good research directions**:
- Replace `lodash` with `lodash-es` for better tree-shaking
- Replace `moment` with `date-fns` (tree-shakeable)
- Convert barrel re-exports to direct imports
- Enable `moduleResolution: "bundler"` in tsconfig
- Split vendor chunk vs application chunk
- Try `sideEffects: false` in package.json dependencies
- Enable Bun's `--minify` and `--minify-syntax` flags
- Inline small assets vs reference them as separate files
- Try different chunk size strategies (single bundle vs granular chunks)

---

## 3. Test Suite Speed

**Goal**: Minimize total test suite wall-clock time (lower = better).

**Target files**: test runner config (`vitest.config.ts`, `bun test` flags via script),
test utilities, setup files, or mock implementations.

**Benchmark command**:
```bash
# Run N times and take median to reduce noise
for i in 1 2 3; do
  { time bun test 2>&1; } 2>&1 | grep real
done
```

Or simpler with hyperfine:
```bash
hyperfine --runs 3 'bun test' --export-json bench.json
jq '.results[0].median * 1000 | floor | "test_ms: \(.)"' bench.json
```

**What can be changed**:
- Test runner configuration (parallelism, worker count, timeout settings)
- Test setup/teardown (in-memory db vs real db, mock strategies)
- Shared fixtures and setup hooks
- Import structure in test files (avoid expensive imports in hot paths)

**What cannot be changed**:
- Test assertions and correctness (no tests may be removed or weakened)
- Test coverage (all features must remain tested)
- The application source code

**Good research directions**:
- Increase worker concurrency in vitest/bun test config
- Replace filesystem fixtures with in-memory equivalents
- Move expensive `beforeAll` setup to module-level singletons
- Use `--bail` to fail fast in CI (not for benchmarking, but for config exploration)
- Mock expensive external calls (database, API) with faster in-memory mocks
- Reduce unnecessary `await` in async test chains

---

## 4. Web Performance / Lighthouse

**Goal**: Maximize Lighthouse performance score (higher = better) or minimize specific metrics
(LCP, TBT, CLS).

**Target files**: HTML template, CSS entry point, component files, server-side rendering config.

**Benchmark command** (requires Puppeteer / Lighthouse CLI):
```bash
# Start server in background
bun run start &
SERVER_PID=$!
sleep 2

# Run Lighthouse
npx lighthouse http://localhost:3000 \
  --output json \
  --output-path lighthouse.json \
  --chrome-flags="--headless --no-sandbox" \
  --quiet 2>/dev/null

kill $SERVER_PID

# Extract score
jq '.categories.performance.score * 100 | floor | "lighthouse_score: \(.)"' lighthouse.json
```

**What can be changed**:
- Asset loading strategies (lazy loading, preload hints, defer/async)
- Critical CSS extraction and inlining
- Image optimization settings
- Font loading strategy (font-display, preconnect)
- JavaScript splitting and deferred execution
- HTTP caching headers configuration
- Compression settings

**What cannot be changed**:
- Page content and functionality
- Visual design (no removing images or content to cheat the score)
- Accessibility (ARIA labels, semantic HTML)

**Good research directions**:
- Add `<link rel="preload">` for LCP image
- Inline critical CSS, defer non-critical
- Replace render-blocking scripts with `defer`
- Add `font-display: swap` to @font-face rules
- Implement responsive images with `srcset`
- Add `loading="lazy"` to below-fold images
- Enable gzip/brotli compression
- Minimize third-party script impact (defer analytics)

---

## 5. API Latency

**Goal**: Minimize p50 or p95 response latency in milliseconds (lower = better).

**Target files**: route handler, middleware stack, database query files, cache layer.

**Benchmark command**:
```bash
# Start server
bun run start &
SERVER_PID=$!
sleep 1

# Run k6 or wrk benchmark (fixed concurrency, fixed duration)
k6 run --duration 10s --vus 10 --summary-export bench.json bench.js 2>/dev/null

kill $SERVER_PID

# Extract p95
jq '.metrics.http_req_duration.values["p(95)"] | "p95_ms: \(. * 1000 | floor)"' bench.json
```

Or with `wrk`:
```bash
wrk -t4 -c10 -d10s http://localhost:3000/api/endpoint 2>&1 | \
  grep "Latency" | awk '{print "p95_ms: " $3}'
```

**What can be changed**:
- Database query optimization (indexes, query structure, N+1 elimination)
- Caching layer (add Redis, in-memory LRU, HTTP cache headers)
- Middleware order and selection (remove unnecessary middleware)
- Response serialization (JSON stringify optimization)
- Connection pooling settings

**What cannot be changed**:
- API response shape (consumers depend on it)
- Authentication/authorization logic
- Data correctness

**Good research directions**:
- Add database indexes for frequently-queried columns
- Implement response caching for read-heavy endpoints
- Replace `JSON.stringify` with faster alternatives (`fast-json-stringify`)
- Reduce middleware chain (remove unused middleware)
- Use `SELECT` column lists instead of `SELECT *`
- Batch N+1 queries with DataLoader pattern
- Move expensive computation to background jobs

---

## 6. Compiler / Build Time

**Goal**: Minimize full build time in milliseconds (lower = better).

**Target files**: Build config (`tsconfig.json`, build scripts), source file organization.

**Benchmark command**:
```bash
# Cold build (no cache)
rm -rf dist/ .tsbuildinfo
{ time bun run build 2>&1; } 2>&1 | grep real | awk '{print "build_ms: " $2}'
```

**What can be changed**:
- `tsconfig.json` compiler options (target, lib, skipLibCheck, incremental)
- Build script flags (parallelism, workers)
- Module boundaries (barrel file structure affects TypeScript resolution)
- `isolatedModules` and similar settings

**What cannot be changed**:
- Source code logic
- Output format requirements
- Type safety (cannot weaken type checking to cheat build time)

---

## 7. General Checklist

When adapting autoresearch to any new target:

- [ ] Define the **single primary metric** as a number from a command (not subjective)
- [ ] Ensure the benchmark is **deterministic enough** (run 3x, median should be stable ±5%)
- [ ] Define **what is frozen** explicitly before writing program.md
- [ ] Create a `checks.sh` if correctness must be verified (tests, lint, type check)
- [ ] Run the baseline manually first — confirm you can parse the metric
- [ ] Add the metric parse command to `.gitignore`-excluded `results.tsv`
- [ ] Write 15–30 research directions before starting the loop
- [ ] Set the simplicity threshold relative to baseline (e.g., "must improve by > 0.5%")

### Noise considerations

For fast benchmarks (< 1 second), add statistical stabilization:
- Run 3–5 times and take the median
- Use `hyperfine` for reliable benchmarking
- Warm up the system before timing (one throwaway run)

For slow benchmarks (> 5 minutes), consider:
- Reducing the benchmark scope (fewer samples, smaller dataset) while keeping it representative
- Using a proxy metric that correlates with the real metric (faster to evaluate)
