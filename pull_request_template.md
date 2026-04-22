### Description (if necessary)

Fixes #

### Checklist

Before submitting, please confirm you've:

- [ ] self-reviewed the diff
- [ ] removed any code that doesn't belong to this change
- [ ] added tests for new behaviour and run the existing suite
- [ ] run the repo's linters / formatters
- [ ] removed any secret or environment variable from the diff
- [ ] updated AWS Secrets Manager (and any other config store in use) with any env variables added, changed, or removed
- [ ] **not passed any secret, credential, API key, private key, or other sensitive value to a logger as a keyword argument or via `logger.bind(...)`** — these bypass the `tf_utils.logger` redactor. Log secrets only through the message string, where the redactor scrubs them.
- [ ] registered any new secret env var in the consuming service's `REDACT_ENV_VARS` / `REDACT_VALUES_BY_NAME_KEYS` (see `docs/security/procedures/log-masking-verification.md` in the `docs` repo)
- [ ] made sure you're up to date with `main`

You can remove this checklist before submitting.
