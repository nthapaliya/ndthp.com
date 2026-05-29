# Setup guide

## 1. Create the GitHub repo

```bash
gh repo create ndthp.com --public
git remote add origin https://github.com/nthapaliya/ndthp.com.git
git push -u origin main
```

## 2. Enable GitHub Pages

In the repo on GitHub:
- **Settings → Pages → Source:** select **GitHub Actions** (not a branch)
- GitHub will automatically use the `deploy.yml` workflow

## 3. Set the custom domain

In **Settings → Pages → Custom domain**, enter `ndthp.com` and save.
GitHub will create a `CNAME` file in your repo automatically.

## 4. Configure DNS in Cloudflare

Add these records in Cloudflare DNS for `ndthp.com`:

| Type | Name | Content | Proxy |
|------|------|---------|-------|
| A | `@` | `185.199.108.153` | DNS only (grey cloud) |
| A | `@` | `185.199.109.153` | DNS only |
| A | `@` | `185.199.110.153` | DNS only |
| A | `@` | `185.199.111.153` | DNS only |
| CNAME | `www` | `nthapaliya.github.io` | DNS only |

> **Important:** set these records to **DNS only** (grey cloud), not proxied.
> GitHub Pages handles its own TLS — Cloudflare proxying breaks the certificate validation.

After adding records, go back to **Settings → Pages** in GitHub and click
"Enforce HTTPS" once it becomes available (usually a few minutes).

## 5. Transfer domain registration to Cloudflare (optional but recommended)

This moves registration from Squarespace to Cloudflare Registrar, so both
registration and DNS live in one dashboard. At-cost pricing (~$8.57/year for .com).

**Steps:**
1. Log in to Squarespace → **Domains → ndthp.com → Advanced Settings → Transfer Domain Out**
2. Unlock the domain and copy the **authorization / EPP code**
3. In Cloudflare: **Domain Registration → Transfer Domains → enter ndthp.com**
4. Paste the EPP code when prompted and pay the transfer fee (one year added)
5. Approve the transfer in the email Squarespace sends
6. Transfer completes in 5–7 days; DNS stays live throughout

> You can skip this step and keep registration at Squarespace — it only affects
> where you pay the renewal fee, not how the site works.

## 6. Add your resume PDF

Copy the generated resume into `static/`:

```bash
cp ../resume/Niraj\ Thapaliya\ Resume\ 2026\ DS.pdf static/resume.pdf
```

The resume page at `/resume/` links to `/resume.pdf`.

## Local development

```bash
hugo server --buildDrafts
# → http://localhost:1313
```

## Publishing new content

Create a new project page:

```bash
hugo new content projects/my-new-project.md
# edit content/projects/my-new-project.md
# set draft: false when ready
git add . && git commit -m "add: my new project" && git push
```

The GitHub Action builds and deploys automatically on every push to `main`.
PRs trigger a dry-run build check without deploying.

## Cloudflare Pages (alternative to GitHub Pages)

If you later want to move hosting to Cloudflare Pages instead:

1. In Cloudflare: **Workers & Pages → Create → Pages → Connect to Git**
2. Select this repository, set:
   - Build command: `hugo --minify`
   - Build output directory: `public`
   - Environment variable: `HUGO_VERSION = 0.162.1`
3. Under **Custom Domains**, add `ndthp.com` — one click since DNS is already here
4. Delete the GitHub Pages `A` and `CNAME` records from Cloudflare DNS
5. You can keep the `deploy.yml` workflow or delete it (Cloudflare Pages handles CI/CD)

The main advantage: Cloudflare's edge CDN vs GitHub Pages' CDN, and
slightly faster deploys. Both are free for this traffic level.
