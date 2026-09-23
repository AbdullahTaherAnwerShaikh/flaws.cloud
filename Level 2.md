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
aws s3 --profile YOUR_ACCOUNT ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud
```
this command lists the objects inside the bucket.



## 🏁 Solution

AWS explicitly defines an S3 Authenticated Users group. It means all AWS accounts, not just the bucket owner's account. But the request must be authenticated/signed with AWS credentials.
Your AWS account does not need to own the bucket or be specifically listed by account ID. That's the whole point of the AuthenticatedUsers group. AWS's documentation says that granting this group access allows any AWS account to access the resource, provided requests are authenticated.

## 🛡️ Security Lesson

### Vulnerability / Misconfiguration

Misconfiguring the AuthenticatedUsers allows any user to access publicly readable data, unintentionally storing sensitive data will jeopardize your files.


### Why it matters

If an S3 bucket is unintentionally misconfigured, an attacker may be able to enumerate or retrieve sensitive objects with authentication.

### How it could be prevented

* AWS specifically recommends that a public S3 static website grant anonymous users s3:GetObject, but not bucket-listing (s3:ListBucket) or write permissions.
* Don't grant s3:ListBucket to the AuthenticatedUsers.
* Use S3 Block Public Access where possible.

---

## 🔗 References

* [flaws.cloud](http://flaws.cloud/)
* [AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide)
