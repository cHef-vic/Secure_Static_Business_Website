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

[Screenshot showing CloudFrontwith OAC attached
](screenshots/cloudfront_oac.png)

[Screenshot showing CloudFront Behaviour](screenshots/cloudfront_behaviour.png)
