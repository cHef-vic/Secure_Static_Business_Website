# Secure_Static_Business_Website

## 1. Project Overview

The goal of this project was to design and deploy a professional static business website using AWS, hosting the site in Amazon S3 and delivering it globally through Amazon CloudFront with the deployment following AWS security best practices so that the S3 bucket remains private while visitors access content through CloudFront over HTTPS.

The project is based on a scenario in which a business wants to establish an online presence with a professional website that customers can access from different geographical locations, without the cost and operational overhead of maintaining traditional web servers.

Website sections: Homepage, About Us, Services, Portfolio, Contact

Stack: Amazon S3 (private origin) → CloudFront OAC (SigV4) → CloudFront CDN → HTTPS

Cost: Deployed entirely within the AWS Free Tier

Live URL: https://d2273fpwmvxbi.cloudfront.net

## 2. Architecture
[Architecture Diagram](images/architecture-diagram.png)

Design principle: the S3 bucket is never reachable from the internet. CloudFront signs every request it sends to S3, and the bucket policy is written to trust only that one distribution. This keeps the origin private while still delivering content from edge locations closest to the visitor.

## 3. AWS Services Used
Amazon S3

Amazon CloudFront

CloudFront Origin Access Control (OAC)

## 4. Implementation Steps
### 4.1 Website
Downloaded a ready-made responsive business website template consisting of index.html, a CSS stylesheet, and script.js.

Sections: Homepage, About Us, Services, Portfolio, Contact.

Responsiveness was verified on a physical mobile device as well as in the browser with navigation and layout rendering correctly on small screens.

### 4.2 Amazon S3 (private bucket)
Created a bucket in us-east-1.

Block Public Access: ON.

Object Ownership: ACLs disabled.

Versioning: disabled.

Default encryption left at the S3 default (SSE-S3).

Uploaded the website files via the console with index.html at the bucket root.

[Screenshot showing S3 bucket with Block Public Access enabled
](screenshots/s3_block_public_access.png)

### 4.3 CloudFront distribution
Created the distribution with the S3 bucket selected as the origin.

Origin access: Origin access control settings → Create new OAC (the console
generates the OAC during distribution creation).

Signing behaviour: Sign requests

Signing protocol: SigV4

Viewer protocol policy: Redirect HTTP to HTTPS.

Default root object: index.html.

Cache behaviour / TTLs: left at defaults (no custom cache policy, no compression
changes, no custom error pages configured).

[Screenshot showing default root object](screenshots/cloudfront_default_object.png)

[Screenshot showing CloudFrontwith OAC attached](screenshots/cloudfront_oac.png)

[Screenshot showing CloudFront Behaviour](screenshots/cloudfront_behaviour.png)

### 4.4 Bucket policy
No manual policy editing was required. With the current AWS console flow, the CloudFront console automatically updates the S3 bucket policy when OAC is configured during distribution creation. The resulting policy is scoped to the distribution and grants read-only access (s3:GetObject) to the CloudFront service principal, and the AWS:SourceArn condition restricts it to this specific distribution only.

[Screenshot showing S3 bucket policy](screenshots/s3_bucket_policy.png)

### 4.5 Content updates and invalidation
I edited index.html and re-uploaded it to S3 and then created an invalidation, waited for the status to be completed and then refreshed the distribution URL and confirmed the updated content was being served.

This demonstrates that the CloudFront caching layer can be refreshed on demand so site updates reach visitors promptly.

[Screenshot showing CloudFront Invalidation](screenshots/cloudfront_invalidation.png)

## 5. Security Considerations

Enabling S3 Public Access prevents accidental public exposure of objects via any policy or ACL change

Disabling ACLs	ensures access is managed purely through bucket policy.

OAC with SigV4 cryptographically signs every CloudFront to S3 request so S3 can verify the request genuinely came from CloudFront. This was chosen over the legacy OAI because it is the current AWS recommended approach and follows security best practices

The Bucket policy scope	grants only s3:GetObject permissions to cloudfront.amazonaws.com, with the AWS:SourceArn condition limiting access to this one distribution. 

Transport encryption (viewer → edge) redirects HTTP to HTTPS	so all visitors' traffic is encrypted in transit.

Transport encryption (edge → origin)	with HTTPS to S3 with SigV4 signing via OAC	protects data between CloudFront and S3

Encryption at rest with S3 default encryption (SSE-S3)	ensures objects are encrypted at rest with no extra configuration or key management.

## 6. Testing & Validation

1. When the CloudFront distribution URL was opened in a browser,	the homepage loads indicating that the site loads correctly
[Screenshot](screenshots/cloudfront_site_live.png)

2. When the site was visited via http://, the request was redirect to https://	indicating that redirects to HTTPS works correctly

3. When the S3 object URL was visited directly, an access denied message was receivedindicating that the bucket is private
[Screenshot](screenshots/s3_direct_access_denied.png)

## 7. Challenges Faced
### 7.1 CloudFront URL returned an error page
Symptom: Visiting the distribution domain name returned an error page (403 AccessDenied) instead of the homepage.

Reason: No default root object had been configured, so a request to / had no object to map to.

Fix: I configured the default root object in the distribution's behaviour settings.

### 7.2 Default root object value was incorrect
Symptom: After setting the default root object, the error persisted.

Cause: The value was entered as /index.html (with a leading slash).

Fix: I corrected it to index.html. After refreshing, the site loaded normally.

This was the main time sink in the project, several refreshes and a re-check of every prior configuration step before spotting the stray /. The lesson was to check the simplest, most literal details first (exact field values) before re-auditing the whole setup.

## 8. Key Takeaways
The S3 bucket can be made completely unreachable from the internet while its content is still served globally. CloudFront becomes the single controlled entry point, and OAC signing plus the _AWS:SourceArn_ condition ensures that only the intended distribution can read from it.

CloudFront caching is why invalidations exist; a site update in S3 is not immediately visible to visitors without one.

Small configuration details (like a leading / in the default root object) can cause failures that look like much bigger architecture problems.
