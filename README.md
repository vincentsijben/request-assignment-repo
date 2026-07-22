# request-assignment-repo

Public self-service intake repository for students to request assignment repositories.

Students submit one Issue Form. A GitHub Action validates the request and creates/invites access to a repository automatically.

## How it works

1. Student opens a single public URL and submits the issue form.
2. Workflow validates:
   - access code (shared in Moodle)
  - organization/owner target and allowlist (optional)
  - assignment choice (final or final-second)
   - classroom format (optional regex)
   - email format and domain (optional)
   - account age / request rate limits (anti-abuse)
3. Workflow creates the target repo when missing.
4. Workflow invites/adds the student GitHub account with push access.
5. Workflow comments the result in the issue and tags the owner, so you get a GitHub notification.

## Student fields

- Access code
- Organization
- Assignment (final or final-second)
- Classroom
- Full name
- Email address

GitHub handle is derived from the issue author automatically.

## Required setup (once)

### 1. Repository secret

Set this in Settings -> Secrets and variables -> Actions -> Secrets:

- `ADMIN_TOKEN`
  - PAT from your instructor account
  - scopes needed:
    - personal target owner: `repo`
    - org target owner: `repo`, `admin:org`

- `REQUEST_CODE`
  - the shared Moodle code used for request validation

### 2. Repository variables

Set these in Settings -> Secrets and variables -> Actions -> Variables:

- `TARGET_OWNER` (optional default)
  - default owner/org where student repos are created
  - used when Organization field is empty (or as strict match when no allowlist is set)
- `ALLOWED_TARGET_OWNERS` (optional, recommended)
  - comma- or newline-separated list of allowed owners/orgs students may request
  - example: `semester-1-org,semester-2-org,instructor-account`
- `REPO_PREFIX` (optional fallback)
  - used when assignment-specific prefixes are not set
  - example: `final-`
- `REPO_PREFIX_FINAL` (recommended)
  - prefix used when Assignment is `final`
  - example: `final-`
- `REPO_PREFIX_FINAL_SECOND` (recommended)
  - prefix used when Assignment is `final-second`
  - example: `final-second-`
- `REPO_VISIBILITY` (optional)
  - `private` (default) or `public`
- `TEMPLATE_OWNER` (optional)
  - owner of template repo
- `TEMPLATE_REPO` (optional)
  - template repo name; if set, new repos are generated from template
- `ALLOWED_EMAIL_DOMAIN` (optional)
  - example: `zuyd.nl`
- `ALLOWED_CLASSROOM_REGEX` (optional)
  - example: `^CMD1[A-C]$`
- `MAX_REQUESTS_PER_DAY` (optional, default `3`)
- `MIN_ACCOUNT_AGE_DAYS` (optional, default `3`)

## Notifications

GitHub Actions cannot reliably send custom email without external services.

This setup instead posts a result comment and mentions `@<repo-owner>` in GitHub, which triggers GitHub notifications.

## Moodle link

Share this URL in Moodle:

`https://github.com/<your-user>/request-assignment-repo/issues/new?template=request-assignment-repo.yml`

## Abuse protection included

- Only processes issues with `repo-request` label (from the form)
- Rejects bot accounts
- Access code check against secret
- Organization/owner format and allowlist validation
- Assignment validation (must be final or final-second)
- Basic email and classroom validation
- Daily request limit per GitHub account
- Minimum account age check
- Redacts access code from stored issue body

## Local development

No local runtime is required. Logic runs in `.github/workflows/self-service-intake.yml`.
