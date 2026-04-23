I've been looking for a way to share knowledge more consistently with my peers—without it becoming just another task on my plate.

So I built a system:

📚 I save interesting articles to a GitHub repo and label them "unshared".
🔄 A Power Automate cloud flow runs daily, picks a random article, and triggers a desktop flow
✍️ The flow updates my Outlook signature with that day's article link
💌 Every email I send now includes the article link + a note: "I really enjoyed this and I think you will too!"
Here's the secret sauce:

The Two Pieces (Cloud vs Desktop)
Cloud Flow (Runs in your web browser at make.powerautomate.com)
This is the "orchestrator" It:

Runs on a schedule (5 AM every day)
Talks to GitHub to fetch an article
Tells your computer to update the signature file
Marks the article as "shared"
Desktop Flow (Runs on your PC via Power Automate Desktop)
This is the "worker" It:

Receives instructions from the cloud flow
Opens your Roaming Outlook signature file
Removes yesterday's article link
Inserts today's article link
Saves the file
Think of it like an assistant sitting at your computer who reads your instructions and manually edits a file.

How They Work Together
Cloud Flow (5 AM)
    ↓
1. Check GitHub for unshared articles
    ↓
2. Extract article URL and title
    ↓
3. Remove "unshared" label, add "shared" label
    ↓
4. Pass the URL and title to Desktop Flow
    ↓
Desktop Flow (on your PC)
    ↓
5. Read your signature file from disk
    ↓
6. Remove old article block (between HTML comment markers)
    ↓
7. Insert new article block with today's URL
    ↓
8. Save the file
    ↓
Done! Your next email will include the new article.
The Three Key Technologies
GitHub Repository Issues = Your Article Library
Each article you want to share becomes a GitHub "Issue" in your repo.

Issue Title: The article headline (clickable link text)
Issue Body: Just the article URL (nothing else)
Issue Label "unshared": Marks articles you haven't shared yet
Issue Label "shared": Marks articles that have already been shared.
Example:

Title: "Why Remote Work Is Here to Stay"
Body: https://example.com/remote-work-article
Label: [unshared]
Once shared, the label changes to [shared] and the flow won't pick it again.

Cloud Flow = The Scheduler & GitHub Coordinator
Built in make.powerautomate.com, this flow:

Waits for the scheduled time (5 AM daily)
Calls GitHub's API to say: "Give me one open issue with the 'unshared' label, oldest first"
GitHub sends back the issue data (number, title, URL)
The flow reads that data
The flow calls GitHub again to remove the "unshared" label
The flow calls GitHub again to add the "shared" label
The flow hands the URL and title to the desktop flow
Why HTTP/API? The cloud flow talks to GitHub using HTTP actions (web requests). Each request is like sending a message that says:

GET: "Give me the unshared articles"
DELETE: "Remove the 'unshared' label from issue #42"
POST: "Add the 'shared' label to issue #42"
Desktop Flow = The File Editor
Built in Power Automate Desktop (the app on your PC), this flow:

Receives the article URL and title from the cloud flow
Opens your signature file: C:\Users\YourName\AppData\Roaming\Microsoft\Signatures\your-signature.htm
Reads the entire file into memory
Uses regex (pattern matching) to find and delete the old article block (between <!--ARTICLE-START--> and <!--ARTICLE-END--> markers)
Builds new HTML containing the article link
Inserts it before the closing </body> tag
Writes the updated file back to disk
Why markers? The HTML comment markers (<!--ARTICLE-START--> and <!--ARTICLE-END-->) act as anchors. Instead of trying to match complex HTML styles (which break easily), the flow just deletes everything between the markers and inserts new content. It's foolproof.

