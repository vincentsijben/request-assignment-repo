# request-assignment-repo

Public self-service intake repository for students to request assignment repositories.

Students submit one Issue Form. A GitHub Action validates the request and creates/invites access to a repository automatically.

## How it works

1. Student opens a single public URL and submits the issue form.
2. Workflow validates:
  - access code (shared in Moodle)
  - repoName target format (`owner/prefix`)
  - organization/owner allowlist (optional)
  - classroom format (optional regex)
  - email format and school domain (`@zuyd.nl` by default)
  - account age / request rate limits (anti-abuse)
3. Workflow creates the target repo from template `<owner>/<prefix>startercode` when missing.
4. Workflow invites/adds the student GitHub account with push access.
5. Workflow comments the result in the issue and tags the owner, so you get a GitHub notification.
6. Issue template assigns the repository owner on creation, and workflow reinforces assignment for extra visibility.

## Student fields

- Full name
- Email address
- Classroom
- repoName (`owner/prefix`, for example `mia-mmt1-2627/final-`)
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

### 2. Repository variables

Set these in Settings -> Secrets and variables -> Actions -> Variables:

- Variables page:
  <https://github.com/vincentsijben/request-assignment-repo/settings/variables/actions>

- `ALLOWED_TARGET_OWNERS` (optional, recommended)
  - comma- or newline-separated list of allowed owners/orgs in `repoName`
  - example: `semester-1-org,semester-2-org,instructor-account`
- `REPO_VISIBILITY` (optional)
  - `private` (default) or `public`
- `TEMPLATE_OWNER` (optional)
  - override template owner
  - default: same owner as `repoName`
- `TEMPLATE_REPO` (optional)
  - override template repo name
  - default: `<prefix>startercode` derived from `repoName`
- `ALLOWED_EMAIL_DOMAIN` (optional)
  - default is `zuyd.nl`
  - set this only if you want a different required domain
- `ALLOWED_CLASSROOM_REGEX` (optional)
  - example: `^CMD1[A-C]$`
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
7. `GMAIL_SMTP_USERNAME` = `your-gmail@example.com`
8. `GMAIL_SMTP_APP_PASSWORD` = the 16-char app password from Google
9. `GMAIL_TO` = `alerts@example.com`

## Moodle link

Share this URL in Moodle:

`https://github.com/<your-user>/request-assignment-repo/issues/new?template=request-assignment-repo.yml`

You can prefill fields in Moodle by appending query parameters that match form IDs. Example:

`https://github.com/<your-user>/request-assignment-repo/issues/new?template=request-assignment-repo.yml&classroom=CMD1A&repo_name=mia-mmt1-2627%2Ffinal-`

## Abuse protection included

- Only processes issues with `repo-request` label (from the form)
- Rejects bot accounts
- Access code check against secret
- Full name check (must include at least first and last name)
- repoName format validation (`owner/prefix`)
- Organization/owner allowlist validation
- Template repository existence and access check
- School email check (`@zuyd.nl` by default) and classroom validation
- Daily request limit per GitHub account
- Minimum account age check
- Redacts access code from stored issue body

## Local development

No local runtime is required. Logic runs in `.github/workflows/self-service-intake.yml`.
