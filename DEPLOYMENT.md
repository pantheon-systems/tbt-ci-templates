---
contract_version: 2
kind: tool                          # CI templates that two Terminus plugins clone from this repo's default branch at run time
service: null                       # doesn't apply

trigger: merge-to-main              # there is no release: users get whatever main is when they run the command
merged_implies_deployed: true       # verified: both consumers `git clone` the default branch (build-tools: --depth 1), no ref pinned
other_paths_to_prod: []             # none: users can pass their own --ci-template repo, but that's not this repo

pipeline:
  system: other                     # none: the repo has no CI (no workflows, no CircleCI config)
  workflow: null
  jobs_in_order: []
  typical_minutes: 0                # live as soon as the merge lands
  minutes_by_change: {}
  watch: "nothing to watch; `git ls-remote https://github.com/pantheon-systems/tbt-ci-templates HEAD` == the merge SHA"
  concurrency_group: null
  cancel_in_progress: false
  blocked_by_waiting_run: null
  approval_gate: null

repo_properties:
  observed_at: 2026-11-10
  trust_level: TL4
  ccb_status: in-ccb
  owner: developer-experience
  repo_type: library

gating_checks:                      # `gh api repos/pantheon-systems/tbt-ci-templates/rules/branches/main`, 2026-11-10
  required: ["Wiz IaC Scanner", "Wiz Vulnerability Scanner", "Wiz Data Scanner"]   # org TL4 only; no repo ruleset
  approvals: {required: 1, code_owner: true, last_push_approval: false, strict_up_to_date: false, merge_queue: false}
  run_but_not_required: []          # no CI exists
  conditional: []

environments: []

artifact:
  image: null
  tag_scheme: none                  # verified: no tags, no releases
  tag_mutable: null
  retention: "git history"
  version_signal: none              # consumers record nothing about which commit they copied
  tag_to_sha: null

release:
  mechanism: "none: the default branch is the release"
  never: "force-push or rewrite main (it's what users clone)"

consumers:                          # gh search code --owner pantheon-systems "tbt-ci-templates", 2026-11-10
  - {repo: terminus-build-tools-plugin, pin: "src/Commands/ProjectCreateCommand.php default --ci-template=git@github.com:pantheon-systems/tbt-ci-templates.git", version: "main (floating)", owner: developer-experience, bumped_by: null}
    # `terminus build:project:create` copies <cms>/.ci, <cms>/providers/<ci>/ and <cms>/tests into new projects whose source has no CI
  - {repo: terminus-conversion-tools-plugin, pin: "src/Commands/ConvertToComposerSiteCommand.php copyCiTemplate()", version: "main (floating)", owner: developer-experience, bumped_by: null}

deployed_when:
  - "the merge is on main (git ls-remote ... HEAD == merge SHA); the next command run by any user copies it"

health:
  soak_minutes: null
  baseline: null
  checks:
    - name: a fresh project's CI passes
      kind: command
      run: null                     # TODO(owner): there's no automated check; the realistic one is running `terminus build:project:create` for the changed CMS/provider and watching that project's first CI run
      healthy: null
  known_baseline_errors: []
  dashboards: []
  alerts: []

rollback:
  options:
    - when: always
      kind: revert-and-merge
      steps: "git revert <bad-merge-sha>, PR, 1 code-owner approval, merge: live for the next user immediately"
      time_to_rollback_minutes: 15  # inferred: review time dominates
  not_restored: ["projects created while the bad version was on main keep the copied files; their owners must fix them"]
  one_way_changes: ["files copied into users' projects"]
  currently_blocked: null
  last_exercised: null
  capture_before_merge: "git ls-remote https://github.com/pantheon-systems/tbt-ci-templates HEAD   # last good SHA"
  who_can: ["@pantheon-systems/developer-experience (code owners)"]
  verify: "main's HEAD tree matches the last good SHA for the reverted paths (git diff <last good> HEAD -- <paths> is empty)"

escalation:
  slack: "#ask-hosting"
  owner_team: developer-experience
  paged: false                      # inferred

preflight:
  - "gh auth status"

automation_blockers:
  - blocker: "every merge is live immediately for every user, with no version, no CI and no way to tell which commit a project copied"
    clears_when: "the plugins pin a tag of this repo (and a release is cut), or at least the repo gets CI that lints the templates"
  - blocker: "no health signal"
    clears_when: "a smoke test that creates a project from each template (or a scheduled one) exists"

verified:
  at: null
  by: null
  change: null
  running_digest: null              # doesn't apply
---

# Deployment

This repo holds the CI configuration templates (CircleCI, GitHub Actions, GitLab, Bitbucket, per CMS: `d7`, `d8`, `d9`, `wp`) that Terminus copies into new projects. There's no build and no release: **the default branch is what users get.** A merge is live for the next user immediately.

## How a change reaches production

1. **PR → merge.** TL4 org ruleset: 1 approval from a code owner (`@pantheon-systems/developer-experience`). Only the Wiz checks are required. There's no repo CI and no merge queue.
2. **Users get it on their next run** of:
   - `terminus build:project:create` ([terminus-build-tools-plugin](https://github.com/pantheon-systems/terminus-build-tools-plugin)): when the source project has no CI of its own, it runs `git clone --depth 1` of this repo's default branch and copies `<cms>/.ci`, `<cms>/providers/<ci provider>/` and `<cms>/tests`.
   - The Composer conversion in [terminus-conversion-tools-plugin](https://github.com/pantheon-systems/terminus-conversion-tools-plugin) (`copyCiTemplate()`), which clones the default branch the same way.
   Neither pins a ref.

Last template change: 2024-06-06. The templates use `quay.io/pantheon-public/build-tools-ci` images (`6.x`, `8.x-php7.4`, `8.x-php8.1`), an image line Pantheon has stopped updating.

## How to confirm it deployed

`git ls-remote https://github.com/pantheon-systems/tbt-ci-templates HEAD` returns the merge SHA. That's all there is.

## How to confirm it's healthy

There's no signal. The only real test is to create a project from the changed template (`terminus build:project:create` with the matching CMS and `--ci` provider) and watch its first CI run. TODO(owner): is that the expected check, and which CMS/provider combinations still matter?

## Rollback

Revert and merge: `git revert <bad-merge-sha>`, open a PR, get one code-owner approval, merge. The next user gets the reverted templates immediately. Projects created in the meantime keep the files they copied; tell their owners if the problem breaks their CI.

Capture the last good SHA before merging: `git ls-remote https://github.com/pantheon-systems/tbt-ci-templates HEAD`.

## Escalation

- Owner: `@pantheon-systems/developer-experience`. Ask in `#ask-hosting`. Not paged.

## Suggested pipeline changes

- Have the plugins clone a tag (`--branch <tag>`) and cut releases here, so a merge isn't instantly live and rollback is a repin.
- Add CI that at least lints the YAML templates.
- Move the templates off the `build-tools-ci` images.
