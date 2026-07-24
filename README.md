# Founder Fork — HTML Publisher

Publish a styled HTML page to a live, shareable URL by asking Claude. No local setup, no build step, no deploy button. You write the document in a conversation, Claude pushes it, and about thirty seconds later it's a link you can send.

Any page can optionally be locked behind a username and password.

---

## What this does

- Turns a conversation into a live web page with a permanent URL
- Keeps every page you've ever published, indefinitely, each at its own address
- Gives you one index page listing everything you've published, updated automatically
- Lets you password-protect individual documents, one at a time
- Updating a page keeps the same URL — send the link once, change the contents later

## What this does not do

Worth knowing up front so nothing surprises you:

- **It is not a website builder.** Pages are standalone documents, not a navigable site with menus.
- **It has no per-person access control.** Protection is a shared username and password. Anyone you give them to can pass them along, and you can't see who opened the page. If you need to know exactly who viewed a document, use a tool built for that.
- **Unprotected pages are public.** Anyone with the URL can read them. There's no "unlisted" middle ground — a page is either open or password-protected.
- **It doesn't handle payments, forms, logins, or databases.** It serves documents.
- **It won't tell you when someone opens a link.** No read receipts, no analytics.

---

## What you need before you start

1. A **GitHub account** — this is where your documents live
2. A **Railway account** — this is what serves them to the internet (a free account is fine to begin with)
3. **Claude** with the GitHub connector enabled — this is what publishes for you

Optionally, the Railway connector too. It isn't required, but without it you'll set passwords by hand in the Railway dashboard instead of just asking. Both paths are documented below.

---

## Setup

Roughly fifteen minutes, once.

### 1. Fork this repository

At the top right of this page, click **Fork**, then **Create fork**. You now have your own copy under your account. Everything from here happens in your copy.

Keep it a fork rather than downloading the files — that connection is what lets you pull in improvements later with one click. See *Getting updates* below.

### 2. Turn on Actions — don't skip this

GitHub switches off automation in forked repositories by default. If you skip this step, your index page will stay empty forever and nothing will tell you why.

In **your fork**, click the **Actions** tab. You'll see a notice about workflows being disabled. Click the green **"I understand my workflows, go ahead and enable them"** button.

That's it. This is the single easiest step to miss and the one most likely to make you think the tool is broken.

### 3. Deploy it to Railway

In a new browser tab, go to **railway.app** and sign in.

1. Click **New Project**
2. Choose **Deploy from GitHub repo**
3. Select your fork
4. Railway detects Node automatically — no configuration needed

Wait for the first deploy to finish. It takes a minute or two.

### 4. Give it a web address

Still in Railway, click into the service, then **Settings → Networking → Generate Domain**.

Copy the URL it gives you. It'll look something like:

```
https://founder-fork-html-publisher-production.up.railway.app
```

**Save this URL somewhere you'll find it.** It's both the home of everything you publish and your index page — see below.

### 5. Check that it works

Paste that URL into your browser. You should see an index page — empty, because you haven't published anything yet. If you see it, setup worked.

### 6. Tell Claude where to publish

Claude needs to know your repository and your Railway URL. Give it both, and it will handle publishing from then on.

### 7. Make it yours (optional)

Open **`site.config.json`** and edit it — your name, the page title, colours, and the group names your documents get sorted into.

That one file holds every setting. Nothing else needs touching, and keeping your settings there is what lets you accept future updates without conflicts.

---

## Getting updates

This tool will keep improving. Bugs get found, features get added. Here's how you take them.

### The short version

When there's an update, your fork's page on GitHub shows a line like *"This branch is 3 commits behind."* Next to it is a **Sync fork** button.

1. Go to your fork on GitHub
2. Click **Sync fork**
3. Click **Update branch**

Railway redeploys automatically. That's the whole thing — no files to download, nothing to copy, nothing to reinstall. Your documents, your settings, and your passwords are all untouched.

### How you'll know an update exists

You'll be told directly. You can also get notified automatically: go to the **original repository**, click **Watch** at the top right, choose **Custom**, and tick **Releases**. GitHub will email you whenever a new version ships.

### Why this won't break your stuff

The repository is deliberately split into files that are *yours* and files that are *the tool's*:

| Yours — never updated | The tool's — safely updated |
|---|---|
| `proposals/` — your documents | `server.js` |
| `site.config.json` — your settings | `scripts/generate-index.js` |
| `protected.json` — what's protected | `.github/workflows/` |
| `index.html` — your index | `README.md` |

Updates only touch the right-hand column. That's why your settings live in `site.config.json` rather than inside the script — so improvements to the script never collide with your branding.

### If Sync fork reports a conflict

Rare, and it only happens if you've hand-edited a file in the right-hand column. GitHub will offer to open a pull request instead of syncing directly.

Don't try to untangle it yourself. Say what happened and it'll be sorted out for you — the fix is usually one line.

### One case worth knowing

If a conflict is reported on `index.html`, it's safe to take whichever version and move on. That file is regenerated from scratch on every push, so any wrong choice corrects itself the next time you publish something.

---

## Your index page

**Your index is the Railway URL by itself**, with nothing added to the end:

```
https://your-project-production.up.railway.app
```

There is nothing to activate and no separate address. It's live from the moment you deploy, and it's empty until you publish something.

Every document you publish adds a card to it automatically — title, slug, date, and a "Protected" badge if it's password-protected. Documents are sorted into the groups you configured, newest first, and there's a search box at the top that filters as you type.

