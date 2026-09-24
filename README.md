# Portfolio — Static Site on GitHub Pages


A static personal portfolio site, hosted on GitHub Pages, which serves it
through GitHub's own CDN (Fastly) over HTTPS.

## Live site
https://thembisileboipelo00-sudo.github.io/portfolio-site/

## Demo video
[Link to 5–10 min YouTube demo — add after recording]

## Architecture

```
Visitor
   |
   v
GitHub Pages (Fastly CDN at the edge, automatic HTTPS)
   |
   v
Static files served directly from this repo's main branch
```

## Why GitHub Pages, and the decision behind it

The original plan for this project was Amazon S3 + CloudFront (see
"What I originally planned" below) — that architecture is still the
right one to know for real cloud work, and the reasoning behind it
(private origin, CDN in front, HTTPS termination at the edge) is
explained in that section.

In practice, I moved to GitHub
Pages, which achieves the same core outcome — a static site served
over HTTPS through a CDN — using infrastructure I already had access
to through my existing GitHub account.

**What's the same conceptually:**
- Files are served from edge locations close to the visitor (GitHub
  Pages uses Fastly's CDN), not from a single origin server
- HTTPS is automatic and enforced
- The deployment is triggered by pushing to the repository, not a
  manual upload step

**What's different from the S3 + CloudFront plan:**
- No separate storage layer (bucket) and CDN (distribution) — GitHub
  Pages bundles both into one service, so there's no Origin Access
  Control or bucket policy to configure, because there's no separate
  origin to protect
- No AWS-specific concepts like IAM, OAC, or S3 bucket policies were
  exercised in the final deployment — that trade-off is the honest
  limitation of this pivot, noted below
- Build/deploy is push-to-deploy by default, versus a manual
  `aws s3 sync` + CloudFront cache invalidation

## What I originally planned (S3 + CloudFront)

For the record, and because I can still explain every part of it:

1. **HTTPS** — S3 static website hosting alone doesn't support HTTPS
   on its own domain; CloudFront terminates TLS at the edge.
2. **Edge caching** — CloudFront caches files at edge locations near
   the visitor instead of every request hitting a single S3 region.
3. **Private origin** — using CloudFront with Origin Access Control
   (OAC) keeps the S3 bucket itself unreachable directly; only
   CloudFront can read from it, which is safer than the common
   tutorial pattern of making the bucket public.

## Deployment steps actually used (GitHub Pages)

1. Push the site's `index.html`, `error.html`, and `css/` folder to
   the root of the `main` branch of this repository
2. Repo → **Settings** → **Pages**
3. **Source**: Deploy from a branch
4. **Branch**: `main`, folder: `/ (root)`
5. Save — GitHub builds and publishes automatically within a minute
   or two
6. Any future push to `main` re-deploys the site automatically —
   no manual cache invalidation step needed

## Known limitations

- No custom domain configured — using the default
  `github.io` subdomain.
- No AWS-specific IAM/OAC/bucket-policy configuration was actually
  exercised, since this pivoted away from S3 + CloudFront — the
  reasoning is documented above for transparency.
- Deploys are automatic on every push to `main`, which is convenient
  but means there's no staging/review step before changes go live.

## Project structure

```
portfolio-site/
├── index.html
├── error.html
├── css/
│   └── style.css
└── README.md
```
