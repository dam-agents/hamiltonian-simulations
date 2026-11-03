# Hamiltonian Simulations on IBM Quantum hardware

A DAM starter kit for running Hamiltonian simulation workflows on IBM Quantum hardware.

This repository is only the kit definition: [`kit.yaml`](kit.yaml) and
[`ONBOARDING.md`](ONBOARDING.md). The working code, scripts, and skills live in the LAHS GitHub
Enterprise repositories:

- [`lahs-agentic-workflows`](https://github.ibm.com/LAHS/lahs-agentic-workflows)
- [`lahs-workflows`](https://github.ibm.com/LAHS/lahs-workflows)

## What it sets up

In its first session, the agent:

1. clones `lahs-agentic-workflows` into `~/work/lahs-agentic-workflows`;
2. runs `~/work/lahs-agentic-workflows/bin/dam-setup setup`, which updates that checkout, clones
   `lahs-workflows` into `~/work/lahs-workflows`, and installs the minimal tooling environment;
3. checks GitHub Enterprise access;
4. guides you through saving your IBM Quantum account (Qiskit Functions CRN and API key) using
   the session's terminal;
5. confirms that the `lahs-workflows` Qiskit Functions are deployed.

The kit installs these skills:

- `connect-cluster`, `preflight`, `run-workflow`, `report-run` from `lahs-agentic-workflows`;
- `run-lahs-workflow`, `run-qiskit-function`, `analyse-sweep` from `lahs-workflows`.

## Before you start

- Access to the LAHS repositories on `github.ibm.com`.
- An IBM Quantum account with a Qiskit Functions instance (its CRN and an IBM Cloud API key).
- The `lahs-workflows` (CPU) and `lahs-workflows-gpu` (GPU) Qiskit Functions deployed. If they are
  not deployed, onboarding explains how to deploy them from a `lahs-workflows` checkout.

The API key is typed into a hidden prompt in the agent's terminal. Never put an API key,
CRN-plus-key pair, or cluster credentials in this repository.

## Use it

Create an agent from the **Hamiltonian Simulations** kit (`lahs-workflows-runner`) and follow its first-session onboarding.

This kit is public metadata only. It contains no pipeline code, scripts, skills, cluster
configuration, or credentials.
