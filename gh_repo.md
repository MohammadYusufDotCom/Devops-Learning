Option 1: Delete all images via GitHub UI (manual, slow)

Only reasonable if you have very few images.

Go to your repo on GitHub

Click Packages (right sidebar)

Click the container package (e.g. ghcr.io/org/repo)

Manually delete each version

❌ No bulk delete
❌ Painful for many tags

Option 2: Delete all container images using GitHub CLI (recommended)
Prerequisites
gh auth login


Ensure you have:

Admin or maintainer access to the repo

gh CLI ≥ v2.0

Step 1: List all container package versions

Replace values accordingly.

OWNER=your-org-or-username
PACKAGE=repo-name

gh api \
  -H "Accept: application/vnd.github+json" \
  /users/$OWNER/packages/container/$PACKAGE/versions \
  --paginate \
  --jq '.[].id'


If it’s an organization repo, use:

/orgs/$OWNER/packages/container/$PACKAGE/versions

Step 2: Delete all versions (⚠️ irreversible)
for id in $(gh api \
  /orgs/$OWNER/packages/container/$PACKAGE/versions \
  --paginate \
  --jq '.[].id'); do
  gh api \
    -X DELETE \
    /orgs/$OWNER/packages/container/$PACKAGE/versions/$id
done


✅ Deletes all Docker images/tags for that repo
✅ Fast and scriptable

Option 3: Using raw GitHub REST API (curl)

Useful for CI or automation.

TOKEN=ghp_xxx
OWNER=your-org
PACKAGE=repo-name


List versions:

curl -H "Authorization: Bearer $TOKEN" \
  https://api.github.com/orgs/$OWNER/packages/container/$PACKAGE/versions


Delete a version:

curl -X DELETE \
  -H "Authorization: Bearer $TOKEN" \
  https://api.github.com/orgs/$OWNER/packages/container/$PACKAGE/versions/VERSION_ID

Important Notes

This deletes images only, not the package itself

You cannot delete the last package version via UI, but the API allows it

Deletion is permanent

Works only for GHCR, not Docker Hub

Want a one-liner?

Tell me:

Org or user repo?

Repo name

Linux/macOS or Windows

I’ll tailor a copy-paste-ready command for you.

 gh api \
  -H "Accept: application/vnd.github+json" \
  /orgs/$OWNER/packages/container/$PACKAGE/versions \
  --paginate
{
  "message": "You need at least read:packages scope to get a package's versions.",
  "documentation_url": "https://docs.github.com/rest/packages/packages#list-package-versions-for-a-package-owned-by-an-organization",
  "status": "403"
}

That error means your GitHub token used by gh does not have the required read:packages permission (and for deletion you’ll also need delete:packages).

This is very common 👍

Fix: Grant the correct scopes to GitHub CLI
Step 1: Check your current auth
gh auth status


If you see something like:

Token scopes: repo, workflow


→ packages scopes are missing.

Step 2: Re-authenticate with package scopes (recommended)

Run:

gh auth refresh -h github.com -s read:packages,delete:packages


If that fails or you want a clean re-login:

gh auth login


When prompted:

Account: GitHub.com

Auth method: HTTPS

Authenticate GitHub CLI: Yes

Scopes:
Add:

read:packages
delete:packages


✅ This updates the token used by gh api.

Step 3: Verify scopes
gh auth status


You should now see:

Token scopes: repo, read:packages, delete:packages

Step 4: Retry the command
gh api \
  -H "Accept: application/vnd.github+json" \
  /orgs/$OWNER/packages/container/$PACKAGE/versions \
  --paginate


It should now return JSON with version IDs.

Common gotchas (important)
1. Org SSO enforcement

If your org enforces SSO:

Go to GitHub → Settings → Applications

Authorize GitHub CLI for that organization

Otherwise you’ll still get 403s.

2. Correct OWNER type

Org-owned repo → /orgs/$OWNER/...

User-owned repo → /users/$OWNER/...

Using the wrong one also causes 403.

Once it works: delete everything (final command)
for id in $(gh api \
  /orgs/$OWNER/packages/container/$PACKAGE/versions \
  --paginate \
  --jq '.[].id'); do
  gh api -X DELETE \
    /orgs/$OWNER/packages/container/$PACKAGE/versions/$id
done


⚠️ Irreversible

If you want, tell me:

Is this an org or personal repo?

Are you using GitHub-hosted runners or local machine?

I’ll sanity-check the exact endpoint and scopes for your setup.
