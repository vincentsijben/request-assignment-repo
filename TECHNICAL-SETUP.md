# Technical Setup and Operations

This document contains the full instructor/admin configuration for the self-service assignment repository intake workflow.

## How it works

1. Student opens a single public URL and submits the issue form.
2. Workflow validates:
   - access code (shared in Moodle)
   - Repository target format (`owner/prefix`)
   - organization/owner allowlist (optional)
  - email format
   - account age / request rate limits (anti-abuse)
3. Workflow creates the target repo from template `<owner>/<prefix>-startercode` when missing.
4. Workflow invites/adds the student GitHub account with maintain access.
5. Workflow comments the result in the issue and tags the owner, so you get a GitHub notification.
6. Issue template assigns the repository owner on creation, and workflow reinforces assignment for extra visibility.

## Student fields

- Full name
- Email address
- Classroom
- Repository (`owner/prefix`, for example `mia-mmt4-2526/final` or `mia-mmt4-2526/final-`)
- Access code

GitHub handle is derived from the issue author automatically, and the issue title is normalized to `request assignment for <github-handle>`.

## Required setup (once)

### 1. Repository secret

Set this in Settings -> Secrets and variables -> Actions -> Secrets:

- Secrets page:
  <https://github.com/vincentsijben/request-assignment-repo/settings/secrets/actions>

- `ADMIN_TOKEN`
  - PAT from your instructor account <https://github.com/settings/tokens>
  - scopes needed:
    - personal target owner: `repo`
    - org target owner: `repo`, `admin:org`

- `REQUEST_CODE`
  - the shared Moodle code used for request validation
  - supports multiple values as a comma-separated list (for example `spring2026,fall2026`)

### 2. Repository variables

Set these in Settings -> Secrets and variables -> Actions -> Variables:

- Variables page:
  <https://github.com/vincentsijben/request-assignment-repo/settings/variables/actions>

- `ALLOWED_TARGET_OWNERS` (optional, recommended)
  - comma- or newline-separated list of allowed owners/orgs in `Repository`
  - example: `semester-1-org,semester-2-org,instructor-account`
- `MAX_REQUESTS_PER_DAY` (optional, default `10`)
- `MIN_ACCOUNT_AGE_DAYS` (optional, default `3`)

## Notifications

GitHub Actions cannot reliably send custom email without external services.

This setup posts a result comment and mentions `@<repo-owner>` in GitHub for both successful and rejected requests, which triggers GitHub notifications.

Optional: Gmail SMTP alert notifications can also be sent for each processed issue. The email includes submitted data, the issue reply text, and a direct issue URL.

If `GMAIL_SMTP_USERNAME`, `GMAIL_SMTP_APP_PASSWORD`, or `GMAIL_TO` is missing or empty, the Gmail alert steps are skipped automatically.

### Gmail SMTP setup

1. Create an App Password (do not use your normal Gmail password).
2. In Google Account, go to Security: <https://myaccount.google.com/security>
3. Open App Passwords: <https://myaccount.google.com/apppasswords>
4. Create a new app password (name it something like `GitHub Actions Intake`).
5. Copy the 16-character password once. Save it temporarily; Google will not show it again.
6. Add GitHub repository secrets. In your repo, open Settings -> Secrets and variables -> Actions -> New repository secret, then add:
7. `GMAIL_SMTP_USERNAME` = your Gmail address
8. `GMAIL_SMTP_APP_PASSWORD` = the 16-char app password from Google
9. `GMAIL_TO` = destination email address for alerts

## Moodle link

Share this URL in Moodle:

<https://github.com/vincentsijben/request-assignment-repo/issues/new?template=request-assignment-repo.yml>

You can prefill fields in Moodle by appending query parameters that match form IDs. Example:

[Open prefilled request form](https://github.com/vincentsijben/request-assignment-repo/issues/new?template=request-assignment-repo.yml&repository=mia-mmt1-2627%2Ffinal)

## Abuse protection included

- Only processes issues with `repo-request` label (from the form)
- Rejects bot accounts
- Access code check against secret
- Full name check (must include at least first and last name)
- Repository format validation (`owner/prefix`)
- Organization/owner allowlist validation
- Template repository existence and access check
- Daily request limit per GitHub account
- Minimum account age check
- Redacts access code from stored issue body

## Local development

No local runtime is required. Logic runs in `.github/workflows/self-service-intake.yml`.
