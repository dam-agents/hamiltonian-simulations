# Onboarding

The first session of an agent created from [`kit.yaml`](kit.yaml) follows this file to set the
agent up to run lahs-workflows. The public kit seeds only its `kit.yaml` and this `ONBOARDING.md`
into the agent's home directory; the working `lahs-agentic-workflows` checkout is cloned into
`~/work/lahs-agentic-workflows`, and `lahs-workflows` into `~/work/lahs-workflows`. Commands below
run from `~/work/lahs-agentic-workflows` after step 1.

> **Nomenclature:**
> * `LAHS` is the GitHub Enterprise organization (`LAHS/lahs-agentic-workflows`,
>   `LAHS/lahs-workflows`). In user-facing text these two repositories are the **Hamiltonian Simulations
>   knowledge bases** — same code, same URLs, just the branded name. Treat “LAHS repositories”,
>   “Hamiltonian Simulations repos”, and “knowledge bases” as synonyms.
> * **Samples vs shots:** In user-facing text say **samples** (e.g. “samples per circuit”, “total samples”);
>   internally the library, `quantum_circuit_run.shots`, and QPU budgets still use `shot`/`shots`.
>   Never expose “shots” to users.
> * **IBM Quantum managed workflows, not “Qiskit Functions”:** workflows run on the deployed
>   `lahs-workflows` or `lahs-workflows-gpu` IBM Quantum managed workflows; never call the offering
>   “Qiskit Functions” (that now names the catalog service) or “formerly …”. A cluster satellite is
>   the secondary alternative (`connect-cluster` skill).

Follow these steps in order. If a step fails, resolve it before continuing. Never ask the user to
share a token, API key or CRN-plus-key pair in chat, and never print or enter a key yourself.

Call `set_onboarding_checklist` with five steps:
- id `setup`, label "Clone lahs-agentic-workflows into ~/work and update both repos (bin/dam-setup setup)";
- id `github`, label "Grant this agent a GitHub Enterprise connection with read access to both Hamiltonian Simulations
  knowledge bases on github.ibm.com (or the Object Storage mirror)";
- id `skills`, label "Set up the twelve skills from both Hamiltonian Simulations knowledge bases";
- id `account`, label "Save your IBM Quantum account (managed workflows instance CRN and API key) in the
  session's terminal";
- id `function`, label "Confirm the deployed IBM Quantum managed workflows (CPU and GPU)".

1. **Clone and update the GHE checkout (first).** The public kit seed cannot contain the GHE
   checkout, so clone it and run its setup. `dam-setup` tries GitHub first, then the COS mirror.

   ```bash
   [ -d ~/work/lahs-agentic-workflows/.git ] \
       || git clone https://github.ibm.com/LAHS/lahs-agentic-workflows.git ~/work/lahs-agentic-workflows
   bash ~/work/lahs-agentic-workflows/bin/dam-setup setup
   ```

   `dam-setup` pulls `~/work/lahs-agentic-workflows` and `~/work/lahs-workflows` and installs the
   minimal tooling env. It tries `git fetch`/`clone` from `github.ibm.com`; if that fails it falls
   back to the read-only COS mirror (`s3://hs-distributions-bucket/hamiltonian-simulations-kit/`,
   `aws s3 cp` + the granted Object Storage profile auto-discovered by endpoint — name can be anything).
   The manifest is saved persistently to `~/work/hamiltonian-simulations-kit-manifest.json` (not `/tmp`,
   which is cleared on hibernate). In this sandbox use `uv run --no-project --with …` for tooling;
   never install the execution env (`uv sync --extra core`/`--all-extras`) or run the pipeline
   (`uv run workflow run/orchestrate/study run`) from `~/work/lahs-workflows`, because it pulls
   the CUDA/torch/jax env (~2 GB) and can fill the home disk. A plain `uv sync` is the ~0.7 GB
   planning env. Setup also checks the AWS CLI profiles for the code, issues and reports buckets.
   The code and issues buckets are optional while `github.ibm.com` works; without the reports
   bucket, usage reports stay local. Warnings do not fail setup. Successful
   setup prints `dam-setup: ready`. If the clone or setup fails because neither github.ibm.com nor
   the COS mirror is reachable, do step 2, grant a GitHub Enterprise *or* Object Storage connection,
   and re-run step 1. If setup fails for another reason, report the error without exposing secrets
   and stop. When it prints `dam-setup: ready`, call `complete_onboarding_step` for `setup`.

