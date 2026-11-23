# Hamiltonian Simulations on IBM Quantum hardware

A DAM starter kit for running Hamiltonian simulation workflows on IBM Quantum hardware.

This repository is only the kit definition: [`kit.yaml`](kit.yaml) and
[`ONBOARDING.md`](ONBOARDING.md). The Hamiltonian Simulations knowledge bases are provided by the
GitHub Enterprise repositories:

- [`lahs-agentic-workflows`](https://github.ibm.com/LAHS/lahs-agentic-workflows) — `connect-cluster`, `preflight`, `run-workflow`, `report-run`, `agent-run-report`
- [`lahs-workflows`](https://github.ibm.com/LAHS/lahs-workflows) — `run-lahs-workflow`, `run-managed-workflow`, `analyse-sweep`, `run-research-study`, `build-surface-model`, `report-agent-issue`, `request-skill`

## What it sets up

In its first session, the agent:

1. clones `lahs-agentic-workflows` into `~/work/lahs-agentic-workflows`;
2. runs `~/work/lahs-agentic-workflows/bin/dam-setup setup`, which updates that checkout, clones
   `lahs-workflows` into `~/work/lahs-workflows`, and installs the minimal tooling environment;
3. checks access — GitHub Enterprise *or* the read-only COS mirror (`s3://hs-distributions-bucket/hamiltonian-simulations-kit/` on `s3.us-south.cloud-object-storage.appdomain.cloud`);
4. installs all twelve skills from both sources when they exist; if a source is missing, it notes
   that the skills are still readable from the cloned checkouts and adding the source is optional;
5. guides you through saving your IBM Quantum account (managed workflows instance CRN and API key) using
   the session's terminal;
6. confirms that the `lahs-workflows` IBM Quantum managed workflows are deployed.

The kit declares these skills:

- `connect-cluster`, `preflight`, `run-workflow`, `report-run`, `agent-run-report` from
  `lahs-agentic-workflows`;
- `run-lahs-workflow`, `run-managed-workflow`, `analyse-sweep`, `run-research-study`,
  `build-surface-model`, `report-agent-issue`, `request-skill` from `lahs-workflows`.

If the platform did not install them at creation, onboarding installs them from the two sources;
if a source is missing, the same skills remain readable from the cloned checkouts, so adding the
source is optional.

## Before you start

- Access to the Hamiltonian Simulations knowledge bases — either via `github.ibm.com` (GitHub Enterprise) *or* the read-only COS mirror `hs-distributions-bucket/hamiltonian-simulations-kit` on `s3.us-south.cloud-object-storage.appdomain.cloud`.
- An IBM Quantum account with a managed workflows instance (its CRN and an IBM Cloud API key).
- The `lahs-workflows` (CPU) and `lahs-workflows-gpu` (GPU) IBM Quantum managed workflows deployed. If they are
  not deployed, onboarding explains how to deploy them from a `lahs-workflows` checkout.

The API key is typed into a hidden prompt in the agent's terminal. Never put an API key,
CRN-plus-key pair, or cluster credentials in this repository.

## Use it

Create an agent from the **Hamiltonian Simulations** kit (`hs-workflows-runner`) and follow its first-session onboarding.

This kit is public metadata only. It contains no pipeline code, scripts, skills, cluster
configuration, or credentials.
