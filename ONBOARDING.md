# Onboarding

The first session of an agent created from [`kit.yaml`](kit.yaml) follows this file to set the
agent up to run lahs-workflows. The public kit seeds only its `kit.yaml` and this `ONBOARDING.md`
into the agent's home directory; the working `lahs-agentic-workflows` checkout is cloned into
`~/work/lahs-agentic-workflows`, and `lahs-workflows` into `~/work/lahs-workflows`. Commands below
run from `~/work/lahs-agentic-workflows` after step 1. Workflows run on the deployed
`lahs-workflows` or `lahs-workflows-gpu` Qiskit Function on IBM Quantum; a cluster satellite is the
alternative (`connect-cluster` skill).

Follow these steps in order. If a step fails, resolve it before continuing. Never ask the user to
share a token, API key or CRN-plus-key pair in chat, and never print or enter a key yourself.

Call `set_onboarding_checklist` with five steps:
- id `setup`, label "Clone lahs-agentic-workflows into ~/work and update both repos (bin/dam-setup setup)";
- id `github`, label "Grant this agent a GitHub Enterprise connection with read access to both LAHS
  repositories on github.ibm.com";
- id `skills`, label "Install the seven skills from both LAHS repositories";
- id `account`, label "Save your IBM Quantum account (Qiskit Functions CRN and API key) in the
  session's terminal";
- id `function`, label "Confirm the deployed Qiskit Functions (CPU and GPU)".

1. **Clone and update the GHE checkout (first).** The public kit seed cannot contain the GHE
   checkout, so clone it and run its setup:

   ```bash
   [ -d ~/work/lahs-agentic-workflows/.git ] \
       || git clone https://github.ibm.com/LAHS/lahs-agentic-workflows.git ~/work/lahs-agentic-workflows
   bash ~/work/lahs-agentic-workflows/bin/dam-setup setup
   ```

   `dam-setup` pulls `~/work/lahs-agentic-workflows` and `~/work/lahs-workflows` and installs the
   minimal tooling env. Successful setup prints `dam-setup: ready`. If the clone or setup fails
   because github.ibm.com is unreachable, do step 2, grant the connection, and re-run step 1. If
   setup fails for another reason, report the error without exposing secrets and stop. When it
   prints `dam-setup: ready`, call `complete_onboarding_step` for `setup`.

2. **GitHub access.** Run both:

   ```bash
   git ls-remote https://github.ibm.com/LAHS/lahs-agentic-workflows.git HEAD
   git ls-remote https://github.ibm.com/LAHS/lahs-workflows.git HEAD
   ```

   Each command must print a commit hash. If either fails, name the repository whose access check
   failed. For a connection or permissions issue, ask the user to grant this agent a GitHub
   Enterprise connection with read access to both repositories under agent → Connections. If DAM
   requests network approval, ask the user to approve github.ibm.com in DAM's Activity. Wait for
   the user, then repeat both checks and re-run step 1 if the clone failed. Continue only after
   both checks pass; do not bypass a failed check. When both pass, call
   `complete_onboarding_step` for `github`.

3. **Install the skills.** The kit's `skills:` entries may install only from sources that already
   exist; a user without them can get none of the seven. Check your available skill sources first.
   If `https://github.ibm.com/LAHS/lahs-agentic-workflows` is missing, or
   `https://github.ibm.com/LAHS/lahs-workflows/tree/main/.bob/skills` is missing, ask the user to
   add it in DAM's **Skills → Add source**; reuse a source that already points at the same URL
   rather than creating a duplicate. If the tree URL is rejected, add the repository URL with
   `.bob/skills` as the source path. The user may also run `dam skill source add <url>` from their
   machine. Then install all seven skills from those sources with your skill tools; skip any
   already present:
   - `lahs-agentic-workflows`: `connect-cluster`, `preflight`, `run-workflow`, `report-run`;
   - `lahs-workflows`: `run-lahs-workflow`, `run-qiskit-function`, `analyse-sweep`.

   If your harness has no skill tools, ask the user to turn the seven on in the Skills panel. When
   all seven are installed, call `complete_onboarding_step` for `skills`; otherwise report which
   are missing and leave it open.

4. **Qiskit Function.** Ask the user whether the `lahs-workflows` Qiskit Functions are deployed on
   their IBM Quantum Functions instance, and ask them to confirm the titles: `lahs-workflows` (CPU)
   and `lahs-workflows-gpu` (GPU) by default.
   - **Deployed:** note the titles they confirm and continue to step 5.
   - **Not deployed:** tell the user to clone https://github.ibm.com/LAHS/lahs-workflows.git on
     their laptop and ask an agent there to "deploy the lahs-workflows Qiskit Functions for CPU
     and GPU" (the `run-qiskit-function` skill builds and deploys both). Do not build or deploy
     from this sandbox. Continue with step 5; workflows can run once the functions are deployed.

5. **IBM Quantum account.** The agent reaches the Qiskit Function with the user's own account: the
   Qiskit Functions instance **CRN** and an IBM Cloud **API key**. The user types the key into a
   hidden prompt; it never goes through the chat. Never put the API key in `kit.yaml`, an agent
   environment variable, or chat: the kit is public and the running agent can read its environment.
   The hidden prompt is the only supported path. Tell the user, step by step:

   a. Get an API key at https://cloud.ibm.com/iam/apikeys → **Create** (keep it ready to paste),
      and the CRN of the IBM Quantum instance that holds the Qiskit Functions (IBM Quantum
      platform → Instances; the Functions instance, not a runtime-only one).

   b. Ask the user to run the command in a terminal that can show the hidden prompt. Give both
      routes:
      - **In DAM (no local terminal):** open the agent, start a new session with no messages, and
        choose **Terminal (browser)** beside the message box. Run the command there. If the hidden
        prompt does not appear, fall back to the local route below.
      - **From their machine:** run `dam ssh connect <agent name>` for a plain shell. The UI's
        **VS Code / Zed (local)** shows the same command; **Terminal (local)** shows `dam chat`,
        which attaches the harness TUI rather than a plain shell.

      ```bash
      ~/work/lahs-agentic-workflows/bin/qiskit-account --name lahs-functions
      ```

      Paste the API key at the hidden prompt, then pick the Functions instance from the list (or
      pass `--instance <CRN>`). Wait until it prints `saved account`.

   c. Check it yourself: `~/work/lahs-agentic-workflows/bin/qiskit-account check` must list
      `lahs-functions` with its CRN (not
      `can not connect`). A TLS EOF,
      `InvalidAccountError` or timeout means a blocked host: ask the user to approve it in DAM's
      Activity, then retry. When it passes, call `complete_onboarding_step` for `account`.

   For the satellite alternative, the account lives on the satellite machine, not in this
   sandbox: the user runs `bin/qiskit-account` there, from their own checkout of this repository.

6. **Check the deployed titles.** If the user said the functions are deployed, check the titles
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
   `mark_onboarding_complete`. If the `function` check failed or the functions are not deployed,
   say setup is complete for everything else and leave that step open; don't mark it done just to
   finish onboarding. Explain that workflows run on the Qiskit Function and that this agent's
   skills guide the next steps:
   `run-workflow` prepares and runs a pipeline, `preflight` checks access before a job, and
   `report-run` reports it; `connect-cluster` sets up the cluster satellite alternative. From
   lahs-workflows come `run-lahs-workflow`, `run-qiskit-function` (deploys the function) and
   `analyse-sweep`. If a skill is not listed, read its `SKILL.md` under
   `~/work/lahs-agentic-workflows/skills/` or `~/work/lahs-workflows/.bob/skills/` instead.
