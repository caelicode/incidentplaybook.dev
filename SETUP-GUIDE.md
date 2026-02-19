# Incident Response Playbook - Complete Setup Guide

This document covers every step taken to build, launch, and deploy the Incident Response Playbook Bundle as a paid digital product. Use it as a reference to reproduce the setup, onboard someone new, or launch a similar product.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Product Creation (PDF + Templates)](#2-product-creation-pdf--templates)
3. [Landing Page Build](#3-landing-page-build)
4. [Payment Setup (Lemon Squeezy)](#4-payment-setup-lemon-squeezy)
5. [Email Capture Setup (Kit / ConvertKit)](#5-email-capture-setup-kit--convertkit)
6. [Deployment (Vercel + Custom Domain)](#6-deployment-vercel--custom-domain)
7. [Marketing Content](#7-marketing-content)
8. [Architecture Overview](#8-architecture-overview)
9. [How to Make Changes](#9-how-to-make-changes)
10. [Troubleshooting](#10-troubleshooting)
11. [Accounts and Credentials](#11-accounts-and-credentials)
12. [Cost Breakdown](#12-cost-breakdown)
13. [Design Decisions](#13-design-decisions)

---

## 1. Project Overview

**Product:** The Incident Response Playbook Bundle
**URL:** [incidentplaybook.dev](https://incidentplaybook.dev)
**Pricing:** $49 (Individual License) / $79 (Team License)
**Format:** 56-page PDF + 7 downloadable YAML/Markdown templates
**Target audience:** DevOps engineers, SREs, platform engineers, engineering managers at small-to-mid teams (5-100 engineers)

**What the customer gets:**

- A 56-page PDF covering severity classification, escalation frameworks, 10 debugging playbooks, postmortem framework, on-call operations, chaos engineering, and common mistakes
- 7 template files (YAML and Markdown) they can drop into their workflow
- The Team License additionally includes editable Google Docs and Notion versions

**Lead magnet:** A free 3-page Blameless Postmortem Template PDF, gated behind email signup

**Strategic context:** This is Phase 1, Product #1 of a Digital Products to SaaS roadmap. Goal is $500-$2,000/month from digital products before building SaaS.

---

## 2. Product Creation (PDF + Templates)

### 2.1 Source Content

All chapter content lives in `chapters/` as Markdown files:

```
chapters/
  00-introduction.md        - Who this is for, how to use it
  00b-quickstart.md         - The entire framework on one page
  01-severity-classification.md  - P1-P4 definitions, war stories
  02-escalation-frameworks.md    - Escalation paths, IC role, templates
  03-debugging-playbooks.md      - 10 step-by-step debugging procedures
  04-postmortem-framework.md     - 5 Whys, postmortem library
  05-oncall-operations.md        - Rotations, handoffs, runbooks
  06-chaos-engineering.md        - Experiments, tools, maturity model
  07-common-mistakes.md          - 10 anti-patterns with fixes
```

### 2.2 Building the PDF

The PDF is generated using Python + ReportLab with a custom Markdown-to-PDF parser:

```bash
cd incident-response-playbook
pip install reportlab --break-system-packages
python build_guide.py
# Output: output/Incident-Response-Playbook-Bundle.pdf (56 pages, ~141 KB)
```

The parser in `build_guide.py` handles: headings (H1-H4), code blocks, tables, bullet/numbered lists, blockquotes, inline formatting (bold, italic, code spans), and page numbers in the Table of Contents.

### 2.3 Building the Lead Magnet

```bash
cd lead-magnet
python build_postmortem_template.py
# Output: lead-magnet/Blameless-Postmortem-Template.pdf (3 pages)
```

### 2.4 Templates

Seven template files ship alongside the PDF:

```
templates/
  incident-declaration.yaml    - Structured incident declaration
  severity-reference.md        - One-page P1-P4 quick reference
  postmortem-template.md       - Full postmortem document
  runbook-template.md          - Blank runbook structure
  oncall-handoff.md            - End-of-rotation handoff checklist
  stakeholder-comms.md         - 6 communication templates
  chaos-experiment.yaml        - Experiment planning structure
```

### 2.5 Creating the ZIP Bundle

The downloadable product is a ZIP containing the PDF + all 7 templates:

```bash
cd incident-response-playbook
zip -r Incident-Response-Playbook-Bundle.zip \
  output/Incident-Response-Playbook-Bundle.pdf \
  templates/
# Output: Incident-Response-Playbook-Bundle.zip (~113 KB)
```

This ZIP is what gets uploaded to Lemon Squeezy for delivery to customers.

---

## 3. Landing Page Build

### 3.1 Architecture

The landing page is a single `index.html` file with all CSS inline. No JavaScript framework, no build step. This keeps it fast, simple, and deployable anywhere.

**File:** `landing-page/index.html`

### 3.2 Page Sections

The page has these sections in order:

1. **Hero** - Headline, subtitle, CTA button ($49), secondary CTA (free template), price badge
2. **Pain Points** - 6 cards with Flaticon icons describing common problems
3. **What's Inside** - Chapter-by-chapter breakdown with numbered badges
4. **10 Debugging Playbooks** - Grid listing all 10 playbook titles
5. **7 Templates** - Cards showing each template filename and description
6. **Social Proof / Stats** - 56 pages, 10 playbooks, 7 templates, 4 communication templates
7. **Pricing** - Two cards: Individual ($49) and Team ($79, featured with green border)
8. **Lead Magnet** - Email capture form for free Blameless Postmortem Template
9. **FAQ** - 6 common questions
10. **Footer** - Domain, email, coming soon teaser, Flaticon attribution

### 3.3 Fonts

Uses Google Fonts with two typefaces:

- **Inter** (weights 400, 500, 600) - Body text, paragraphs, descriptions
- **Manrope** (weights 600, 700, 800) - Headings, buttons, stats, chapter numbers, prices

The font import and preconnect links are in the `<head>`:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=Manrope:wght@600;700;800&display=swap" rel="stylesheet">
```

### 3.4 Icons

The pain point cards use PNG icons from Flaticon (not emoji - emojis look AI-generated). All icons are 512px resolution, stored in `landing-page/icons/`:

| File | Flaticon ID | Concept |
|------|-------------|---------|
| `warning.png` | 550096 | Severity confusion |
| `stopwatch.png` | 3867499 | Debugging from scratch |
| `megaphone.png` | 2011858 | Escalation judgment calls |
| `recurring.png` | 7838482 | Same incidents recurring |
| `fatigue.png` | 3736108 | On-call burnout |
| `chat.png` | 1246332 | Stakeholder communication |

**Flaticon attribution** is required for free icons and is included in the footer:

```html
<p>Icons by <a href="https://www.flaticon.com">Flaticon</a></p>
```

### 3.5 Color Scheme

Defined as CSS custom properties (`:root`):

| Variable | Hex | Usage |
|----------|-----|-------|
| `--navy` | #1B4F72 | Hero background, headings, stat numbers, prices |
| `--dark` | #2C3E50 | Body text, footer background |
| `--blue` | #2980B9 | Links, chapter numbers, playbook numbers |
| `--green` | #27AE60 | CTA buttons, checkmarks, featured card border |
| `--amber` | #F39C12 | Hero heading accent ("Start Following the Playbook") |
| `--gray-50` | #F8F9FA | Section backgrounds (pain, proof) |
| `--gray-100` | #EAECEE | Borders, dividers |
| `--gray-600` | #566573 | Secondary text, descriptions |

### 3.6 Responsive Design

Mobile breakpoint at 600px:

- Hero heading shrinks from 2.6rem to 1.9rem
- Hero padding reduces
- Section padding reduces
- Section titles shrink
- Price cards go full width

---

## 4. Payment Setup (Lemon Squeezy)

### 4.1 Account Setup

1. Go to [lemonsqueezy.com](https://lemonsqueezy.com) and create an account
2. Store name: **Incident Playbook**
3. Store URL: `incidentplaybook.lemonsqueezy.com`

### 4.2 Product Configuration

Created a single product with **two variants**:

**Product name:** The Incident Response Playbook Bundle

**Individual License variant ($49):**

- Price: $49, single payment
- File: Upload `Incident-Response-Playbook-Bundle.zip`
- License keys: Enabled
- License key activation limit: 1
- License key length: Unlimited (no expiry)

**Team License variant ($79):**

- Price: $79, single payment
- Same file upload
- License keys: Enabled
- License key activation limit: Unlimited
- License key length: Unlimited

### 4.3 Checkout URLs

After publishing the product, Lemon Squeezy generates checkout URLs for each variant:

- **Individual:** `https://incidentplaybook.lemonsqueezy.com/checkout/buy/3156a1e5-8dd9-482e-9ca8-38ac7912ab37`
- **Team:** `https://incidentplaybook.lemonsqueezy.com/checkout/buy/56185f5a-03ab-404d-a5f1-e63fb34e211c`

### 4.4 Wiring Buy Buttons

Three buttons in `index.html` link to checkout:

1. **Hero CTA** ("Get the Playbook - $49") - Links to Individual checkout
2. **Individual pricing card** ("Buy Individual License") - Links to Individual checkout
3. **Team pricing card** ("Buy Team License") - Links to Team checkout

### 4.5 Store Activation (Stripe)

Lemon Squeezy uses Stripe for payment processing. To accept real payments:

1. Go to Lemon Squeezy Settings > Store
2. Click Activate / Go Live
3. Complete Stripe identity verification:
   - Website: `incidentplaybook.dev`
   - Product description: Describe what you sell (digital playbook bundle for DevOps teams, one-time purchase)
   - Personal/business details and bank account for payouts
4. Wait for Stripe verification (usually minutes to hours)

**Fees:** Lemon Squeezy takes 5% + $0.50 per transaction.

| Sale | Revenue | Fee | Net |
|------|---------|-----|-----|
| $49 Individual | $49 | $2.95 | $46.05 |
| $79 Team | $79 | $4.45 | $74.55 |

### 4.6 Payment Methods

In Lemon Squeezy payment settings, these are enabled:

- Credit/debit cards
- Apple Pay
- Google Pay
- PayPal

---

## 5. Email Capture Setup (Kit / ConvertKit)

### 5.1 Account Setup

1. Go to [kit.com](https://kit.com) (formerly ConvertKit)
2. **Important:** Select the **free plan**, not the 14-day Pro trial. The free plan is sufficient for email capture and includes up to 10,000 subscribers
3. Create account with personal info
4. Brand name: **Incident Playbook**
5. Categories: Educator + Maker
6. Topics: Technology, Education, Business

### 5.2 Creating the Email Capture Form

1. Go to **Grow > Landing Pages & Forms**
2. Click **Create new > Form > Inline**
3. Select the **Clare** template (clean, minimal)
4. Customize the form fields as needed
5. Click **Save and Publish**

### 5.3 Setting Up the Incentive Email (Lead Magnet Delivery)

The incentive email automatically sends the free Blameless Postmortem Template PDF to subscribers:

1. In the form settings, go to the **Incentive** tab
2. Enable **Send incentive email**
3. Subject line: Something like "Your Blameless Postmortem Template"
4. Body: Thank them, describe the template, include download link
5. Upload `lead-magnet/Blameless-Postmortem-Template.pdf` as the downloadable file
6. Save

### 5.4 Getting the Embed Code

1. After saving the form, go to the **Embed** tab
2. Select **HTML** embed (not JavaScript)
3. Copy the form action URL and attributes

Key values from the embed code:

- Form action URL: `https://app.kit.com/forms/9105651/subscriptions`
- Form ID attribute: `data-sv-form="9105651"`
- UID attribute: `data-uid="76813a51b2"`
- Email field name: `name="email_address"`

### 5.5 Wiring the Form in the Landing Page

The email form in `index.html` (in the Lead Magnet section) uses these attributes:

```html
<form class="email-form"
      action="https://app.kit.com/forms/9105651/subscriptions"
      method="POST"
      data-sv-form="9105651"
      data-uid="76813a51b2">
    <input type="email" name="email_address" placeholder="your@email.com" required>
    <button type="submit" class="btn btn-primary">Download Free Template</button>
</form>
```

**How it works:** When someone submits their email, Kit processes the subscription, adds them to your subscriber list, and sends the incentive email with the PDF download link.

---

## 6. Deployment (Vercel + Custom Domain)

### 6.1 GitHub Repository

The landing page code lives in its own Git repo. The repo was pushed from the `landing-page/` directory (not the parent `incident-response-playbook/` directory).

- **GitHub org:** caelicode
- **Repo:** `caelicode/incidentplaybook.dev`
- **Repo URL:** `https://github.com/caelicode/incidentplaybook.dev`
- **Visibility:** Public (required for Vercel free tier with org repos)

**Repo contents (at repo root):**

```
index.html          - The landing page
icons/              - 6 Flaticon PNG icons
  warning.png
  stopwatch.png
  megaphone.png
  recurring.png
  fatigue.png
  chat.png
.gitignore
.vercel/
```

### 6.2 Pushing to GitHub

```bash
cd ~/Documents/Side-Project/incident-response-playbook/landing-page

# Initialize and commit (already done)
git init
git add index.html icons/
git commit -m "Initial landing page with icons"

# Add remote and push
git remote add origin https://github.com/caelicode/incidentplaybook.dev.git
git push -u origin main
```

**Note on authentication:** GitHub requires a Personal Access Token (PAT) for HTTPS push, not a password. Generate one at: GitHub > Settings > Developer settings > Personal access tokens > Tokens (classic). Select the `repo` scope.

**Common issue - SSH vs HTTPS:** If you get `Permission denied (publickey)`, the remote is using SSH (`git@github.com:...`). Switch to HTTPS:

```bash
git remote set-url origin https://github.com/caelicode/incidentplaybook.dev.git
```

**Common issue - index.lock:** If you get `Unable to create '.git/index.lock': File exists`, run:

```bash
rm -f .git/index.lock
```

### 6.3 Deploying to Vercel

We used the Vercel CLI (the dashboard had issues with org repos on the free tier):

```bash
# Install Vercel CLI
npm install -g vercel

# Deploy from the landing page directory
cd ~/Documents/Side-Project/incident-response-playbook/landing-page
vercel
```

During the interactive setup:

- **Set up and deploy?** Yes
- **Which scope?** Your personal Vercel account
- **Link to existing project?** No
- **Project name?** `incidentplaybook`
- **Which directory is your code in?** `./`
- **Want to modify settings?** No

It will prompt you to log in via `vercel.com/device` on first use.

**Result:** Deployed to `incidentplaybook.vercel.app`

**Auto-deploy:** The CLI connected the GitHub repo, so future pushes to `main` auto-deploy.

### 6.4 Custom Domain Setup

1. Buy the domain (`incidentplaybook.dev`) from your registrar of choice

2. Add the domain to Vercel:

```bash
vercel domains add incidentplaybook.dev
```

3. Add DNS records at your domain registrar:

| Type | Host/Name | Value | TTL |
|------|-----------|-------|-----|
| A | `@` (or blank) | `76.76.21.21` | Automatic |
| CNAME | `www` | `cname.vercel-dns.com` | Automatic |

4. Wait for DNS propagation (usually instant to a few minutes)

5. Vercel automatically provisions an SSL certificate

**Result:** `https://incidentplaybook.dev` serves the landing page with HTTPS.

---

## 7. Marketing Content

Pre-written marketing content is in the `marketing/` directory:

### 7.1 LinkedIn / Dev.to Posts

**File:** `marketing/linkedin-devto-posts.md`

Five teaser posts, each targeting a different angle:

1. Severity classification confusion
2. Debugging at 3 AM without runbooks
3. Communication anti-patterns during incidents
4. Postmortem failures
5. On-call burnout

### 7.2 Product Hunt Launch Copy

**File:** `marketing/product-hunt-copy.md`

Contains:

- Tagline (under 60 characters)
- Full product description
- Maker's first comment

### 7.3 Reddit Posts

**File:** `marketing/reddit-posts.md`

Three value-first posts for:

- r/devops
- r/sysadmin
- r/SRElife

Each post leads with actionable value before mentioning the product.

---

## 8. Architecture Overview

```
User visits incidentplaybook.dev
         |
         v
    Vercel (CDN)
    Serves index.html + icons/
         |
         |--- Buy button clicked ---> Lemon Squeezy checkout
         |                                   |
         |                                   v
         |                            Stripe processes payment
         |                                   |
         |                                   v
         |                            Customer gets ZIP download
         |                            (PDF + 7 templates)
         |
         |--- Email form submitted --> Kit (ConvertKit)
                                           |
                                           v
                                    Subscriber added to list
                                           |
                                           v
                                    Incentive email sent with
                                    Blameless Postmortem PDF
```

**Key services and what they do:**

| Service | Purpose | Cost |
|---------|---------|------|
| Vercel | Hosts the static landing page | Free (Hobby plan) |
| GitHub | Source control for landing page | Free |
| Lemon Squeezy | Payment processing + digital delivery | 5% + $0.50 per sale |
| Kit (ConvertKit) | Email capture + lead magnet delivery | Free (up to 10K subscribers) |
| Domain registrar | `incidentplaybook.dev` domain | ~$12/year |

---

## 9. How to Make Changes

### 9.1 Updating the Landing Page

1. Edit `landing-page/index.html` locally
2. Commit and push:

```bash
cd ~/Documents/Side-Project/incident-response-playbook/landing-page
git add index.html
git commit -m "Description of change"
git push
```

3. Vercel auto-deploys within seconds

### 9.2 Updating the PDF Content

1. Edit the relevant file(s) in `chapters/`
2. Rebuild the PDF:

```bash
python build_guide.py
```

3. Rebuild the ZIP:

```bash
zip -r Incident-Response-Playbook-Bundle.zip output/Incident-Response-Playbook-Bundle.pdf templates/
```

4. Upload the new ZIP to Lemon Squeezy (Product > Files > replace the ZIP)

### 9.3 Updating the Lead Magnet

1. Edit and rebuild:

```bash
cd lead-magnet
python build_postmortem_template.py
```

2. Upload the new PDF to Kit (Form > Incentive > replace the file)

### 9.4 Changing Prices

1. Go to Lemon Squeezy > Products > Edit the relevant variant
2. Update the price
3. The checkout URLs stay the same (no landing page change needed)
4. Optionally update the prices displayed on the landing page in `index.html`

### 9.5 Changing the Domain

1. In Vercel: Settings > Domains > Add new domain
2. Update DNS records at your registrar
3. Remove old domain from Vercel

---

## 10. Troubleshooting

### Git push fails with "Permission denied (publickey)"

Your remote is using SSH. Switch to HTTPS:

```bash
git remote set-url origin https://github.com/caelicode/incidentplaybook.dev.git
```

Use a GitHub PAT (Personal Access Token) as the password.

### Git push fails with "index.lock: File exists"

Another git process left a lock file. Remove it:

```bash
rm -f .git/index.lock
```

### Vercel says org repos require Pro plan

Make the GitHub repo public. This is a landing page with no sensitive code, so public is fine.

### DNS not resolving after adding records

DNS can take up to 48 hours (usually much faster). Verify your records:

```bash
dig incidentplaybook.dev A
# Should return 76.76.21.21
```

### Checkout buttons not working

Verify the Lemon Squeezy store is activated (not in test mode). Check Lemon Squeezy > Settings > Store for activation status.

### Email form submissions not arriving

Check Kit > Subscribers to see if signups are coming through. If the form submits but subscribers don't appear, verify the `action` URL and `data-sv-form` / `data-uid` attributes match what Kit provided.

---

## 11. Accounts and Credentials

| Service | URL | Account |
|---------|-----|---------|
| Lemon Squeezy | lemonsqueezy.com | bertrandmbanwi@gmail.com |
| Kit (ConvertKit) | kit.com | bertrandmbanwi@gmail.com |
| Vercel | vercel.com | Linked to GitHub (bertrandmbanwi) |
| GitHub (org) | github.com/caelicode | caelicode org |
| GitHub (personal) | github.com/bertrandmbanwi | bertrandmbanwi |
| Domain registrar | (wherever purchased) | incidentplaybook.dev |

---

## 12. Cost Breakdown

### Fixed Costs

| Item | Cost | Frequency |
|------|------|-----------|
| Domain (incidentplaybook.dev) | ~$12 | Annual |
| Vercel hosting | $0 | Free tier |
| Kit email | $0 | Free tier (up to 10K subscribers) |
| GitHub | $0 | Free tier |
| **Total fixed cost** | **~$12/year** | |

### Per-Sale Costs (Lemon Squeezy)

| Sale Type | Revenue | Lemon Squeezy Fee (5% + $0.50) | Net |
|-----------|---------|--------------------------------|-----|
| Individual ($49) | $49 | $2.95 | $46.05 |
| Team ($79) | $79 | $4.45 | $74.55 |

### Break-Even

You need 1 individual sale per year to cover the domain cost. Everything after that is profit.

---

## 13. Design Decisions

These are intentional choices made during the build, documented so future changes don't accidentally undo them:

1. **No em dashes or en dashes anywhere in the project.** Em dashes are an AI writing fingerprint. All content uses " - " (space-hyphen-space) instead. This applies to the PDF chapters, templates, landing page copy, and marketing posts.

2. **No emoji icons on the landing page.** Emoji in web design signals AI-generated content. We use Flaticon PNG icons (outline style, 512px) instead. Flaticon attribution is required in the footer for free icons.

3. **Single-file HTML landing page.** No CSS/JS files, no framework, no build step. This makes deployment trivial (any static host works) and page load fast.

4. **Google Fonts (Inter + Manrope) instead of system fonts.** System fonts looked "dull" per feedback. Inter is a clean body font, Manrope adds personality to headings with its geometric shapes and tight letter-spacing.

5. **Vercel CLI deployment instead of dashboard.** The Vercel dashboard had issues importing org repos on the free tier. The CLI worked immediately and also connected the GitHub repo for auto-deploy.

6. **HTTPS remote instead of SSH for GitHub.** SSH required key setup that was blocking deployment. HTTPS with a PAT is simpler for a single-repo workflow.

7. **Checkbox rendering in lead magnet.** Unicode box characters and ZapfDingbats both rendered as filled squares in common PDF readers. We use Courier `[ ]` brackets instead.

8. **TOC page numbers are hardcoded.** After measuring actual page positions from a build, the TOC tuples are set manually in `build_guide.py`. If chapter content changes significantly, rebuild once, check page positions, and update the TOC.

---

## Build Log

| Date | Session | What Was Done |
|------|---------|---------------|
| 2026-02-19 | Session 1 | Created strategic roadmap doc. Researched market. Set up project structure. Wrote all 6 chapters. Created 7 templates. Built lead magnet PDF. Built PDF assembler. Generated 41-page PDF. |
| 2026-02-19 | Session 2 | Removed all em dashes (105 instances) across 12 files. Rebuilt all PDFs. |
| 2026-02-19 | Session 3 | Added Introduction + Quick Start chapters. Expanded Chapters 4-6 (postmortem, on-call, chaos engineering). Fixed checkbox rendering. Updated TOC. PDF: 41 to 51 pages. |
| 2026-02-19 | Session 4 | Added war stories to Ch 1. Added communication anti-patterns to Ch 2. Added Chapter 7 (10 common mistakes). Added H4 support + TOC page numbers. PDF: 51 to 56 pages. |
| 2026-02-19 | Session 5 | Packaged ZIP bundle. Built landing page. Wrote marketing posts (LinkedIn, Product Hunt, Reddit). |
| 2026-02-19 | Session 6 | Replaced emoji with Flaticon icons. Changed fonts to Inter + Manrope. Visual polish pass. Set up Lemon Squeezy (product, two variants, checkout URLs). Set up Kit (form, incentive email). Wired buy buttons + email form. Deployed to Vercel. Connected incidentplaybook.dev domain. Completed Stripe verification. |
