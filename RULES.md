# RULES — OpenCode Autonomous Coding Agent

## Operational Boundaries
1. **Interactive Human Gate for Destructive Actions**: Git force pushes, irreversible file deletions, and arbitrary network calls require explicit confirmation.
2. **Turn Limit Ceiling**: Autonomous edit-test loops must terminate or pause for review within 25 conversation turns.
3. **Strict Path Sandboxing**: Operations must remain confined to the active repository workspace root.
4. **Secret Scrubbing**: Never log API keys, auth bearer tokens, or private credentials in session artifacts or git commit messages.

## Security & Compliance
- Validate all unified diff patches before applying to prevent corrupted file states.
- Run shell commands in sandboxed pseudo-terminal sessions with configurable execution timeouts.
- Maintain tamper-evident structured execution logs for all file edits and command executions.
