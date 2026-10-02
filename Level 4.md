# Level 4 — Unencrypted EC2

## 🎯 Objective

> This level wants us to find the subdomain.

The main goal is to discover information from an unencrypted EC2 instance and use it to identify the target subdomain.

---

## 🧠 What I Learned

* EC2 instances can expose sensitive information if they are not properly secured.
* Unencrypted data can potentially be accessed by someone who gains access to the underlying storage.
* Sensitive information stored on an instance should be protected with appropriate encryption and access controls.
* AWS resources should not rely solely on network-level security to protect sensitive data.

---

## 🔎 Findings

### Finding 1

The level involves an **unencrypted EC2 instance**. If an attacker is able to gain access to the underlying storage or snapshot containing the instance's data, they may be able to inspect files that were not adequately protected.

This can potentially reveal information that helps identify the subdomain or other sensitive information about the environment.

---

## 💥 Exploitation

### Step 1

Since we already have access to the account user which is a backup iam user, we can just use commands to display snapshots of the site.

```bash
# Command used
aws --profile [level 3 profile] ec2 describe-snapshots --region us-west-2 --owner-id 975426262029
```

This was used to display snapshots of the site and we can use that to recreate it before it was protected by credentials.
If you remove the owner id you'd be listed

AWS Account = entire AWS environment, identified by a 12-digit Account ID.
IAM User = a user/identity inside that account, e.g. backup.
Resources belong to the AWS account, not individual IAM users.
--owner-id = AWS Account ID, not username.
Without --owner-id, you can see snapshots from other accounts if they're public/shared.

Account = container. User = identity inside the container.

### Step 2

We then create an ec2 volume in our own account by copying the snapshot.

```bash
# Command used
aws --profile [profule] ec2 create-volume --availability-zone us-west-2a --region us-west-2 --snapshot-id [snapshot id]
```

This allowed us to copy the snapshot onto our account and let us start an ec2 instance which is connected to the volume, we then ssh into it.

### Step 3

We then ssh into the instance.

```bash
# Command used
 ssh -i [key pair file path] ec2-user@[public ip of instance]
```

### Step 4

We then need to list blocks so we can mount the volume.

```bash
# Command used
lsblk
sudo mkdir /mnt/snapshot
sudo mount /dev/nvme1n1p1 /mnt/snapshot
```

We can then navigate into home -> ubuntu -> setupNginx.sh to retrieve credentials.


---

## 🏁 Solution

After identifying the unencrypted EC2 storage, we were able to inspect the available data and find the information needed to identify the target subdomain.

The important lesson was that **encryption at rest is an important layer of protection for sensitive AWS data**. If storage containing sensitive information is left unencrypted, obtaining access to that storage can expose the underlying data.

---

## 🛡️ Security Lesson

### Vulnerability / Misconfiguration

**Unencrypted EC2 storage**

Data stored on EC2 instances should be protected with appropriate encryption. Leaving storage unencrypted increases the potential impact if an attacker gains unauthorized access to the underlying storage or a copy of it.

### Why it matters

If sensitive data is stored without encryption, an attacker who obtains access to the storage may be able to read the data directly.

This could expose:

* Credentials
* Configuration files
* Application data
* Internal information
* Other sensitive information stored on the instance

### How it could be prevented

* Enable encryption for EBS volumes.
* Use encrypted EBS snapshots.
* Follow the principle of least privilege for access to EC2 and EBS resources.
* Avoid storing secrets directly on instances when possible.
* Use AWS services such as AWS Secrets Manager or Systems Manager Parameter Store for sensitive configuration.
* Regularly review existing EC2 and EBS resources for encryption.

---

## 📝 Key Takeaways

* **AWS service:** EC2 / EBS
* **Security concept:** Encryption at rest
* **Vulnerability:** Unencrypted storage
* **Main lesson:** Encryption provides an additional layer of protection if storage is accessed without authorization.
* **Practical lesson:** Sensitive data should never rely on access controls alone for protection.

---

## 🔗 References

* [flaws.cloud](http://flaws.cloud/)
* [AWS Documentation](https://docs.aws.amazon.com/AmazonS3/)
* [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/)
* [Amazon EBS Encryption](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-encryption.html)
