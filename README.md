# Copilot Radar

Copilot Radar keeps track of every change Microsoft announces for **Microsoft 365 Copilot**, **Copilot Chat**, **Copilot Studio** and **Copilot and AI features in SharePoint**. Each day it collects updates from Microsoft's public sources, works out what has changed, and publishes them in three places:

- **A website** with a page for each product, showing what's new, what's arriving now and what needs action.
- **A weekly email** every Monday summarising the previous week.
- **An RSS feed** for anyone who wants updates in Outlook, Teams or a feed reader.

Everything runs automatically on GitHub. Nobody needs to start it, and no server needs looking after.

---

## Contents

- [Where to find it](#where-to-find-it)
- [Common tasks](#common-tasks)
- [How it works](#how-it-works)
- [What the website shows](#what-the-website-shows)
- [What the weekly email contains](#what-the-weekly-email-contains)
- [Where everything is stored](#where-everything-is-stored)
- [Settings reference](#settings-reference)
- [When something goes wrong](#when-something-goes-wrong)
- [For developers](#for-developers)

---

## Where to find it

| What | Where |
| --- | --- |
| Website | `https://<owner>.github.io/<repository-name>/`, for example `https://jayasekhon.github.io/Microsoft-Updates-Tracker/` |
| Past weekly emails | The **Weekly briefings** link on the website (`…/digests/`) |
| RSS feed | `…/feed.xml` on the website |
| Run history and logs | The **Actions** tab of this repository → **Tracker** |

---

## Common tasks

All of these are done on GitHub, in this repository. You need admin or write access.

### Add or remove people from the weekly email

Who gets the email is set by a setting called `DIGEST_TO`.

1. Go to **Settings → Secrets and variables → Actions → Secrets**.
2. Click the pencil next to `DIGEST_TO`.
3. Enter the full list of addresses, separated by commas, for example:
   ```
   team-list@company.com, first.person@company.com, second.person@company.com
   ```
4. Click **Update secret**. The next Monday email goes to the new list.

Things to know:

- **GitHub never shows the current list.** When you edit it, you must type the whole list again, not just the new address. Keep a note of who is on it.
- **Everyone in `DIGEST_TO` can see each other's addresses**, because they all appear on the To line. To hide addresses, put them in `DIGEST_BCC` instead (same steps, same format).
- **The easiest option is a single distribution list.** If `DIGEST_TO` contains just one Outlook distribution list or Microsoft 365 group address, people can be added and removed in Outlook, and nobody needs to touch GitHub. The list must allow email from **outside the organisation**, because the email is sent from a Gmail account. Without that setting, Microsoft 365 rejects the email.
- **If an address stops working** (for example, someone leaves), everyone else still gets the email. A "delivery failed" notice appears in the sending Gmail account's inbox. Remove that address from the list.

### Send a test email to yourself

1. Go to **Actions → Tracker → Run workflow**.
2. Type your email address in **test_to**. Leave the tick boxes empty.
3. Click **Run workflow**.

Only you receive it, with `[TEST]` at the start of the subject line. It covers the most recent full Monday-to-Sunday week. A run takes between a few minutes and about 25 minutes.

### Send this week's email again, or send it now

1. Go to **Actions → Tracker → Run workflow**.
2. Tick **send_digest**.
3. If this week's email has already gone out, also tick **force**. Without it, the system won't send the same week twice.
4. Click **Run workflow**.

### Update the website straight away

Go to **Actions → Tracker → Run workflow**, leave everything empty, and click **Run workflow**. The website normally updates itself once a day.

### Change what is tracked

The products, the keywords used to decide what belongs to each product, and the blog feeds are all in [`config/sources.json`](config/sources.json). For example, to stop tracking a feed, delete its entry from the `feeds` list. To exclude another kind of Roadmap item, add its tag to `exclude_tag_prefixes`. Changes take effect on the next run.

---

## How it works

```
 Microsoft 365 Roadmap ─┐
                        ├─► 1. Collect ─► 2. Compare with ─► 3. Summarise ─► 4. Publish website
 Microsoft blogs (RSS) ─┘                  yesterday           with AI          (and on Mondays,
                                                                                 send the email)
```

**Every day** (scheduled for 05:17 UTC), a GitHub Actions workflow:

1. **Collects** everything from the [Microsoft 365 Roadmap](https://www.microsoft.com/microsoft-365/roadmap) and a set of official Microsoft blogs, then keeps only what belongs to the four products.
2. **Compares** it with the saved copy from the previous run, and records what changed and when: new items, status changes (for example *In development → Rolling out*), date changes, and items removed from the Roadmap. This history is what makes "This week" and the weekly email possible. The Roadmap itself only shows each item's current state.
3. **Summarises** new or changed items with AI (Google Gemini). It writes a plain-English summary, who is affected, whether action is needed, and a suggested next step. Each item is summarised once, and again only if Microsoft rewrites it.
4. **Publishes** the website, then saves the updated data back into this repository.

**Every Monday** (scheduled for 06:43 UTC), it does all of the above, then also builds the briefing for the previous Monday to Sunday, saves a copy on the website, and emails it.

**Scheduled runs often start late.** GitHub usually starts scheduled runs some time after the set time, sometimes by several hours when it's busy. Times are in UTC.

**If the AI is unavailable**, nothing breaks. The system tries a lighter backup model, and if that is busy too, it uses Microsoft's own wording until a later run can summarise the item.

---

## What the website shows

**Homepage:** a radar showing the latest changes, a one-line summary of the week, and a rotating carousel with one card per product. Each card shows three numbers: changes this week, items rolling out now, and items needing action. Click a card, or drag the carousel to turn it. At the bottom is the full searchable list of every update, with filters and three views:

- **Feed:** updates grouped by week.
- **Board:** columns for In development, Rolling out and Launched.
- **Release timeline:** updates grouped by target month.

**Product pages** (`topic.html?p=m365`, `chat`, `studio` or `sharepoint`) show one product, most important first:

| Section | What it lists |
| --- | --- |
| Needs action | Items that need an admin or a heads-up for users, and have arrived (see below) |
| This week | Everything that changed in the last 7 days |
| Rolling out now | Features arriving in organisations now |
| Coming soon | Features in development that are due in the next three months |
| Recently launched | Features that became generally available recently |
| From Microsoft's blogs | Announcements from the last 60 days |

Clicking any item opens a panel with the full Microsoft description, key dates, its history of changes, and a link to the Microsoft source.

### What the labels mean

| Label | Meaning |
| --- | --- |
| **In development / Rolling out / Launched** | Microsoft's status from the Roadmap. |
| **Likely rolled out** | Microsoft still says "Rolling out", but its target date was more than 2 months ago. Microsoft often doesn't update finished items, so these are treated as launched. |
| **Admin action** | An IT admin needs to set up, enable, disable, license or review something, or a default setting is changing. |
| **Prepare users** | It changes how people already work (something is removed, moved or behaves differently), or something new switches on for everyone by default. |
| **Needs action** | An item labelled *Admin action* or *Prepare users* that has arrived: it is rolling out now, or launched in the last 30 days. Items still in development never count. Blog posts never count either, although they may show a "suggested next step". |
| **New** | Added to the Roadmap in the last week or so. |

---

## What the weekly email contains

Sent on Mondays, covering the previous Monday to Sunday:

1. **Four headline numbers:** now available, newly announced, date changes, need action.
2. **Needs attention this week:** items that need action and started rolling out or launched that week. Each item appears here only once, in the week it arrives.
3. **Now available, Newly announced, Timeline changes, From the Microsoft blogs, Cancelled or removed:** one card per item, with a summary, key dates, who is affected, a **View in tracker** button and a **Read on Microsoft** link. Empty sections are left out.
4. **Coming in the next 30 days:** features still in development that are due within the next month or so.

The email is sent from the Gmail account in the settings, with the sender name **Copilot Radar**. A plain-text version is included for email apps that don't show formatting. Each week's email is also saved on the website under **Weekly briefings**.

---

## Where everything is stored

| What | Where | Notes |
| --- | --- | --- |
| Email recipients, passwords and keys | **Settings → Secrets and variables → Actions** in this repository | Secrets can be replaced but never viewed. See [Settings reference](#settings-reference). |
| Sending account | A Gmail account | Its address and an *App password* are stored as secrets. |
| AI key | A Google AI Studio account (<https://aistudio.google.com/apikey>) | Stored as the `GEMINI_API_KEY` secret. The free tier is enough. |
| What is tracked | [`config/sources.json`](config/sources.json) | Products, keywords, blog feeds and exclusions. |
| Tracked data and history | [`data/items.json`](data/items.json) | Every item and its full history of changes. Updated by each run. **Don't delete it:** without it, the history of past changes is lost. |
| Data the website reads | `data/site.json` | Rebuilt from `items.json` on every run. |
| Past weekly emails | [`digests/`](digests/) | One HTML file per week, plus `index.json`, which also records which weeks have been sent. |
| Schedule and steps | [`.github/workflows/tracker.yml`](.github/workflows/tracker.yml) | Change the `cron` lines to change when it runs. |
| Website files | [`site/`](site/) | Plain HTML, CSS and JavaScript. No build step. |
| Program code | [`tracker/`](tracker/) | Python, using only the standard library. |

**Accounts to hand over** when ownership changes: the GitHub repository (admin access), the Gmail account used for sending, and the Google account that owns the AI key. After a handover, create a new Gmail App password and a new AI key under the new owner, and update the secrets.

**Failure notifications:** GitHub emails a failed scheduled run to the person who last edited the `cron` lines in the workflow file. A new owner who wants these emails should make a small edit to those lines (for example, nudge a time by a minute) and save it.

---

## Settings reference

Set in **Settings → Secrets and variables → Actions**.

### Secrets (hidden once saved)

| Name | Required | What to put in it |
| --- | --- | --- |
| `GMAIL_ADDRESS` | For email | The Gmail address that sends the email. |
| `GMAIL_APP_PASSWORD` | For email | A 16-character **App password** for that account (not its normal password). The account needs 2-Step Verification turned on first. Create one at <https://myaccount.google.com/apppasswords>. |
| `DIGEST_TO` | For email | Recipients, separated by commas. Ideally one distribution list address. |
| `DIGEST_BCC` | No | Hidden recipients, separated by commas. |
| `GEMINI_API_KEY` | For AI summaries | A Google AI Studio API key. Without it, the system still works, but uses Microsoft's wording and no action labels. |

### Variables (optional, visible to people with repository access)

| Name | Default | What it changes |
| --- | --- | --- |
| `SITE_URL` | `https://jayasekhon.github.io/Microsoft-Updates-Tracker/` | The website address used in email links. **Set this if the repository is moved or renamed**, or the links in the email will point to the old address. |
| `DIGEST_FROM_NAME` | `Copilot Radar` | The sender name on the email. |
| `GEMINI_MODEL` | `gemini-flash-latest` | The main AI model. |
| `GEMINI_FALLBACK_MODEL` | `gemini-flash-lite-latest` | The backup model, used when the main one is busy. |
| `AI_MAX_ITEMS` | `200` | The most items summarised in one run. A run also stops AI work after 20 minutes. |
| `AI_PROVIDER` | `gemini` when `GEMINI_API_KEY` is set, otherwise none | Set to `bedrock` to use AWS Bedrock instead of Gemini. This also needs `BEDROCK_MODEL_ID`, `AWS_REGION` and `AWS_ROLE_ARN` (an IAM role that GitHub can sign in to, with permission to call the model). |

### Website hosting

GitHub Pages must be set to **Settings → Pages → Build and deployment → Source: GitHub Actions**.

---

## When something goes wrong

Open **Actions → Tracker**, click the failed (red) run, then the step with the red cross, to read the message.

| What you see | What it means | What to do |
| --- | --- | --- |
| **Email briefing** step fails with "Email not configured" | `GMAIL_ADDRESS` or `GMAIL_APP_PASSWORD` is missing. | Add both secrets. Check the names are spelled exactly. |
| **Email briefing** step fails with "Username and Password not accepted" | Google rejected the App password. | Create a new App password and update `GMAIL_APP_PASSWORD`. Check 2-Step Verification is still on. |
| "No recipients" | `DIGEST_TO` and `DIGEST_BCC` are both empty. | Add recipients (see [Common tasks](#common-tasks)). |
| The run is green but nobody got the email | Gmail sent it, but it was blocked or filtered on the way. | Check Junk and the company's email quarantine. Check the sending Gmail account's **Sent** folder (to confirm the recipients) and its inbox (for "delivery failed" notices). Ask IT to allow the sending address. |
| "This week's briefing was already sent" | The email for that week has gone out already. | Run again with **send_digest** and **force** ticked to resend. |
| **Fetch Microsoft sources** step fails, with `[roadmap] ERROR` in the log | Microsoft's Roadmap was unreachable. | Usually temporary. The next daily run catches up, or run it again by hand. |
| "batch failed" or "is busy" messages in the log | The AI service was overloaded. | No action needed. Those items keep Microsoft's wording until a later run summarises them. |
| `[rss] note:` or `[rss] warning:` lines in the log | One blog feed couldn't be read. | No action needed unless it keeps happening. Then check or remove that feed in `config/sources.json`. |
| **Deploy to GitHub Pages** fails | Pages isn't set up for GitHub Actions. | See [Website hosting](#website-hosting). |
| Runs stop happening on their own | GitHub pauses schedules on repositories with no activity for 60 days. | Re-enable the workflow from the Actions tab. |

---

## For developers

### Running locally

Requires Python 3.10 or newer. No packages need installing (`boto3` is needed only for AWS Bedrock).

```bash
python -m tracker update                        # collect, compare and summarise (AI only if GEMINI_API_KEY is set)
python -m tracker update --no-ai                # the same, without AI
python -m tracker digest --preview preview.html # write last week's email to a file to look at
python -m tracker build                         # assemble the website into _site/
python -m http.server -d _site 8000             # view it at http://localhost:8000
python -m unittest discover -s tests -t .       # run the tests
```

The tests also run automatically on every push (the **Tests** workflow).

### Folder map

```
.github/workflows/
  tracker.yml       daily and Monday runs: collect, summarise, publish, email
  tests.yml         runs the tests on every push
config/sources.json what is tracked: products, keywords, feeds, exclusions
tracker/            the Python program
  sources/          reading the Microsoft Roadmap and blog feeds
  classify.py       deciding which product an item belongs to
  store.py          saving items and recording what changed
  summarize.py      AI summaries and action labels (Gemini or AWS Bedrock)
  digest.py         building the weekly email and its archive copy
  mailer.py         sending the email through Gmail
  __main__.py       the commands: update, build, digest
site/               the website
  index.html        homepage
  topic.html        product pages
  digests.html      weekly briefings archive page
  assets/           styles, scripts and the product illustrations
data/               tracked data, updated by every run
digests/            saved weekly emails
tests/              automated tests
```
