# Email Signature Updater

*Complete Click-by-Click Setup Guide*
*No experience needed. No development knowledge required.*

---

## 📑 Table of Contents

* [Before You Start — What You Need](#before-you-start--what-you-need)
* [Part 1: GitHub Setup](#part-1-github-setup)
* [Part 2: Cloud Flow](#part-2-cloud-flow)
* [Part 3: Desktop Flow](#part-3-desktop-flow)
* [Part 4: Connect the Flows](#part-4-connect-the-flows)
* [Part 5: Prepare Your Signature File](#part-5-prepare-your-signature-file)
* [Part 6: Test Everything](#part-6-test-everything)
* [Daily Use](#daily-use--how-to-add-articles)
* [Troubleshooting](#troubleshooting)

---

## ✓ Before You Start — What You Need

* [ ] **GitHub account**
* [ ] **Power Automate Premium license**
* [ ] **Power Automate Desktop installed**
* [ ] **Outlook signature file location**

> 💡 **Tip:** You'll create a GitHub token later—save it when you do.

---

# 📚 PART 1: GitHub Setup

## 1️⃣ Create Account

Go to `github.com` → Sign up → verify email

---

## 2️⃣ Create Repository

* Name: `outlook-sig-articles`
* Visibility: Private

---

## 🏷️ Labels

Create:

* `unshared` (green)
* `shared` (gray)

---

## 📝 Add Articles

Each article = Issue

* Title → headline
* Body → URL only
* Label → `unshared`

---

## 🔑 Token

Create a **fine-grained personal access token**

* Repo access: `outlook-sig-articles`
* Permissions: Issues (Read & Write)

---

# ☁️ PART 2: Cloud Flow

## Schedule

* Runs daily at **5 AM**

---

## HTTP Request

```bash
https://api.github.com/repos/USERNAME/outlook-sig-articles/issues?labels=unshared&state=open&sort=created&direction=asc&per_page=1
```

Headers:

| Key           | Value                       |
| ------------- | --------------------------- |
| Accept        | application/vnd.github+json |
| Authorization | Bearer YOUR_TOKEN           |
| User-Agent    | PowerAutomate               |

---

## Parse JSON

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "number": { "type": "integer" },
      "title": { "type": "string" },
      "body": { "type": "string" }
    }
  }
}
```

---

# 🖥️ PART 3: Desktop Flow

## Variables

* `ArticleURL`
* `ArticleTitle`

---

## File Path

```
C:\Users\YOURNAME\AppData\Roaming\Microsoft\Signatures\your-signature.htm
```

---

## Remove Old Article

```
<!--ARTICLE-START-->.*?<!--ARTICLE-END-->
```

---

## Insert HTML

```html
<!--ARTICLE-START--><p style="margin: 10px 0 0 0; font-family: Segoe UI, sans-serif; font-size: 10pt;"><span style="font-weight: bold;">I enjoyed this article and I think you will too!</span><br><a href="%ArticleURL%" style="color: #0563C1;">%ArticleTitle%</a></p><!--ARTICLE-END-->
```

---

# 🔗 PART 4: Connect Flows

* Add **Run Desktop Flow**
* Map:

  * URL → `ArticleURL`
  * Title → `ArticleTitle`

---

# ✏️ PART 5: Signature Setup

Add before `</body>`:

```html
<!--ARTICLE-START--><!--ARTICLE-END-->
```

---

# 🧪 PART 6: Testing

* Run flow manually
* Check GitHub labels
* Check signature file
* Test in Outlook

---

# 📅 Daily Use

1. Create GitHub issue
2. Add headline + URL
3. Label `unshared`

Done—updates automatically.

---

# 🔧 Troubleshooting

**401 error** → token issue
**404 error** → repo/username wrong
**No update** → PC asleep or Outlook cached

---

# 🎉 Done

Every day:

* Pulls article
* Updates signature
* Marks as shared

Fully automated.
