# Security Rules

This repository must never contain:

- AWS access keys or secret access keys
- Passwords, tokens, session cookies, or private keys
- AWS account IDs, billing data, or personal information
- Real customer, employer, or school credentials
- Unredacted screenshots containing sensitive data
- Terraform state files or populated variable files

Use placeholders such as `YOUR_ACCOUNT_ID`, `EXAMPLE_BUCKET_NAME`, and `REDACTED`.

Before every commit:

1. Review changed files with `git diff --staged`.
2. Confirm that screenshots are redacted.
3. Confirm that resources created for a lab were deleted when no longer needed.
4. Never use root-user credentials in scripts or the AWS CLI.

If a secret is committed, revoke or rotate it immediately. Deleting it from the latest commit is not sufficient because Git history may still contain it.
