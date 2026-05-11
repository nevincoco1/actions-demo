# Chainguard Actions Demo

This is a repository to be used to demo Chainguard Actions. It leverages actions from `chainguard-actions` and is configured to run Dependabot weekly to update the actions's hashes.

## Actions in Repo
Current actions are `chainguard-actions/cosign-installer` and `chainguard-actions/setup-crane` since these are publicly accesible. Workflow installs cosign and crane, and Dependabot updates by SHA.

### Quick Demo
- Show `test.yml` pointing to SHA-pinned references to Chainguard Actions.
- Show `dependabot.yml` which has no change from typical implementation in Actions docs.
- Show open PR to update by SHA.
- Show `HARDENING.md` for `setup-crane` action [here](https://github.com/chainguard-actions/setup-crane/blob/main/HARDENING.md). Note the script-injection vulnerability, and show the difference in the code from the upstream: 
  - Upstream [Line 22](https://github.com/imjasonh/setup-crane/blob/main/action.yml#L22) takes an `${{ inputs.version }}` user-supplied input into a shell could be attacked with remote code execution. The problem here is input is interpolated directly into the shell script.
  - Chainguard version [Line 26](https://github.com/chainguard-actions/setup-crane/blob/main/action.yml#L26) takes the user input and then assigns into to an envar, so it does not execute the input as code.