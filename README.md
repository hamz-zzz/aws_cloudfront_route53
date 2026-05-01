# AWS CloudFront + Route 53 Static Website (Private S3 Origin)

## Architecture Diagram

![Architecture](architecture/aws_cloudfront_route53_architecture.svg)

This project implements a secure, globally distributed static website using Amazon Route 53, CloudFront, and Amazon S3. Traffic is routed through a custom domain and resolved via DNS, delivered through CloudFront edge locations over HTTPS, and served from a private S3 bucket. The architecture enforces a secure access model using Origin Access Control (OAC), preventing direct access to the origin and ensuring all content is delivered through the CDN.

---

## Key Design Decisions

- **Custom Domain (Route 53)**
  - Domain: `systemsbyhamza.com`
  - Hosted zone manages DNS resolution
  - Alias records map domain → CloudFront distribution

- **S3 Private Origin**
  - Static content stored in S3 bucket
  - Block Public Access fully enabled
  - No direct public access to objects

- **CloudFront Distribution**
  - Acts as global CDN and entry point
  - Handles caching, HTTPS, and origin communication
  - Default root object: `index.html`

- **Origin Access Control (OAC)**
  - CloudFront signs requests to S3
  - Bucket policy allows access only from CloudFront
  - Enforces least-privilege access model

- **HTTPS (ACM)**
  - Certificate issued in `us-east-1`
  - Covers:
    - `systemsbyhamza.com`
    - `www.systemsbyhamza.com`
  - Attached to CloudFront for TLS termination

- **DNS Dual Stack (IPv4 + IPv6)**
  - A (IPv4) and AAAA (IPv6) alias records configured
  - Enables global dual-stack connectivity

- **Caching Strategy**
  - CloudFront caches content at edge locations
  - Cache lifecycle validated using headers and invalidation

---

## Deployment Steps

### 1. Domain & DNS (Route 53)

1. Registered domain `systemsbyhamza.com`  
2. Created hosted zone and verified NS/SOA records  
3. Configured DNS records:
   - A (alias) → CloudFront  
   - AAAA (alias) → CloudFront  
   - CNAME (`www`) → root domain  

---

### 2. Storage Layer (S3)

1. Created S3 bucket for static content  
2. Enabled:
   - Block Public Access (all settings)  
   - Default encryption  
3. Uploaded:
   - `index.html`  
   - `/images` directory (Ubuntu, Fedora, Arch images)  

---

### 3. Security Layer (OAC)

1. Created CloudFront Origin Access Control  
2. Attached OAC to S3 origin  
3. Applied bucket policy:
   - Allows only CloudFront distribution access  
4. Verified:
   - Direct S3 access returns AccessDenied  

---

### 4. Edge Layer (CloudFront)

1. Created CloudFront distribution  
2. Configured:
   - Origin: S3 bucket (not website endpoint)  
   - Viewer protocol policy: HTTP → HTTPS redirect  
   - Default root object: `index.html`  
3. Attached ACM certificate  
4. Added alternate domain names  

---

### 5. HTTPS (ACM)

1. Requested certificate in `us-east-1`  
2. Domains:
   - `systemsbyhamza.com`  
   - `www.systemsbyhamza.com`  
3. Validated via DNS records in Route 53  
4. Attached certificate to CloudFront  

---

## DNS Configuration

### Domain Registration

![Domain Registered](screenshots/1-1-domain-registered.png)

### Hosted Zone

![Hosted Zone](screenshots/1-2-hosted-zone-overview.png)

### A Record (IPv4)

![A Record](screenshots/1-3-root-alias-a-record.png)

### AAAA Record (IPv6)

![AAAA Record](screenshots/1-4-root-alias-aaaa-record.png)

### WWW CNAME

![CNAME](screenshots/1-5-www-cname-record.png)

- Route 53 resolves the domain to CloudFront  
- Dual-stack DNS enables IPv4 and IPv6 access  

---

## Storage Layer (S3)

### Bucket Overview

