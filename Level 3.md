# Level 3 — Leaked Access Key

## 🎯 Objective

> This level wants us to find sub-domain

Git history contains every line you edit, if unintentionally a private key was saved then someone can use the git history to view and access them.

---

## 🧠 What I Learned

* S3 bucket listing is open for everyone
* Anyone can view and figure out the git history is stored


---

## 🔎 Findings

Document the important evidence you discovered.

### Finding 1

Since the bucket listing is allowed to everyone just like in level 1, we can view the objects stored which revealed there's a git directory meaning we can access git history and potentially reveal access keys.

---

## 💥 Exploitation

### Step 1

```bash
# Command used
aws s3 ls s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud --profile [YOUR_ACCOUNT]
```
To inspect which files are contained in the bucket, which revealed the git directory.


### Step 2

```bash
# Command used
ws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ . --no-sign-request
```
We can use sync to sync the files to the current directory. "." is used as current directory. 


### Step 3

```bash
# Command used
git log
```
To list out every commit made on the repository.


### Step 4

```bash
# Command used
git diff [commit id]
```
To provide the details of the changes made.


## 🏁 Solution

After syncing the bucket we can inspect the git history.  
Access the previous commit and use the access keys to make another profile.

## 🛡️ Security Lesson

### Vulnerability / Misconfiguration

People often leak AWS keys and then try to cover up their mistakes without revoking the keys. You should always revoke any AWS keys (or any secrets) that could have been leaked or were misplaced. Roll your secrets early and often.

### Why it matters

Hackers could misuse your access keys and be able to tamper with your account as well as racking up bills.

### How it could be prevented

* If a key is ever misplaced or leaked, revoke that key and make a new one.
* Roll your keys early and often.

---

## 🔗 References

* [flaws.cloud](http://flaws.cloud/)
* [AWS documentation](https://docs.aws.amazon.com/AmazonS3)
