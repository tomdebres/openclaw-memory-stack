# Scripts

This directory documents the expected CLI/script surface for the OpenClaw Memory Stack.

The stack described by this repo is layered:

- **memory-core**: local typed memory for preferences, facts, history, and metrics
- **document memory**: indexing and retrieval over documents, typically backed by PostgreSQL + pgvector
- **conversation memory**: raw sessions plus embeddings and summaries
- **health/ops**: freshness, drift, coverage, and alerting helpers

The scripts here are reference-facing placeholders, not the full production implementation.

## Generic repo vs. real deployments

This public repo describes the generic pattern.

A concrete deployment such as TomOS may add:

- host-specific paths
- LaunchAgent or cron wrappers
- local session storage conventions
- environment-specific alert routing
- deployment-specific helper scripts

Use this directory as an interface guide, not as proof that every runtime detail is implemented exactly here.
