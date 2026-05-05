# Automated Email Signature Updater – Outlook Classic  
_Design + Configuration (Single Document, Re-creatable)_

## Purpose
Provide a single document that combines architecture/design and concrete configuration details (Power Automate Cloud Flow + Power Automate Desktop) so another person can recreate this solution in their environment for Outlook Classic on Windows.

**Source grounding:**  
Configuration details are taken from your enterprise guide step-by-step.htm. Where that guide explicitly omits certain cloud-flow steps, this document includes a clearly marked recommended completion pattern.

---

## 1. Overview

The solution updates a user’s Outlook Classic (desktop) signature by editing the local signature HTML file.

A scheduled Power Automate cloud flow:
- Selects one article (title + URL) from a GitHub issue queue
- Triggers a Power Automate Desktop flow to inject the article link into the signature file

---

## 2. Architecture

### Components
- GitHub repository used as an article queue (Issues)
- Power Automate Cloud Flow for orchestration
- Power Automate Desktop Flow for local execution
- Outlook Classic signature file (.htm) in AppData (Roaming)

### 2.1 End-to-end flow

**Cloud Flow (scheduled)**  
1. Get oldest GitHub issue labeled `unshared`  
2. Parse JSON response  
3. Extract IssueNumber, ArticleTitle, ArticleURL  
4. Trigger Desktop Flow (attended)  
5. Mark item as shared  

**Desktop Flow (local)**  
6. Read signature HTML file  
7. Remove prior injected block between markers  
8. Insert new block  
9. Save file  

---

## 3. Prerequisites

- GitHub account and repository  
- Power Automate (cloud) with HTTP action  
- Power Automate Desktop installed  
- Machine must be awake and user signed in (attended mode)  
- Outlook Classic for Windows with file-based signatures  

---

## 4. GitHub Configuration (Article Queue)

### 4.1 Repository
Create a private repository named:
```
outlook-sig-articles
```

### 4.2 Labels
- unshared  
- shared  

### 4.3 Add Articles as Issues
- Issue Title: headline (used as link text)  
- Issue Body: URL only (single line)  
- Apply label: `unshared`

### 4.4 Personal Access Token
Create a fine-grained PAT:
- Scope: repository  
- Permissions: Issues (Read and write)

---

## 5. Power Automate Cloud Flow (Scheduled Orchestrator)

**Flow Name:**  
Daily Article Signature Update

### 5.1 Trigger: Recurrence
Run daily at 5:00 AM (local timezone)

---

### 5.2 HTTP Action: Get One Unshared Article

```
GET https://api.github.com/repos/<GITHUB_USERNAME>/outlook-sig-articles/issues?labels=unshared&state=open&sort=created&direction=asc&per_page=1
```

**Headers**
```
Accept: application/vnd.github+json
Authorization: Bearer <GITHUB_PAT>
User-Agent: PowerAutomate
```

---

### 5.3 Parse JSON

**Schema**
```
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

### 5.4 Remaining Cloud Flow Steps (Recommended Pattern)

#### 5.4.1 Condition: Stop if No Issues

```
@greater(length(body('Parse_JSON')), 0)
```

- No → Terminate  
- Yes → Continue  

---

#### 5.4.2 Extract Values

```
IssueNumber:
@first(body('Parse_JSON'))?['number']

ArticleTitle:
@first(body('Parse_JSON'))?['title']

ArticleURL:
@trim(first(body('Parse_JSON'))?['body'])
```

---

#### 5.4.3 Run Desktop Flow

- Action: Run a flow built with Power Automate Desktop  
- Flow: Update Outlook Signature  
- Mode: Attended  
- Inputs:
  - ArticleTitle  
  - ArticleURL  

---

#### 5.4.4 Mark Issue as Shared

```
PUT https://api.github.com/repos/<GITHUB_USERNAME>/outlook-sig-articles/issues/@{outputs('IssueNumber')}/labels
```

**Body**
```
{
  "labels": ["shared"]
}
```

---

## 6. Power Automate Desktop Flow (Local Signature Update)

**Flow Name:**  
Update Outlook Signature

**Inputs**
- ArticleURL  
- ArticleTitle  

---

### 6.1 Set Signature File Path

Example:
```
C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Signatures\<SIGNATURE_NAME>.htm
```

Shortcut:
```
%APPDATA%\Microsoft\Signatures
```

---

### 6.2 Read File

- Action: Read text from file (UTF-8)  
- Output: CurrentSignature  

---

### 6.3 Remove Old Article Block

Regex:
```
<!--ARTICLE-START-->.*?<!--ARTICLE-END-->
```

Replace with:
```
(empty)
```

---

### 6.4 Build New HTML Block

```
<!--ARTICLE-START-->
<p style="margin:0;">
I enjoyed this article and I think you will too:
<a href="%ArticleURL%">%ArticleTitle%</a>
</p>
<!--ARTICLE-END-->
```

- Store in variable: ArticleHTML  

---

### 6.5 Insert and Save

Replace:
```
</body>
```

With:
```
%ArticleHTML%</body>
```

- Write file (overwrite, UTF-8)

---

## 7. One-time Signature File Preparation

Add markers to your signature file:

```
<!--ARTICLE-START-->
<!--ARTICLE-END-->
```

Only content between markers is managed by automation.

---

## 8. Testing

1. Add unshared GitHub issues  
2. Run cloud flow manually  
3. Verify signature file updated  
4. Open new Outlook email  

Note: Restart Outlook if signature does not refresh

---

## 9. Troubleshooting

### Cloud Flow
- 401 Unauthorized → Invalid or expired PAT  
- 404 Not Found → Repo name or permissions issue  

### Desktop Flow
- Not running → Machine asleep or locked  
- Old content remains → Marker or regex mismatch  

---

## Appendix A – Copy/Paste Blocks

### A.1 GitHub Issues GET URI
```
https://api.github.com/repos/<GITHUB_USERNAME>/outlook-sig-articles/issues?labels=unshared&state=open&sort=created&direction=asc&per_page=1
```

---

### A.2 Parse JSON Schema
```
{ "type": "array", "items": { "type": "object", "properties": { "number": { "type": "integer" }, "title": { "type": "string" }, "body": { "type": "string" } } } }
```

---

### A.3 Regex Pattern
```
<!--ARTICLE-START-->.*?<!--ARTICLE-END-->
```

---

### A.4 HTML Injection Block
```
<!--ARTICLE-START-->
<p style="margin:0;">
I enjoyed this article and I think you will too:
<a href="%ArticleURL%">%ArticleTitle%</a>
</p>
<!--ARTICLE-END-->
```