2. **GitHub or COS access.** The agent needs *one* of:
   - **GitHub Enterprise** (preferred when available): read on both `LAHS/lahs-agentic-workflows`
     and `LAHS/lahs-workflows` on `github.ibm.com`, or
   - **Object Storage** mirror: read on `hs-distributions-bucket/hamiltonian-simulations-kit`
     on `https://s3.us-south.cloud-object-storage.appdomain.cloud` (any connection name; `dam-setup`
     finds it by endpoint via `aws s3 cp`, the agent never writes there).

   Check what is granted:

   ```bash
   git ls-remote https://github.ibm.com/LAHS/lahs-agentic-workflows.git HEAD
   git ls-remote https://github.ibm.com/LAHS/lahs-workflows.git HEAD
   ```

   Each command must print a commit hash *or* `~/work/hamiltonian-simulations-kit-manifest.json` must
   exist from a COS fallback (`cat ~/work/hamiltonian-simulations-kit-manifest.json`). If both
   `git ls-remote` fail and no manifest exists, name which path failed. For GitHub, ask the user to
   grant a GitHub Enterprise connection with read access to both repositories under agent →
   Connections; for COS, ask them to grant an Object Storage connection with endpoint
   `https://s3.us-south.cloud-object-storage.appdomain.cloud` and bucket `hs-distributions-bucket`
   (read-only). If DAM requests network approval, ask the user to approve `github.ibm.com` or
   `s3.us-south.cloud-object-storage.appdomain.cloud` in DAM's Activity. Wait for the user, then repeat both checks and re-run step 1 if the
   clone failed. Continue only after at least one path passes; do not bypass a failed check. When
   either passes, call `complete_onboarding_step` for `github`.

   For automatic usage JSON uploads, also grant a read/write Object Storage connection to
   `lahs-agentic-sessions-data-bucket` using the bucket's actual endpoint. If unavailable, keep
   reports locally and mark uploads pending; the read-only mirror grant does not cover reporting.
   Each `usage_statistics_*.json` stores its workflow's `managed_workflows_ids` and IBM Quantum
   Platform `job_ids`; the session folder's files cover the session so the run archives and
   circuits can be fetched later, only when the user asks.

3. **Set up the skills.** The kit's `skills:` entries may install only from sources that already
   exist. Check your available skill sources and the cloned checkouts first.
   - If both repository sources exist — `https://github.ibm.com/LAHS/lahs-agentic-workflows` and
     `https://github.ibm.com/LAHS/lahs-workflows` — install the twelve skills from them with your
     skill tools; skip any already present.
   - If a source exists but a listed skill is missing, refresh the source first (`dam skill source
     refresh <url>` drops its scan cache; the next catalog re-scans upstream), then install it.
   - If a source is missing, do not block onboarding: the same skills are already readable from
     the checkouts at `~/work/lahs-agentic-workflows/skills/` and `~/work/lahs-workflows/skills/`.
     Acknowledge that the source is missing, and mention that adding it in DAM's
     **Skills → Add source** is optional (the repository URL is the source; both expose `skills/`
     at the repository root). The user may also run `dam skill source add <url>` from their
     machine.
   The twelve skills are:
   - `lahs-agentic-workflows`: `connect-cluster`, `preflight`, `run-workflow`, `report-run`,
     `agent-run-report`;
   - `lahs-workflows`: `run-lahs-workflow`, `run-managed-workflow`, `analyse-sweep`,
     `run-research-study`, `build-surface-model`, `report-agent-issue`, `request-skill`.

   If your harness has no skill tools, read the `SKILL.md` files directly from the checkouts; that
   is sufficient. Call `complete_onboarding_step` for `skills` when the skills are installed or
   available from the cloned checkouts; otherwise report which are missing and leave it open.

4. **IBM Quantum managed workflows.** Ask the user whether the `lahs-workflows` deployments are
   running on their managed workflows instance, and ask them to confirm the titles: `lahs-workflows`
   (CPU) and `lahs-workflows-gpu` (GPU) by default.
   - **Deployed:** note the titles they confirm and continue to step 5.
   - **Not deployed:** tell the user to clone https://github.ibm.com/LAHS/lahs-workflows.git on
     their laptop and ask an agent there to "deploy the lahs-workflows IBM Quantum managed workflows
     for CPU and GPU" (the `run-managed-workflow` skill builds and deploys both). Do not build or deploy
     from this sandbox. Continue with step 5; workflows can run once the deployments are live.

