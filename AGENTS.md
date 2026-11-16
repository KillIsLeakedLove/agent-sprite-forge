# Repository Security

This is a public repository. Never commit or push files containing sensitive
information, especially API keys, access tokens, passwords, private keys, or
other credentials. Use environment variables or untracked local configuration
for secrets, and use clearly fake placeholders in examples and tests.

Before every commit and push, inspect the files and changes being included for
secrets and other sensitive information. Do not push if any are found; remove
them from the commit history to be pushed and notify the user if credentials
may have been exposed. Never print or disclose their values.
