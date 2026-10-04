# Level 6 — IAM Permissions

## 🎯 Objective

> This level wants us to find the final hidden resource.

The main goal is to use the AWS credentials obtained from the previous level and investigate the permissions available to the IAM role.

---

## 🧠 What I Learned

* IAM roles can be used by AWS resources such as EC2 instances.
* IAM policies determine what actions an identity is allowed to perform.
* Having AWS credentials does not automatically mean that you have full access to an AWS account.
* An attacker can use `sts get-caller-identity` to determine which AWS identity they are currently using.
* IAM permissions should follow the principle of least privilege.
* A role with excessive permissions can allow an attacker who obtains its credentials to access resources they should not have access to.

---

## 🔎 Findings

### Finding 1

We already obtained temporary AWS credentials from the EC2 Instance Metadata Service during Level 5.

These credentials belong to the `flaws` IAM role.

We can confirm the identity with:

```bash
aws sts get-caller-identity --profile level5
```

This is useful because it tells us which AWS identity we are currently operating as.

---

### Finding 2

Since we have valid AWS credentials, we can investigate what permissions are available to the role.

Instead of assuming that the credentials have full access, we should enumerate the permissions that the role has been granted.

---

## 💥 Exploitation

### Step 1

First, we can confirm the current AWS identity.

```bash
aws sts get-caller-identity --profile level5
```

This confirms that the credentials belong to the `flaws` role.

---

### Step 2

We can inspect the IAM role and its policies.

```bash
aws iam list-attached-role-policies \
    --role-name flaws \
    --profile level5
```

This allows us to see policies attached directly to the role.

We can also inspect inline policies:

```bash
aws iam list-role-policies \
    --role-name flaws \
    --profile level5
```

---

### Step 3

Once we identify the relevant policy, we can retrieve its policy document.

```bash
# Command used
[command]
```

The policy shows which AWS actions the role is allowed to perform.

The important thing to look for is an overly broad permission that allows access to the target resource.

---

### Step 4

We can then use the permissions available to the role to enumerate the relevant AWS resource.

```bash
# Command used
[command]
```

**Result:**

```text
[relevant output]
```

This reveals the resource needed to complete the level.

---

### Step 5

We can access the discovered resource using the same credentials.

```bash
# Command used
[final command]
```

The response reveals the final information required by the challenge.

---

## 🏁 Solution

The credentials obtained from the EC2 Instance Metadata Service in Level 5 belong to the `flaws` IAM role.

By investigating the role's permissions, we can determine what AWS resources and actions are available to it.

The role has permissions that allow us to access the resource required by Level 6.

Using the role's temporary credentials, we can access the resource and retrieve the final information needed to complete the challenge.

---

## 🛡️ Security Lesson

### Vulnerability / Misconfiguration

**Overly permissive IAM permissions**

The IAM role has more permissions than are necessary for its intended purpose.

If an attacker obtains the role's temporary credentials, they can potentially use those permissions to access or modify AWS resources.

### Why it matters

Cloud credentials should always be treated as sensitive.

Even temporary credentials can be dangerous if the IAM role has excessive permissions.

For example, if an EC2 role has permissions to access sensitive S3 buckets, compromising the EC2 instance could lead to access to those buckets as well.

The attack chain can therefore look like:

```text
SSRF
 ↓
EC2 Metadata Service
 ↓
Temporary IAM Credentials
 ↓
Overly Permissive IAM Role
 ↓
Unauthorized AWS Resource Access
```

### How it could be prevented

* Follow the principle of least privilege.
* Only grant an IAM role the permissions required by the application.
* Avoid wildcard permissions such as `Action: "*"` or `Resource: "*"` unless absolutely necessary.
* Regularly audit IAM policies.
* Monitor AWS CloudTrail for unusual API activity.
* Rotate or revoke compromised credentials where appropriate.
* Use IMDSv2 and protect applications against SSRF.

---

## 📝 Key Takeaways

* **AWS services:** IAM / S3
* **Security concept:** IAM policies and least privilege
* **Vulnerability:** Excessive IAM permissions
* **Important technique:** Enumerating IAM role permissions
* **Main lesson:** Obtaining AWS credentials is only one part of an attack; the permissions associated with those credentials determine what an attacker can actually do.
* **Practical lesson:** IAM roles should have the minimum permissions required for their intended function.

---

## 🔗 References

* [flaws.cloud](http://flaws.cloud/)
* [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
* [IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
* [IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)
* [AWS S3 Documentation](https://docs.aws.amazon.com/AmazonS3/)