5. **IBM Quantum account.** The agent reaches IBM Quantum managed workflows with the user's own
   account: the managed workflows instance **CRN** and an IBM Cloud **API key**. The user types the
   key into a hidden prompt; it never goes through the chat. Never put the API key in `kit.yaml`, an agent
   environment variable, or chat: the kit is public and the running agent can read its environment.
   The hidden prompt is the only supported path. Tell the user, step by step:

   a. Get an API key at https://cloud.ibm.com/iam/apikeys → **Create** (keep it ready to paste),
      and the CRN of the IBM Quantum instance that holds IBM Quantum managed workflows (IBM Quantum
      platform → Instances; the managed workflows instance, not a runtime-only one).

   b. Ask the user to run the command in a terminal that can show the hidden prompt. Give both
      routes:
      - **In DAM (no local terminal):** open the agent, start a new session with no messages, and
        choose **Terminal (browser)** beside the message box. Run the command there. If the hidden
        prompt does not appear, fall back to the local route below.
      - **From their machine:** run `dam ssh connect <this agent's name>` for a plain shell.
        Replace the placeholder with this agent's own name — the sandbox exports it as
        `$PLATFORM_AGENT_NAME` (run `echo $PLATFORM_AGENT_NAME`; if it is empty, `$HOSTNAME` is the
        agent ID), so give the user the literal command, ready to copy-paste. The UI's
        **VS Code / Zed (local)** shows the same command; **Terminal (local)** shows `dam chat`,
        which attaches the harness TUI rather than a plain shell.

      ```bash
      ~/work/lahs-agentic-workflows/bin/qiskit-account --name lahs-functions
      ```

      Paste the API key at the hidden prompt, then pick the managed workflows instance from the list (or
      pass `--instance <CRN>`). Wait until it prints `saved account`.

   c. Check it yourself: `~/work/lahs-agentic-workflows/bin/qiskit-account check` must list
      `lahs-functions` with its CRN (not
      `can not connect`). A TLS EOF,
      `InvalidAccountError` or timeout means a blocked host: ask the user to approve it in DAM's
      Activity, then retry. When it passes, call `complete_onboarding_step` for `account`.

   For the secondary satellite path, the account lives on the satellite machine, not in this
   sandbox: the user runs `bin/qiskit-account` there, from their own checkout of this repository.

6. **Check the deployed titles.** If the user said the deployments are live, check the titles
   they confirmed (replace them below if they differ from the defaults):

   ```bash
   uv run --no-project --with qiskit-ibm-catalog python - <<'EOF'
   from qiskit_ibm_catalog import QiskitServerless
   try:
       serverless = QiskitServerless(name="lahs-functions")
       for title in ("lahs-workflows", "lahs-workflows-gpu"):
           fn = serverless.load(title)
           print(title, "deployed" if fn else "missing", fn.sizes() if fn else "")
   except Exception as e:
       raise SystemExit(f"check failed before reading the titles: {type(e).__name__}: {e}")
   EOF
   ```

   Report which titles are deployed and their sizes. If the command exits with `check failed`
   (for example a `QiskitServerlessException` with `503`, a TLS error, or a timeout), the check
   did not run: report the exact error and leave the `function` step open. Do not report the
   titles as missing, and do not treat the user's confirmation as a substitute for this check.
   A `503` from `qiskit-serverless.quantum.ibm.com` while `github.com` and `pypi.org` work is a
   DAM egress-gateway problem, not a missing network allow: tell the user and suggest a new
   session or a fresh agent. If a title prints `missing`, give the user the deploy instructions
   from step 4. Call `complete_onboarding_step` for `function` only when both titles load;
   otherwise leave it open and tell the user workflows can run once it is done.

7. **Done.** When the steps that could run are done, tell the user setup is complete and call
   `mark_onboarding_complete`. If the `function` check failed or IBM Quantum managed workflows are
   not deployed, say setup is complete for everything else and leave that step open; don't mark it
   done just to finish onboarding. Explain that workflows run on IBM Quantum managed workflows and
   that this agent's skills guide the next steps:
   `run-workflow` prepares and runs a pipeline, `preflight` checks access before a job,
   `report-run` publishes the science report, and `agent-run-report` always saves usage JSON and
   publishes the operator meta-report on request; `connect-cluster` sets up the secondary cluster path. From lahs-workflows come
   `run-lahs-workflow`, `run-managed-workflow` (builds and deploys both titles), `analyse-sweep`,
   `run-research-study` (multi-run studies), `build-surface-model` (slab/adsorbate geometries),
   `report-agent-issue` (files problems as GitHub issues) and `request-skill` (turns a missing
   procedure into a skill). If a skill is not listed, read
   its `SKILL.md` under `~/work/lahs-agentic-workflows/skills/` or `~/work/lahs-workflows/skills/`
   instead.
