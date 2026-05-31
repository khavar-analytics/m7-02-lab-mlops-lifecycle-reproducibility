# ADR 0001: Reproducibility strategy for NorthStar models

## Context
Currently, NorthStar Logistics operates without a model registry, dataset versioning, or consistent artifact tracking. Models are manually stored in S3 using ad-hoc naming conventions (e.g., `eta_v2_FINAL.onnx`), making it impossible to reliably reproduce past experiments or debug production issues.

## Decision

To guarantee deterministic reproducibility, we enforce strict controls across four layers:

### Environment Layer
All workloads run inside Docker containers with explicitly pinned base images (e.g., `python:3.10-slim`). Dependencies are locked using `pip-tools` with fully resolved hashes. Floating versions and `latest` tags are strictly forbidden.

### Data Layer
Training data is versioned via immutable snapshots from the Data Lake (e.g., Iceberg or Delta tables). Each run computes a SHA-256 `dataset_hash` tied to the exact snapshot and logs it in the Model Registry.

### Code Layer
Every training run must originate from a clean Git state. The CI pipeline captures the exact `git_sha` and attaches it to the model artifact. Runs with uncommitted changes are rejected.

### Randomness Layer
All stochastic components are explicitly controlled:
- Global seed (`SEED = 42`)
- NumPy seed (`numpy.random.seed`)
- Framework seed (`torch.manual_seed`)
- Deterministic execution flags (e.g., `torch.backends.cudnn.deterministic = True`)

## Alternatives rejected
- **S3 timestamps for versioning:** Unreliable due to overwrites and lack of row-level guarantees.
- **Conda environments (`environment.yml`):** Non-deterministic dependency resolution leads to inconsistent builds.
- **No randomness control:** Leads to irreproducible metrics and unstable evaluation.

## Consequences
- **Good:** Full reproducibility — any model can be rebuilt with identical outputs.
- **Good:** Strong debugging and audit capabilities across the team.
- **Bad:** Increased overhead in managing dataset snapshots and dependency locks.
- **Bad:** Higher storage costs due to immutable data retention.

## Revisit if
Revisit if the system transitions to a fully streaming architecture where batch snapshotting becomes infeasible, or if compliance requirements demand adoption of a full-featured feature store (e.g., Feast).