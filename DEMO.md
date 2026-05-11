# DRAFT: Chainguard Actions Demo
Note: This flow is derived from our demo script document.

## Problem Statement / Framing

- CI/CD is one of the customer’s most privileged and least protected layers, and attacks are escalating. How do you handle these security holes, and prevent them in the first place?
- Before state: Teams are relying on community actions, manual review, and partial hygiene like occasional SHA pinning.
  - While SHA pinning ensures specific versions of actions are used, the yaml code itself is not necessarily analyzed for vulnerable practices.
  - Bumping of actions versions is often automated with Dependabot, and PRs are merged by agents or without much scrutiny or analysis baked in.
- After state: Teams use hardened drop-in replacements with the same inputs and outputs, plus continuous re-hardening and a visible hardening record.

## Real World Example: tj-actions/changed-files

- The `tj-actions/changed-files` GitHub Action was compromised in March 2025. It is intended to track all changed files and directories relative to a target branch.
- An attacker used a compromised GitHub PAT to commit a malicious 3rd party script hosted on a GitHub gist to dump CI logs, potentially including secrets.
- This affected more than just one version, since the attacker updated existing version tags to make them all point to the malicious code.
  - The issue is broad: for example, if your Actions strategy was to point to specific version tags, since the tags are mutable, you could have been affected as soon as you re-run your pipeline. If you were pinning to a SHA, and your Dependabot bumped to a compromised version, you could have been affected.

## How Chainguard Actions Helps

- Chainguard Actions is a hardened-alternative to using Actions from the public internet. Our Actions take open source actions and modify them. You can see the Hardening report for each action in HARDENING.md.
- In the case of the `tj-actions/changed-files` action, it lives within a repo under the `chainguard-actions` org, which does not have PATs provisioned to community members, making it impossible for a non-Chainguard employee to publish to the repository.
