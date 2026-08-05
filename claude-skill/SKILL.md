---
name: publish-document
description: >
  Turns a document built in conversation into a live, shareable web link. Use this skill ANY TIME
  the user wants to:
  - Publish, ship, or post a document, one-pager, proposal, overview, or summary
  - Get a link they can send to a client, partner, or candidate
  - Put something online, make it shareable, or "make it a page"
  - Password protect or unprotect a document they have already published
  - Update the contents of a page they published before
  Trigger phrases: "publish this", "make this a link", "get me a link", "send this to a client",
  "put this online", "make it shareable", "ship it", "password protect it", "make it private",
  "update that page", "what have I published".
  Handles the full loop: draft in conversation, user approves, push to GitHub, Railway auto-deploys,
  live URL returned in chat. The user never opens GitHub or Railway.
---

# Publish Document

Publishes a standalone HTML document to a permanent, shareable URL. The user works entirely in
conversation. No deploy button, no dashboard, no file downloads.

---

## Before you use this: fill in two values

**This file is a template. Do not edit it inside your repository.** Copy it out, edit the copy,
and upload that copy to Claude. Leaving the version in your repo untouched is what lets you accept
future updates with `Sync fork` without hitting a conflict.

Copy this file, then replace both placeholders in the table below:

| Placeholder | Where to find it |
|---|---|
| `YOUR_GITHUB_USERNAME` | Your GitHub username. It's in your fork's URL: `github.com/YOUR_GITHUB_USERNAME/founder-fork-html-publisher` |
| `YOUR_RAILWAY_DOMAIN` | The domain from setup step 4. Railway, your service, Settings, Networking, Generate Domain. Paste it without `https://` and without a trailing slash |

Replace every occurrence of both. There are several of each.

Then upload the edited copy to Claude: **Settings, Capabilities, Skills, Upload skill.**

---

## Infrastructure

| What | Value |
|---|---|
| GitHub repo | `YOUR_GITHUB_USERNAME/founder-fork-html-publisher` |
| Railway domain | `https://YOUR_RAILWAY_DOMAIN` |
| Document path | `proposals/[slug].html` |
| Live URL pattern | `https://YOUR_RAILWAY_DOMAIN/proposals/[slug].html` |
| Index page | `https://YOUR_RAILWAY_DOMAIN` |
| Auto-deploy | On. Every push to `main` redeploys Railway in about 30 seconds |
| Index rebuild | Separate GitHub Action, 1 to 2 minutes after the push |

The document itself is live in about 30 seconds. It takes another minute or so to appear on the
index. Hand the user the direct link immediately. Do not wait on the index.

---

## Prerequisites

This skill pushes files using the user's **GitHub connector**. If that connector is not live,
nothing here works. If a push fails with a permissions error or "tool not found", stop and say so.
Do not attempt a workaround.

The **Railway connector is optional**. It only changes how password protection is applied:

- **Railway connector on:** set the password variable directly, the user does nothing
- **Railway connector off:** give the user the exact variable name and value to paste into Railway

---

## Slug rules

The slug is the last part of the URL and it is permanent.

**Format:** `[who-or-what]-[context]-[MMYY]`

Examples:
- `client-partnership-0726`
- `internal-q3-review-0826`
- `acme-scope-of-work-0826`

Rules:
- Lowercase, dashes only, no spaces or underscores
- Never reuse a slug for different content. Reusing it overwrites the existing page
- Pick something recognizable a year from now

Always propose a slug and confirm it before pushing.

---

## The metadata comment

**Every published document must begin with this comment on line one.** Without it the index guesses
the title from the slug, and that guess becomes permanent once it lands.

```html
<!-- meta: group=client; title=Partnership Overview; date=2026-07 -->
```

- `group` must match a key in `site.config.json`. Defaults are `client`, `internal`, `other`
- `title` is the display name on the index card
- `date` is `YYYY-MM`

If the user has customized `site.config.json`, read it before assuming the default group keys still
apply.

---

## Workflow

### Step 1. Draft in conversation

Build a complete, standalone HTML file. No external dependencies except Google Fonts. All CSS
inline in a `<style>` block.

Match the user's brand. If their colors are unknown, ask once and record them in this file.

### Step 2. Show it and get approval

Show the document for review. Do not push until the user explicitly approves.

### Step 3. Confirm the slug

Propose a slug. Confirm before pushing.

### Step 4. Ask the privacy question

Ask this every time:

> Does this need to be private? I can put a username and password on it. Default is open, meaning
> anyone with the link can read it.

If yes, follow the protection section below **after** the document is published.

### Step 5. Check whether the slug already exists

```
github_get_file
  owner: YOUR_GITHUB_USERNAME
  repo: founder-fork-html-publisher
  path: proposals/[slug].html
```

- **Not found:** new document, no `sha` needed
- **Found and this is an intentional update:** capture the `blob_sha`, pass it as `sha`
- **Found and this is a new document:** stop. The slug is taken. Propose a different one

### Step 6. Push

```
github_create_or_update_file
  owner: YOUR_GITHUB_USERNAME
  repo: founder-fork-html-publisher
  path: proposals/[slug].html
  message: "publish: [slug]"
  content: [full HTML string, starting with the meta comment]
  sha: [only when updating an existing file]
```

### Step 7. Return the link

> Published. Live in about 30 seconds:
> **https://YOUR_RAILWAY_DOMAIN/proposals/[slug].html**
>
> It will appear on your index page in a minute or two.

Always return the full clickable URL.

---

## Protecting a document

Works the same whether the document is new or was published months ago. The URL never changes.

### 1. Add the slug to `protected.json`

Read it first, the update needs its SHA:

```
github_get_file
  owner: YOUR_GITHUB_USERNAME
  repo: founder-fork-html-publisher
  path: protected.json
```

Add the slug to the `gated` array and push it back:

```json
{
  "gated": ["client-partnership-0726"]
}
```

This file records which documents are protected. It never contains passwords.

**It must stay valid JSON.** If it breaks, the server seals every document rather than risk
exposing a protected one, and everything returns "temporarily unavailable."

### 2. Generate credentials

- **Username:** the slug with dashes turned into underscores
- **Password:** a generated string, 16 characters or more, no ambiguous characters

### 3. Set the Railway variable

Name is the slug, uppercased, dashes turned into underscores, prefixed with `GATE_`.
Value is `username:password`.

```
GATE_CLIENT_PARTNERSHIP_0726 = client_partnership_0726:Xk4mPq9wRt2vLb8n
```

**With the Railway connector:** find the publisher project with `list-projects`, then `set-variables`.
Setting a variable triggers an automatic redeploy, live in about 60 seconds.

**Without it:** give the user the name and value and this instruction:

> Railway, your publisher service, Variables tab, New Variable, paste the name and the value.
> It redeploys itself in under a minute.

### 4. Hand over the credentials

This conversation is the only place they appear in readable form. Tell the user to send the
username and password to the recipient **separately from the link**, not in the same message.

### Removing protection

Take the slug out of the `gated` array and push. The page is open again at the same URL. The
Railway variable can stay, it is ignored once the slug is ungated.

---

## Updating a published document

Same slug, same URL, new content. Fetch the current `blob_sha` with `github_get_file`, then push
with that `sha`.

The index never recalculates a document's name or group after the first time it appears, so
changing `title` in the meta comment on an update will not change what the index displays. That
requires editing `index.html` directly.

---

## Troubleshooting

| What the user sees | What it means |
|---|---|
| Index is empty though documents were published | GitHub Actions is switched off on the fork. Enable it once in the Actions tab. Most common cause by far |
| A document just published is not on the index | Normal. The index rebuild is a separate Action, 1 to 2 minutes. Refresh |
| Protected page says "not currently available" instead of prompting for a password | The `GATE_` variable is missing or malformed. Check the name matches the slug exactly and the value has a colon between username and password |
| Everything returns "temporarily unavailable" | `protected.json` has a JSON syntax error. Fix it, it recovers on the next deploy |
| Page shows raw HTML instead of a formatted page | The markup was escaped on publish. Republish it |
| Index shows "Your Name" or the wrong branding | `site.config.json` has not been customized, or has a syntax error |
| Push fails with a permissions error | GitHub connector problem, not a publishing problem. Stop and fix the connector |

---

## Checklist

- [ ] Both placeholders replaced in your copy of this file
- [ ] HTML is complete and standalone, Google Fonts only
- [ ] Meta comment on line one with a valid `group` key
- [ ] User approved the document
- [ ] Slug confirmed, unique, correct format
- [ ] Checked for an existing file at that path, captured SHA if updating
- [ ] Privacy question asked
- [ ] Pushed to `proposals/[slug].html`
- [ ] Live URL returned in the conversation
- [ ] If protected: `protected.json` updated, `GATE_` variable set, credentials handed over separately from the link
