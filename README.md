# Terminus Build Tools templates CI Integrations.

[![Actively Maintained](https://img.shields.io/badge/Pantheon-Actively_Maintained-yellow?logo=pantheon&color=FFDC28)](https://pantheon.io/docs/oss-support-levels#actively-maintained-support)

CI Integrations for Terminus Build Tools template repos.

Enter to the right folder and follow the instructions in the README.md file.

## Rollback

- **When:** a template change breaks CI in projects created by `terminus build:project:create` or the Composer conversion.
- **Recover:** the default branch is live for every user, so revert the change and merge; the next user gets the fix immediately. Steps: [DEPLOYMENT.md#rollback](DEPLOYMENT.md#rollback).
- **Verify:** `main` no longer contains the bad change, and a freshly created project's CI passes.
- **Who:** `@pantheon-systems/developer-experience`, `#ask-hosting` (not paged).
