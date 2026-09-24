

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