The Setup Journey (What You Just Did)
Step 1: Prepare GitHub
Create a private repository called outlook-sig-articles
Create two labels: unshared and shared
Add your first few articles as Issues
Step 2: Create the Cloud Flow
Go to make.powerautomate.com
Create a "Scheduled cloud flow"
Set the recurrence: Every day at 5 AM
Add HTTP actions to talk to GitHub:
GET to fetch an unshared issue
DELETE to remove the "unshared" label
POST to add the "shared" label
Add Compose actions to extract the URL, title, and issue number from GitHub's response
Add a condition to check if any articles exist (guard against empty results)
Step 3: Create the Desktop Flow
Open Power Automate Desktop on your PC
Create a flow called "Update Outlook Signature"
Define two input variables: ArticleURL and ArticleTitle
Add steps to:
Read your signature file from disk
Remove the old article block using regex
Build new article HTML
Insert it before </body>
Write the file back
Step 4: Connect Them
In the cloud flow, add a "Run a desktop flow" action
Select your desktop flow
Map the input variables: feed the ArticleURL and ArticleTitle from the Compose steps
Step 5: Seed the Signature File
Open your signature .htm file in Notepad
Add <!--ARTICLE-START--><!--ARTICLE-END--> right before </body>
This is the one-time setup — the flow uses these markers every day
What Happens When It Runs
Day 1 — 5:00 AM:

Cloud flow wakes up, checks GitHub, finds Issue #1: "Remote Work Article"
Cloud flow removes "unshared" label, adds "shared" label
Cloud flow sends URL and title to desktop flow
Desktop flow reads signature file, inserts the article block, saves the file
You send an email — it includes: "I enjoyed this article and I think you will too! [Remote Work Article](https://example.com/..."
Day 2 — 5:00 AM:

Cloud flow checks GitHub again
Issue #1 has the "shared" label, so it's skipped
Issue #2 is now the oldest "unshared" one, so it gets picked
Desktop flow removes the old article block and inserts the new one
Your next email shows: "I enjoyed this article and I think you will too! [New Article Title](https://example.com/..."
Day 3, 4, 5... — Same pattern continues, rotating through your library

Common Gotchas (What We Fixed)
Problem 1: The Desktop Flow Didn't Show Up in the Dropdown
The cloud flow needs to see that the desktop flow has input variables defined. Without them, the flow won't show in the dropdown even if it exists. Solution: Define ArticleURL and ArticleTitle as input variables in the desktop flow.

Problem 2: The GitHub API Calls Returned 404 (Not Found)
The URL was supposed to have the issue number plugged in (like /issues/42/labels/unshared), but instead it was sending the literal expression text (like /issues/first(body...)/labels). Solution: Use concat() expressions to build the URL, which forces Power Automate to evaluate variables at runtime instead of treating them as literal text.

Problem 3: Running Out of Articles
If all articles are marked "shared," the GitHub API returns an empty list. The Compose step tries to call first() on an empty array and crashes. Solution: Add a Condition step that checks length(body('Parse_JSON')) > 0 and exits gracefully if there are no unshared articles.

Problem 4: The Old Article Didn't Get Removed
The original flow tried to match HTML styles with regex, which broke if the signature's HTML structure changed even slightly. Solution: Use HTML comment markers (<!--ARTICLE-START--> and <!--ARTICLE-END-->) as deterministic anchors that never change.

Daily Workflow — Adding New Articles
Find an article you like
Go to your GitHub repo → Issues → New issue
Title: Paste the article headline
Body: Paste just the URL
Labels: Select unshared
Click Submit new issue
That's it. The flow handles the rest tomorrow morning. Check the shared label filter to see your history of recommended articles.

Why This Design?
Why split into Cloud + Desktop?

The cloud flow runs on Microsoft's servers on a schedule, so it works even if your PC is asleep
The desktop flow only runs when needed (when there's actually an article to share)
The cloud flow handles all the "talking to GitHub" complexity
The desktop flow is simple — just read a file, edit it, write it back
Why GitHub Issues?

Free storage for your article library
Built-in labels for status tracking
Simple, human-readable interface (no database to manage)
You can see your sharing history at a glance
Why HTML comment markers?

They're invisible in the rendered signature
They're deterministic (never change)
They survive Outlook re-formatting
The regex pattern <!--ARTICLE-START-->.*?<!--ARTICLE-END--> will always find exactly what you want to replace
The Endgame
Every morning at 5 AM, without you lifting a finger:

A fresh article appears in your signature
Your colleagues see a thoughtful recommendation
The system tracks what you've shared
You build a routine of sharing valuable content
It's automation in its purest form — set it up once, then forget about it.

If you want to see the Step-by-Step process - click here https://github.com/ScottPriestley/email-signature-updater/blob/main/step-by-step.md