A few things worth knowing about it:

- **It updates on its own.** Publishing a document triggers a GitHub Action that rebuilds the page. Give it a minute or two, then refresh.
- **It's public.** Anyone with the URL sees the list of everything you've published, including the titles of protected documents. Clicking a protected one still asks for credentials — but the *name* is visible. Bear that in mind when naming sensitive documents.
- **You can edit it by hand.** Open `index.html` and change any title or grouping. Your edits survive every future rebuild.
- Bookmark it. It's the only address you need to find anything you've ever published.

---

## Publishing a document

Ask Claude for what you want. When it's ready to publish, it will confirm two things with you:

**The slug** — the last part of the URL. The convention is:

```
[who-or-what]-[context]-[MMYY]

client-partnership-0726
internal-q3-review-0826
```

Slugs are permanent. Pick something you'd still recognise a year from now.

**Whether it needs protecting** — Claude will ask:

> Does this file need privacy or gated access? (I'll install a username and password to protect it.) Default is open / not password protected.

Answer, and it publishes. You get the link back in the conversation.

---

## Making a document private

Say yes when asked, or ask later — protecting a page after it's already published works exactly the same, and the URL doesn't change.

Here's what happens underneath:

1. The slug gets added to the `gated` list in `protected.json`. That file records *which* documents are protected. It never contains passwords.
2. A username and password are generated for that specific document.
3. The credentials get stored in Railway as an environment variable named after the document. For a document with the slug `client-partnership-0726`, the variable is:

   ```
   GATE_CLIENT_PARTNERSHIP_0726  =  client_partnership_0726:GeneratedPassword
   ```

   The name is the slug, uppercased, with dashes turned into underscores.

4. Claude gives you the username and password in the conversation. **That's the only place they appear in readable form** — send them to your recipient separately from the link.

**If you have the Railway connector enabled**, Claude does step 3 for you and you don't have to think about it.

**If you don't**, Claude will tell you the exact variable name and value, and you add it yourself: Railway → your service → **Variables** → **New Variable** → paste the name and value → the service redeploys itself in under a minute.

### What your recipient sees

A browser password box, before the page renders. Not a styled login screen — the browser's own prompt. Their password manager can save it, and they'll only be asked once per session.

### Each document has its own password

Credentials for one document do not open any other. Sharing one page's password gives away that page and nothing else.

### Removing protection

Ask Claude to unprotect it. The slug comes out of `protected.json` and the page is open again. Same URL throughout.

---

## The metadata comment

Every published page should carry a comment on its first line:

```html
<!-- meta: group=client; title=Partnership Overview; date=2026-07 -->
```

This controls how the document appears on your index — its display name, which group it's filed under, and its date. Claude adds it automatically. Without it, the index guesses from the slug, and the guess is usually worse.

`group` should match one of the keys in `site.config.json`. Anything unrecognised falls into your catch-all group.

---

## Things worth knowing

**The first index entry sticks.** The index is deliberately non-destructive: once a document appears on it, its name and group are never recalculated, so any manual correction you make survives forever. The flip side is that if a page is published with a broken or missing metadata comment, the guessed name is baked in. Fixing it means editing `index.html` directly, once.

**Protected pages still appear on the index**, marked with a "Protected" badge. Clicking one prompts for credentials. The title is visible to anyone who visits your index — so if the *name* of a document is itself sensitive, name it carefully.

**Protection fails closed.** If a document is marked protected but its Railway variable is missing or malformed, the page is denied to everyone rather than served openly. A configuration mistake locks people out; it never leaks the document.

**Passwords are recoverable.** They live in your Railway variables. Months later, if someone asks what the password was, you can look it up rather than issuing a new one.

**Folders work too.** A directory under `proposals/` containing an `index.html` is published as a single document, and protecting it protects everything inside it.

---

## Costs

GitHub is free for this. Railway charges by usage, and this is a very small always-on service — check their current pricing for what that works out to. There is no cost per document or per visitor.

---

## Troubleshooting

**My index page is empty even though I've published documents.** Almost always Actions being switched off on your fork — see setup step 2. Go to the **Actions** tab of your repository and enable workflows. Then publish anything, or make any small edit to `protected.json`, to trigger a rebuild.

**The index doesn't show a document I just published.** The index regenerates through a GitHub Action, which takes a minute or two after the push. Refresh. If it still hasn't appeared, check the Actions tab for a failed run.

**A protected page says "not currently available" instead of asking for a password.** The Railway variable for that document is missing or malformed. Check that its name matches the slug exactly — uppercase, dashes turned into underscores — and that the value contains a colon between username and password.

**Everything returns "temporarily unavailable."** `protected.json` has a syntax error. The server seals all documents rather than risk exposing a protected one. Fix the JSON and it recovers on the next deploy.

**My branding didn't change.** `site.config.json` has a syntax error, so the tool fell back to defaults. Check for a missing comma or quote. The Actions log will say so explicitly.

**A page shows raw HTML code instead of a formatted page.** The file was published with its markup escaped. Republish it.

---

## What's in here

```
server.js                        serves documents, enforces protection
site.config.json                 YOUR settings — name, colours, groups
protected.json                   which documents are protected (no passwords)
proposals/                       every published document
index.html                       auto-generated index, safe to hand-edit
scripts/generate-index.js        builds the index
.github/workflows/               regenerates the index on every push
```
