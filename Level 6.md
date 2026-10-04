# Level 6 — IAM Permissions & S3 Enumeration

## 🎯 Objective

> This level wants us to find the hidden directory in the Level 6 S3 bucket.

The main goal is to use the temporary AWS credentials obtained from the previous level to interact with the Level 6 S3 bucket and discover the hidden directory.

---

## 🧠 What I Learned

* AWS IAM roles can provide temporary credentials to applications and EC2 instances.
* Temporary credentials can be used with the AWS CLI just like other AWS credentials, but they also require a session token.
* An IAM identity can have permissions to access AWS resources without being an IAM user.
* S3 bucket names and object paths can sometimes reveal useful information during enumeration.
* Having valid AWS credentials does not automatically mean that you have permission to access everything in an AWS account.
* `aws sts get-caller-identity` can be used to determine which AWS identity is currently being used.
* S3 prefixes can behave like directories even though S3 fundamentally stores objects in a flat namespace.

---

## 🔎 Findings

### Finding 1

From Level 5, we obtained temporary AWS credentials from the EC2 Instance Metadata Service.

These credentials belonged to the IAM role attached to the EC2 instance.

Instead of using our original IAM user, we can use these temporary credentials to interact with AWS as the `flaws` role.

### Finding 2

The Level 6 bucket is:

```text
level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud
```

The objective is to enumerate the bucket and find the hidden directory.

---

## 💥 Exploitation

### Step 1 — Verify our AWS identity

Before interacting with the bucket, we can verify which AWS identity our profile is using.

```bash
aws sts get-caller-identity --profile level5
```

The command uses **AWS STS (Security Token Service)** to return information about the identity associated with the credentials.

The response contains information such as:

```text
Account
Arn
UserId
```

The `Arn` allows us to identify the IAM role being used.

This is useful because it confirms that the AWS CLI is actually using the temporary credentials obtained from Level 5.

---

### Step 2 — List the Level 6 bucket

We can now attempt to list the contents of the Level 6 S3 bucket.

```bash
aws s3 ls s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud --profile level5
```

### Breaking down the command

```text
aws
```

Runs the AWS CLI.

```text
s3
```

Selects the S3 service.

```text
ls
```

Lists buckets or objects.

```text
s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud
```

Specifies the S3 bucket we want to inspect.

```text
--profile level5
```

Tells the AWS CLI to use the credentials stored in the `level5` profile instead of the default profile.

The result reveals an unexpected directory/prefix.

---

### Step 3 — Inspect the discovered directory

Suppose the listing reveals:

```text
ddcc78ff/
```

We can enumerate that prefix by adding it to the S3 path:

```bash
aws s3 ls s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/ddcc78ff/ --profile level5
```

The trailing `/` is important because it tells S3 that we are interested in objects under that prefix.

The command allows us to inspect what exists inside the discovered directory.

---

### Step 4 — Access the discovered resource

We can also use the S3 URL directly:

```text
http://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/ddcc78ff/
```

This allows us to access the hidden directory through the Level 6 website.

---

## 🏁 Solution

The Level 6 solution follows directly from the previous level.

In Level 5, we exploited an SSRF vulnerability to access the EC2 Instance Metadata Service and retrieve temporary credentials belonging to the `flaws` IAM role.

We then used those credentials with the AWS CLI.

First, we verified the identity:

```bash
aws sts get-caller-identity --profile level5
```

Then we enumerated the Level 6 bucket:

```bash
aws s3 ls s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud --profile level5
```

The bucket revealed the hidden prefix:

```text
ddcc78ff/
```

We could then inspect it:

```bash
aws s3 ls s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/ddcc78ff/ --profile level5
```

The hidden directory provides the final information required by the challenge.

---

## 🛡️ Security Lesson

### Vulnerability / Misconfiguration

**Excessive AWS permissions combined with exposed temporary credentials**

The previous level demonstrated how an SSRF vulnerability could expose credentials associated with an EC2 IAM role.

Once those credentials were obtained, their permissions determined what AWS resources could be accessed.

This demonstrates why **least privilege** is important for IAM roles.

### Why it matters

An attacker does not necessarily need an administrator account to cause damage.

If an exposed IAM role has permission to access sensitive S3 buckets, an attacker who obtains its credentials may be able to:

* List objects
* Download sensitive files
* Modify objects
* Delete objects
* Access other AWS resources allowed by the role

The impact depends on the permissions attached to the compromised identity.

### How it could be prevented

* Apply least-privilege permissions to IAM roles.
* Avoid granting broad S3 permissions when they are unnecessary.
* Protect EC2 metadata from SSRF attacks.
* Prefer IMDSv2.
* Protect applications against SSRF.
* Monitor unusual AWS API activity.
* Rotate or revoke credentials when exposure is detected.
* Use CloudTrail to monitor API activity.
* Regularly review IAM policies and role permissions.

---

## 📝 Key Takeaways

* **AWS services:** IAM / S3 / STS
* **Security concept:** IAM roles and temporary credentials
* **Technique:** S3 enumeration
* **Important command:** `aws sts get-caller-identity`
* **Main lesson:** The permissions of an IAM role determine what an attacker can do if its credentials are compromised.
* **Practical lesson:** Cloud credentials should always be treated as sensitive secrets, including temporary credentials.

---

## 🔗 References

* [flaws.cloud](http://flaws.cloud/)
* [AWS CLI Documentation](https://docs.aws.amazon.com/cli/)
* [AWS STS Documentation](https://docs.aws.amazon.com/STS/latest/APIReference/)
* [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
* [Amazon S3 Documentation](https://docs.aws.amazon.com/s3/)
