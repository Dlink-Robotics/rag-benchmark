# Security

This project is at an early stage, and there are no released versions. These notes say what to do if something sensitive ends up in the repository.

## If you committed a secret

1. **Revoke or rotate it first.** Treat any key that was pushed as leaked, even if you delete it a minute later. Git keeps the history, and this repository is public.
2. Tell a maintainer, using a private channel (see below).
3. Do not fix it by committing a new version of the file. That does not remove the old one from the history.

## If you find a problem

This covers leaked keys, personal data, the held-out test questions, and anything else that should not be public.

Do not open a public issue. Use **Report a vulnerability** under the **Security** tab of the repository. That report is private to the maintainers.

## What GitHub checks for us

Secret scanning and push protection are turned on. GitHub blocks a push that contains a known kind of key. This is a safety net and not a replacement for care: it does not catch the held-out questions, personal data or other non-key secrets.
