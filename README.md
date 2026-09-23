# flaws.cloud — AWS Security Write-ups

My write-ups and notes while working through [flaws.cloud](http://flaws.cloud/), an AWS security challenge focused on discovering and exploiting common AWS misconfigurations.

The goal of this repository is to document **my methodology, commands, findings, and lessons learned** while solving each level.

> ⚠️ **Spoiler Warning:** These write-ups contain solutions and walkthroughs for the flaws.cloud levels.

---

## 🎯 Goals

Through these challenges, I am practicing:

* AWS security fundamentals
* Amazon S3 enumeration
* AWS resource discovery
* Access control and permissions
* Misconfigured AWS services
* Credential exposure
* Reconnaissance and enumeration
* Security-focused command-line usage
* Understanding how AWS misconfigurations can be exploited


---

## 🛠️ Tools

Tools used throughout the challenges include:

* Linux
* Bash
* `curl`
* `wget`
* `nslookup`
* AWS CLI
* Browser developer tools
* Other AWS/security tools where appropriate

---

## 🧠 Methodology

For each level, I try to document the process rather than only recording the final answer:

```text
Reconnaissance
      ↓
Enumeration
      ↓
Identify interesting behavior
      ↓
Form a hypothesis
      ↓
Test the hypothesis
      ↓
Exploit the misconfiguration
      ↓
Capture evidence
      ↓
Understand the security impact
      ↓
Document the lesson
```

The write-ups focus on **why** a particular technique works, not just which command was executed.

---

## 📚 What I'm Learning

Some of the main AWS/security concepts covered so far include:

* S3 buckets and objects
* S3 bucket URLs
* Bucket listing
* Public vs. authenticated access
* AWS permissions
* AWS resource enumeration
* Information disclosure
* Misconfigured cloud storage
* AWS CLI usage
* DNS and AWS infrastructure

---

## ⚠️ Disclaimer

This repository is for **educational purposes**.

The techniques documented here are performed against intentionally vulnerable environments such as flaws.cloud. They should not be used against AWS resources or other systems without explicit authorization.

---

## 🔗 Resources

* [flaws.cloud](http://flaws.cloud/)
* [AWS Documentation](https://docs.aws.amazon.com/)
* [AWS CLI Documentation](https://docs.aws.amazon.com/cli/)

---

## 📈 Progress

This repository is part of my ongoing cybersecurity learning journey, with a focus on practical hands-on security skills and understanding how cloud infrastructure can be misconfigured and exploited.
