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
