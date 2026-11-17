# Repository Security

This is a public repository. Never commit or push files containing sensitive
information, especially API keys, access tokens, passwords, private keys, or
other credentials. Use environment variables or untracked local configuration
for secrets, and use clearly fake placeholders in examples and tests.

Before every commit and push, inspect the files and changes being included for
secrets and other sensitive information. Do not push if any are found; remove
them from the commit history to be pushed and notify the user if credentials
may have been exposed. Never print or disclose their values.

## Task Completion

- After each task is complete and its required checks pass, commit and push the
  task's changes to the current branch without asking for separate approval.
- Keep each commit limited to the current task. Preserve unrelated user changes
  and do not include them in the commit.
- Inspect the exact changes being committed for sensitive information before
  every commit and push, as required above.
- Report the commit hash and push result in the final response. If checks fail,
  sensitive information is found, or the push fails, report the issue instead
  of claiming the push succeeded.