![Bucket](screenshots/2-1-s3-bucket-overview.png)

### Public Access Blocked

![Block Public](screenshots/2-2-s3-block-public-access.png)

### Object Structure

![Objects](screenshots/2-3-s3-objects-structure.png)

### Images

![Images](screenshots/2-4-s3-images-folder.png)

### Bucket Policy

![Policy](screenshots/2-5-s3-bucket-policy.png)

### Access Denied Validation

![AccessDenied](screenshots/2-6-s3-access-denied-test.png)

- Confirms S3 is private  
- Only CloudFront can access content  

---

## CloudFront Configuration

### Distribution Overview

![Distribution](screenshots/3-1-cloudfront-distribution-overview.png)

### Origin Configuration

![Origin](screenshots/3-2-cloudfront-origin-config.png)

### Behavior Settings

![Behavior](screenshots/3-3-cloudfront-behavior-settings.png)

- CloudFront acts as secure edge layer  
- Enforces HTTPS and caching  

---

## HTTPS (ACM)

### Certificate

![Certificate](screenshots/4-1-acm-certificate-issued.png)

- Certificate successfully issued and attached  
- Enables encrypted communication  

---

## Application Layer

### Homepage

![Homepage](screenshots/5-1-site-homepage.png)

### HTTPS Validation

![HTTPS](screenshots/5-2-site-https-cert.png)

### Static Asset Delivery

![Images](screenshots/5-3-image-direct-load.png)

- Content delivered via CloudFront  
- Images served from S3 through CDN  

---

## Caching Behavior

### Cache Miss

![Miss](screenshots/6-1-cache-miss.png)

### Cache Hit

![Hit](screenshots/6-2-cache-hit.png)

### Invalidation

![Invalidation](screenshots/6-3-cache-invalidation.png)

### Cache Reset

![Reset](screenshots/6-4-cache-after-invalidation.png)

- Demonstrates CloudFront caching lifecycle  
- Shows edge cache behavior and refresh  

---

## DNS Resolution

### nslookup

![nslookup](screenshots/7-1-nslookup-output.png)

### dig

![dig](screenshots/7-2-dig-output.png)

- Multiple IPs returned → CloudFront edge network  
- Confirms Anycast routing and global distribution  

---

## Access Paths

### CloudFront Domain

![CloudFront](screenshots/8-1-cloudfront-domain-access.png)

- Direct access via CloudFront distribution domain  
- Confirms CDN is serving content independently of DNS layer  

---

### Root Domain

![Root Domain](screenshots/5-1-site-homepage.png)

- Access via `https://systemsbyhamza.com`  
- Resolved using Route 53 alias A/AAAA records → CloudFront  

---

### WWW Subdomain (CNAME Resolution)

![WWW Domain](screenshots/8-2-www-domain-access.png)

- Access via `https://www.systemsbyhamza.com`  
- Resolved using Route 53 CNAME record → root domain  
- Confirms subdomain routing and DNS aliasing behavior  

---

- Application accessible via:
  - CloudFront domain  
  - Root domain (`systemsbyhamza.com`)  
  - WWW subdomain (`www.systemsbyhamza.com`)  

---

## What This Project Demonstrates

- DNS resolution and routing using Route 53  
- CDN-based content delivery using CloudFront  
- Secure origin design using S3 + OAC  
- HTTPS implementation using ACM  
- Edge caching and performance optimization  
- IPv4 and IPv6 dual-stack networking  
- Private vs public access enforcement  
- Multi-service AWS architecture integration  

---

## Supporting Artifacts

This repository includes supporting materials used to validate and document the deployment:

- **[configs/](./configs/)**
  - `index.html`  
  - Image assets used for content delivery testing  

- **[screenshots/](./screenshots/)**
  - DNS configuration  
  - S3 security validation  
  - CloudFront setup  
  - HTTPS validation  
  - Caching behavior  
  - DNS resolution and access paths  

These artifacts provide reproducibility and verification of the implemented architecture.