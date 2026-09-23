# Level 2 — Misconfigured Bucket Permissions

## 🎯 Objective

> This level wants us to find sub-domain like the previous level

We are trying to find a sub-domain that could lead us to level 2. In level 1, we were accessing it unauthenticated meaning we didn't need to authenticate using an aws account.
This level is different from that.

---

## 🧠 What I Learned

* S3 bucket permissions
* Bucket enumeration
* Misconfigured resources


---

## 🔎 Findings

Document the important evidence you discovered.

### Finding 1

In the hints, it revealed we could access the directory list by using an aws account meaning authenticated access.
Using authenticated access we can view the list without our account being associated to the bucket or owner of the bucket.


---

## 💥 Exploitation

### Step 1

```bash
# Command used
nsloopup flaws.cloud
```




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
