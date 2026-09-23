# Level 1 — Bucket Misconfiguration

## 🎯 Objective

> This level wants us to find sub-domain

We are trying to find a sub-domain that could lead us to level 2. When hosting a static site using s3 buckets, the domain name must match the bucket. S3 bucekts are global, meaning no 2 buckets can have the same name.
We are essentially looking into other resources contained in the S3 bucket that has been unknowingly exposed.

---

## 🧠 What I Learned

* S3 bucket permissions
* Bucket enumeration
* Misconfigured resources


---

## 🔎 Findings

Document the important evidence you discovered.

### Finding 1

By using the dns forward & reverse lookup, I found out the site is a static site hosted on an S3 storage.
Since its an s3 bucket, we can tweak the url and access 


---

## 💥 Exploitation

### Step 1

```bash
# Command used
nsloopup flaws.cloud
```
To find out which ip address(es) the domain resolves to but not all of them. "Where does this name go?"


### Step 2

```bash
# Command used
host [ip address of the domain]
```
To find out the hostname, it return PTR record if any. "Who does this IP belong to?"
A hostname is a name given to a particular host (computer/server/network device or service endpoint) on a network.



## 🏁 Solution

http://flaws.cloud.s3-website-us-west-2.amazonaws.com/ 
http://flaws.cloud.s3.amazonaws.com
The two URLs are different S3 endpoints that talk to the same bucket in different ways.
The GET request to the website endpoint returns the index document that is specified in the website configuration.
Whereas as the Get request at the REST API endpoint returns the list of the object keys in the bucket.

## 🛡️ Security Lesson

### Vulnerability / Misconfiguration

S3 static website URL can be altered to allow us to access through REST API which allows us to view the list of object keys in the bucket and can access these objects using the URL.


### Why it matters

If an S3 bucket is unintentionally exposed, an attacker may be able to enumerate or retrieve sensitive objects without authentication.

### How it could be prevented

* AWS specifically recommends that a public S3 static website grant anonymous users s3:GetObject, but not bucket-listing (s3:ListBucket) or write permissions.
* Don't grant s3:ListBucket to the public.
* Use S3 Block Public Access where possible.

---

## 🔗 References

* [flaws.cloud](http://flaws.cloud/)
* [AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteEndpoints.html?utm_source=chatgpt.com)
