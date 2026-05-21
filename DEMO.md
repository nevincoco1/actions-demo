# Chainguard Actions Demo

This demo follows the `Tell Show Tell` demo format.

## Problem Statement / Framing

- CI/CD is one of the customer’s most privileged and least protected layers, and attacks are escalating. How do you handle these security holes, and prevent them in the first place?
- Before state: Teams are relying on community actions, manual review, and partial hygiene like occasional SHA pinning.
  - While SHA pinning ensures specific versions of actions are used, the yaml code itself is not necessarily analyzed for vulnerable practices.
  - Bumping of actions versions is often automated with Dependabot, and PRs are merged by agents or without much scrutiny or analysis baked in.
- After state: Teams use hardened drop-in replacements with the same inputs and outputs, plus continuous re-hardening and a visible hardening record.

## Real World Example: tj-actions/changed-files

- The `tj-actions/changed-files` GitHub Action was compromised in March 2025. It is intended to track all changed files and directories relative to a target branch.
- An attacker used a compromised GitHub PAT to commit a malicious 3rd party script hosted on a GitHub gist to dump CI logs, potentially including secrets.
- This affected more than just one version, since the attacker updated existing version tags to make them all point to the malicious code. It led to the exposure of secrets in *23,000* repos.
  - The issue is broad: for example, if your Actions strategy was to point to specific version tags, since the tags are mutable, you could have been affected as soon as you re-run your pipeline. If you were pinning to a SHA, and your Dependabot bumped to a compromised version, you could have been affected.

## How Chainguard Actions Helps

- Chainguard Actions is a hardened-alternative to using Actions from the public internet. Our Actions take open source actions and modify them. You can see the Hardening report for each action in HARDENING.md.
- In the case of the `tj-actions/changed-files` action, it lives within a repo under the `chainguard-actions` org, which does not have PATs provisioned to community members, making it impossible for a non-Chainguard employee to publish to the repository.

## Tell
In this demo, we will see a pipeline that installs some tooling to interact with a remote registry (`crane`) and verify signatures of these images (`cosign`). We will see how we can replace these with `Chainguard Actions`, how these actions are hardened, and how we can keep them up to date in our current dependabot workflow. Finally, we will see how we can enforce the usage of a hardened action alternative using GitHub's native settings.

## Show

In this pipeline we are installing some tooling to interact with a remote registry (`crane`) and verify signatures of these images (`cosign`). Let's view the pipeline code.

### View Current State
- Navigate to `test-original.yml`
  - Note the SHA-pinning practice, but ultimately, pulling from public repositories with various maintainers.

#### *Key points:* 
- SHA-pinned not enough/need to audit every SHA bump*
- Anyone with a PAT for the repos in the workflow can modify a release retroactively

### What's the solution?

- Check for a Chainguard alternative. Visit the [Chainguard Actions organization](https://github.com/chainguard-actions).
- Search for `setup-crane` under `Repositories`.
- Show the `HARDENING.md` (on main branch, otherwise on release's branch).
- Note the `Findings Fixed: script-injection` and the possible RCE vulnerability.
- Compare the upstream code to Chainguard's alternative.
  - Upstream [Line 22](https://github.com/imjasonh/setup-crane/blob/main/action.yml#L22) takes an `${{ inputs.version }}` user-supplied input into a shell could be attacked with remote code execution. The problem here is input is interpolated directly into the shell script.
  - Chainguard version [Line 26](https://github.com/chainguard-actions/setup-crane/blob/main/action.yml#L26) takes the user input and then assigns into to an envar, so it does not execute the input as code.

#### *Key points:*
- We harden actions from the upstream and modify the actual yaml code to follow security best practices
- We have a large catalog of actions (300+ as of May 19 2026)
- We check every release against our hardening policies and publish all findings
- We create a separate commit for each findings for full traceability of changes
- We publish to our own GitHub repository, eliminating risk of modifying releases as a public maintainer

### Show the change
- Navigate to your demo project and view `test-chainguard-tag.yml`
- Show the only change is the reference to `chainguard-actions` and the updated SHA to point to the SHA for our action.
- Navigate to `depandabot.yml` and note that there is no change needed to this file to keep chainguard-actions up to date.
- Visit the [pull request](https://github.com/chainguard-demo/actions-demo/pull/1) bumping the version of the action to a newer SHA, showcasing no change to the depednabot workflow.
- Visit the pipeline run and note that the pipeline still works as expected.

#### *Key points:*
- It's incredibly easy to leverage Chainguard Actions, all you have to do is update the upstream organization to `chainguard-actions` and pin to the SHA of choice (or tag)
- The pipelines have parity
- There is no change to your dependabot configuration to use something like Chainguard Actions, as they are simply modified yaml

### Enforce the best practice
- In your fork, navigate to `Settings > Actions > General` and select `Allow {ORG}, and select non-{ORG}, actions and reusable workflows`
- Click `Allow actions by Marketplace` and in `Allow or block specified actions and reusable workflows`, type `chainguard-actions/*`

#### *Key points:*
- You can enforce using actions in the chainguard-actions organization and your own

### Tell
In this demo, we showed a GitHub Actions pipeline using publically maintained source code that could be affected by the mutablility of actions releases. We saw a real script-injection vulnerability in one of the included actions detected by Chainguard, and a patched action built by Chainguard that eliminated the risk of remote code execution. We saw how to convert to Chainguard Actions, and how it fits in our current configuration for GitHub Actions.

