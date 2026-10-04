# Level 6 — IAM Policy and API Gateway

## 🎯 Objective

> This level gives us AWS credentials for an IAM user with the `SecurityAudit` policy attached. We need to see what else this user can access and find the final level.

The main goal is to inspect the permissions of the provided IAM user and discover an additional policy that gives us access to an API Gateway endpoint.

---

## 🧠 What I Learned

* The `SecurityAudit` policy gives an IAM user read-only access to a large amount of AWS configuration information.
* Read-only permissions can still be dangerous because they can reveal information about the AWS environment.
* IAM users can have multiple policies attached to them.
* IAM policies contain the actual permissions granted to an identity.
* API Gateway can be used to expose and invoke APIs.
* API Gateway can trigger Lambda functions.
* AWS resources can be connected together, so enumerating one service can lead to another service.

---

## 🔎 Findings

### Finding 1

The Level 6 website provides us with an **Access Key ID** and **Secret Access Key**.

We can use these credentials to create an AWS CLI profile and investigate what permissions this user has.

### Finding 2

The user has the `SecurityAudit` policy attached to it.

The `SecurityAudit` policy is intended for security auditing and gives the user permissions to view security configuration information.

However, there is another policy attached to the user called:

```text
list_apigateways
```

This policy gives us additional permissions that are not part of the normal `SecurityAudit` policy.

This is the important finding that allows us to continue.

---

## 💥 Exploitation

### Step 1 — Configure the credentials

The Level 6 page gives us an Access Key ID and Secret Access Key.

We can create a new AWS CLI profile using:

```bash
aws configure --profile level6
```

This allows us to enter the credentials and keep them separate from our other AWS profiles.

We can then use `--profile level6` whenever we want to use these credentials.

---

### Step 2 — Verify the credentials

```bash
aws sts get-caller-identity --profile level6
```

`sts get-caller-identity` tells us which AWS identity the credentials belong to.

The result shows that we are using the `Level6` IAM user.

This is basically the AWS equivalent of checking **whoami**.

---

### Step 3 — Find the policies attached to the user

```bash
aws iam list-attached-user-policies --user-name Level6 --profile level6
```

This command lists the IAM policies directly attached to the `Level6` user.

The result shows two policies:

```text
MySecurityAudit
list_apigateways
```

`MySecurityAudit` is the auditing policy we expected.

The interesting one is:

```text
list_apigateways
```

This tells us that the user has additional permissions specifically related to API Gateway.

---

### Step 4 — Inspect the `list_apigateways` policy

First, we need to get information about the policy:

```bash
aws iam get-policy \
    --policy-arn arn:aws:iam::975426262029:policy/list_apigateways \
    --profile level6
```

This command gives us information about the policy.

The important part is:

```text
DefaultVersionId: v4
```

The policy version tells us which version contains the permissions currently being used.

---

### Step 5 — Read the actual permissions

Now that we know the policy version is `v4`, we can retrieve the actual policy document:

```bash
aws iam get-policy-version \
    --policy-arn arn:aws:iam::975426262029:policy/list_apigateways \
    --version-id v4 \
    --profile level6
```

This shows us what actions the policy actually allows.

The policy gives us permissions to enumerate API Gateway resources.

This is important because now we know that there may be an API Gateway endpoint somewhere in the AWS account.

---

### Step 6 — Find the API Gateway

We can list the REST APIs:

```bash
aws apigateway get-rest-apis --profile level6 --region us-west-2
```

This shows the REST APIs available to us.

Among the results we find:

```text
s33ppypa75
```

This is the **REST API ID**.

We can also inspect the resources belonging to the API:

```bash
aws apigateway get-resources \
    --rest-api-id s33ppypa75 \
    --profile level6 \
    --region us-west-2
```

The result shows a resource called:

```text
/level6
```

So we now know that the API has a `/level6` endpoint.

---

### Step 7 — Find the API stage

We still need the API's stage name.

We can get it with:

```bash
aws apigateway get-stages \
    --rest-api-id s33ppypa75 \
    --profile level6 \
    --region us-west-2
```

The result shows:

```text
Prod
```

So we now have all the information required to construct the API URL:

```text
API ID  = s33ppypa75
Region  = us-west-2
Stage   = Prod
Endpoint = /level6
```

---

### Step 8 — Access the API

AWS API Gateway REST APIs use the following general format:

```text
https://API-ID.execute-api.REGION.amazonaws.com/STAGE/
```

Using the information we discovered, the URL becomes:

```text
https://s33ppypa75.execute-api.us-west-2.amazonaws.com/Prod/level6
```

We can access it using a browser or `curl`:

```bash
curl https://s33ppypa75.execute-api.us-west-2.amazonaws.com/Prod/level6
```

The API returns:

```text
Go to http://theend-797237e8ada164bf9f12cebf93b282cf.flaws.cloud/d730aa2b/
```

This is the final URL.

---


## 🛡️ Security Lesson

### Vulnerability / Misconfiguration

**Over-permissioned IAM user**

The user was supposed to have the `SecurityAudit` policy, but it also had an additional policy that provided API Gateway permissions.

This additional permission allowed us to discover resources that were not necessary for simply performing a security audit.

### Why it matters

Read-only permissions can still expose useful information to an attacker.

An attacker who can enumerate an AWS environment may discover:

* API Gateway endpoints
* Lambda functions
* IAM policies
* AWS resources
* Resource relationships
* Other information that can be used to find further vulnerabilities

This level demonstrates that **read permissions should not automatically be considered harmless**.

### How it could be prevented

* Apply the principle of least privilege.
* Only attach policies that are actually required.
* Regularly review IAM policies attached to users.
* Remove unnecessary or outdated policies.
* Avoid giving users access to AWS resources unrelated to their role.
* Monitor IAM policy changes.

---

## 📝 Key Takeaways

* **AWS services:** IAM / API Gateway / Lambda
* **Security concept:** IAM permissions and enumeration
* **Vulnerability:** Excessive permissions
* **Important command:** `aws iam list-attached-user-policies`
* **Main lesson:** Read-only permissions can still reveal valuable information about an AWS environment.
* **Practical lesson:** IAM users should only receive the permissions they actually need.

---

## 🔗 References

* [flaws.cloud](http://flaws.cloud/)
* [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
* [AWS API Gateway Documentation](https://docs.aws.amazon.com/apigateway/)
* [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
