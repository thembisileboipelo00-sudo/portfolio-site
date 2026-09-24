

A static personal portfolio site, hosted on Amazon S3 and distributed
through Amazon CloudFront as a CDN.

## Live site
`https://[your-cloudfront-domain].cloudfront.net` — add once deployed.

## Demo video
[Link to 5–10 min YouTube demo — add after recording]

## Architecture

```
Visitor
   |
   v
CloudFront (CDN, HTTPS, caching at edge locations)
   |
   v
S3 bucket (private, origin access control only — not public)
```

## Why S3 + CloudFront, and not just S3 alone

S3 can serve a static website directly over plain HTTP, which would have
been the simpler build. CloudFront was added deliberately for three
reasons, each a real design decision rather than "because it's cloud":

1. **HTTPS.** S3 static website hosting does not support HTTPS on its own
   domain. CloudFront terminates TLS at the edge, so the site is served
   securely without needing a separate certificate-management setup on S3
   itself.
2. **Speed via edge caching.** CloudFront caches the site's files at
   edge locations close to the visitor, instead of every request going
   back to a single S3 region. For a portfolio that might be viewed from
   anywhere, this matters more than it would for an internal tool.
3. **The bucket stays private.** Using CloudFront with Origin Access
   Control (OAC) means the S3 bucket itself is never publicly reachable —
   only CloudFront can read from it. This is a meaningfully different (and
   safer) security posture than the common tutorial approach of making the
   S3 bucket public, and was chosen specifically to avoid that.

## Deployment steps (AWS Management Console)

### 1. Create the S3 bucket
- S3 → **Create bucket**
- Name must be globally unique, e.g. `yourname-portfolio-site`
- Region: closest to you
- **Uncheck "Block all public access"** only if you plan to make the
  bucket public directly (not required here, since CloudFront will use
  OAC instead) — for this setup, leave public access **blocked**.
- Create the bucket.

### 2. Upload the site files
- Open the bucket → **Upload**
- Upload `index.html`, `error.html`, and the `css/` folder (keeping the
  folder structure — `css/style.css` should stay inside a `css/` prefix)

### 3. Create the CloudFront distribution
- CloudFront → **Create distribution**
- **Origin domain**: select your S3 bucket from the dropdown (choose the
  S3 option, not the website-endpoint option)
- **Origin access**: choose **Origin access control settings (recommended)**
  → create a new OAC
- **Default root object**: `index.html`
- Viewer protocol policy: **Redirect HTTP to HTTPS**
- Leave other settings at their defaults for a first deployment
- Create the distribution (it takes several minutes to deploy globally)

### 4. Update the S3 bucket policy
CloudFront will show a banner saying the bucket policy needs updating —
click **Copy policy**, then go to your S3 bucket → **Permissions** →
**Bucket policy** → paste it in and save. This is what grants CloudFront
(and only CloudFront) permission to read from the bucket.

### 5. Set the custom error page (for the 404)
- CloudFront distribution → **Error pages** tab → **Create custom error
  response**
- HTTP error code: `403` (S3 returns 403, not 404, for missing objects
  when the bucket is private)
- Customize error response: Yes
- Response page path: `/error.html`
- HTTP response code: `404`

### 6. Test
Once the distribution status shows **Deployed**, visit the CloudFront
domain shown on the distribution's overview page
(`xxxxxxxxxxxx.cloudfront.net`).

## Updating the site after first deploy

```bash
# Re-upload changed files to S3, then invalidate the CloudFront cache
# so visitors see the update immediately instead of a cached copy:
aws s3 sync ./site s3://yourname-portfolio-site --delete
aws cloudfront create-invalidation --distribution-id YOUR_DIST_ID --paths "/*"
```

(Requires the AWS CLI configured with credentials — the console can also
do both of these steps manually if the CLI isn't set up.)

## Known limitations

- No custom domain configured — using the default CloudFront `.net`
  address. Adding one would need Route 53 and an ACM certificate.
- No CI/CD — updates are uploaded manually. A GitHub Actions workflow
  could automate the `s3 sync` + invalidation on every push.
- Cache invalidation after an update currently has to be triggered
  manually.

## Project structure

```
portfolio-site/
├── site/
│   ├── index.html
│   ├── error.html
│   └── css/
│       └── style.css
└── README.md
```

## Author's note

Built as a solo project for the Cloud Computing elective which is the choice to
use CloudFront with Origin Access Control instead of a public S3 bucket.
