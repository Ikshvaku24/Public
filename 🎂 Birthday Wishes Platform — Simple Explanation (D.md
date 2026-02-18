<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# 🎂 Birthday Wishes Platform — Simple Explanation (Databricks Version)


***

## The Big Picture in One Line

> **"Everything lives inside Databricks — the website, the database, the AI. Your team already has access. Nothing new to provision."**

***

## Real Story: Raj's Birthday (Feb 24)

### 📅 February 7 — 10 Working Days Before (9:00 AM)

**The Robot Wakes Up (Power Automate)**

```
Every morning at 9 AM, Power Automate asks:
"Hey Databricks app — any birthdays in 10 working days?"
    ↓
Databricks checks Delta Lake table: "Yes! Raj — Feb 24"
    ↓
Power Automate sends Teams messages automatically
```

**Raj gets this Teams message:**
> 🎂 *"Your birthday is coming! Upload your photo here → [link]"*

**Priya (team member) gets:**
> 🎂 *"Raj's birthday is Feb 24! Submit your wishes here → [link]"*

***

### 🖥️ Raj Opens the Link

```
Raj clicks link
    ↓
Opens Databricks App (website hosted inside Databricks)
    ↓
Databricks already knows it's Raj
(same login he uses for notebooks — no new password!)
    ↓
Databricks checks: "Is Raj a birthday person?"
    ↓ YES
Shows Raj ONLY: 📷 "Upload Your Photo"
    ↓
Raj drags his photo → auto-compressed → saved in Delta table
    ↓
Done! Raj cannot see any wishes. (Surprise protected! 🎁)
```


***

### 🖥️ Priya Opens the Link

```
Priya clicks link
    ↓
Opens same Databricks App
    ↓
Databricks knows it's Priya (same workspace login)
    ↓
Checks: "Is Priya a birthday person?" → NO
    ↓
Shows Priya: Dashboard with upcoming birthdays
    ↓
Priya clicks "Write Wish for Raj"
    ↓
Types: "Happy Birthday Raj! You're an amazing teammate! 🎉"
    ↓
Clicks Submit → Saved in Delta Lake table
    ↓
Priya can also view all wishes others submitted ✅
```


***

### 📅 February 11 — 8 Working Days Before (9:00 AM)

**Power Automate checks who forgot:**

```
"Who hasn't submitted a wish yet?"
    ↓
Databricks checks: Priya submitted ✅, Amit didn't ❌
    ↓
Teams reminder sent ONLY to Amit:
"⏰ Last chance! Submit wish for Raj before deadline!"
```


***

### 📅 February 12 — 7 Working Days Before (8:00 AM)

**The Magic Happens — Fully Automatic! ✨**

```
Step 1: Power Automate calls Databricks App
        "Compile everything for upcoming birthdays!"
    ↓
Step 2: Databricks fetches all 12 wishes from Delta table
    ↓
Step 3: Databricks fetches Raj's photo from Delta table
    ↓
Step 4: Databricks calls its OWN AI model (internal!)
        "Write a poem from these 12 wishes for Raj"
    ↓
Step 5: AI returns poem in 2 seconds:
        "A colleague bright with skills so true,
         Raj inspires in all we do..."
    ↓
Step 6: Returns everything to Power Automate
    ↓
Step 7: Power Automate sends email to manager
        From: your-email@company.com
        To: manager@company.com
        Content: Photo + All 12 wishes + AI poem
    ↓
Done! Raj still has no idea. 🎁
```


***

### 📅 February 24 — Birthday! 🎉

```
Manager opens the email received on Feb 12
    ↓
Presents to the team in a meeting
    ↓
Raj sees all the wishes for the FIRST TIME
    ↓
Everyone celebrates! 🎂
```


***

## 🧩 What Each Piece Does (Super Simple)

### 1. 🔴 Databricks App — The Website

**What it is**: A Python website that runs **inside** your Databricks workspace

**What users see**:


| Who | What They See |
| :-- | :-- |
| Birthday person (Raj) | Photo upload page ONLY |
| Team member (Priya) | Dashboard + wish form + view wishes |
| Admin / HR | All of the above + manage members + holidays |
| Manager | Nothing (just gets an email) |

**Why it's great**: Same login as your Databricks notebooks. No new password. No new app. Zero setup for users.

***

### 2. 🔵 Delta Lake Tables — The Filing Cabinet

**What it is**: Like an Excel sheet, but inside Databricks. Called "Delta Lake".

**What's stored**:

```
📁 birthdays table     → Raj Kumar, raj@co.com, Feb 24, Manager: priya@co.com
📁 wishes table        → "Happy Birthday Raj!" — by Priya, Feb 7
📁 photos table        → Raj's compressed photo (stored as text/base64)
📁 holidays table      → Jan 26 Republic Day, Mar 17 Holi...
📁 admins table        → hr@co.com, admin@co.com
📁 notification_log    → Reminder sent to Raj on Feb 7 ✅
```

**Why not Cosmos DB?** Because Delta Lake is **already in your workspace**. No new database to create or pay for.

***

### 3. 🟣 Power Automate — The Timer Robot

**What it is**: A scheduled robot that runs 3 jobs daily (already included in your Microsoft 365 license)

```
Job 1 — Every day at 9 AM:
    "Any birthdays in 10 working days?" → Send Teams reminders

Job 2 — Every day at 9 AM:
    "Who hasn't submitted yet (8 days left)?" → Send nudge

Job 3 — Every day at 8 AM:
    "Any birthdays in 7 working days?" → Compile + email manager
```

**Why Power Automate?** It has built-in Teams and Outlook connectors. No new permissions. Already paid for via M365.

***

### 4. 🔴 Databricks AI (LLM) — The Poem Writer

**What it is**: Your team's **existing** AI model already running in Databricks

**What it does**:

```
Input (wishes from team):
  - "Amazing teammate"
  - "Always helps others"
  - "Brilliant coder"

Output (poem, in 2 seconds):
  "A teammate bright with code so fine,
   Whose help and skill truly shine..."
```

**Why it's great**: It's an **internal call** — no external API, no new token, no extra cost. The AI is already there in your workspace.

***

### 5. 🟡 Workspace Login — The Security Guard

**What it is**: Databricks already knows who you are when you open the app

```
Raj opens the app
    ↓
Databricks sees: "Oh, this is raj@company.com"
    ↓
(He's already logged into his Databricks workspace)
    ↓
App knows exactly who he is — no login screen needed
```

**Why it's great**: No Azure Entra ID app registration. No new permissions to request. It just works.

***

## 🔒 How It Stays Secure (Simple)

```
1. Only company Databricks users can open the app
   (Public internet users cannot access it)

2. Birthday person is blocked from seeing wishes
   (System checks email — if it's their birthday, wishes are hidden)

3. Only admins can add/remove members
   (Email checked against admins table in Delta Lake)

4. All data encrypted automatically
   (Databricks does this by default — like a locked safe)

5. All actions are logged
   (notification_log table keeps a trail of everything)
```


***

## 📅 Holiday-Aware Working Days (Simple Explanation)

The system **never counts weekends or holidays** when calculating reminders.

**Example:**

```
Birthday: Monday, March 2

10 working days before = ?

Go back 10 working days (skip weekends + holidays):
  March 2  → Feb 28 (Sat) skip
           → Feb 27 (Fri) = day 1
           → Feb 26 (Thu) = day 2
           → Feb 24 (Tue) = skip Feb 25 (Holi holiday!)
           ... and so on

Result: Reminder sent on correct working day ✅
```

Holidays are stored in the `holidays` Delta table. Admin can add/remove them from the app itself.

***

## 💰 Cost (In Simple Terms)

```
Databricks App      → Uses your existing workspace = $0 extra
Delta Lake Tables   → Uses your existing storage   = $0 extra
Databricks AI       → Uses your existing LLM       = $0 extra
Power Automate      → Included in M365 license     = $0 extra
Teams Notifications → Included in M365 license     = $0 extra
Outlook Email       → Included in M365 license     = $0 extra

TOTAL EXTRA COST: $0/month 🎉
```


***

## 🆚 Why This Is Better Than Before

| Question | Old Plan (v2.0) | **New Plan (v3.0)** |
| :-- | :-- | :-- |
| Where is the website? | Azure Static Web Apps (new resource) | **Inside Databricks (existing)** |
| Where is the database? | Cosmos DB (new resource) | **Delta Lake (existing)** |
| How does login work? | Azure Entra ID (new app registration) | **Databricks workspace (automatic)** |
| What language? | JavaScript + Python | **Python only** |
| How does AI work? | External HTTP call | **Internal call (same workspace)** |
| New things to approve? | 2 approvals | **1 approval only** |
| New things to provision? | 2 Azure resources | **0 new resources** |


***

## ✅ All Requirements Met — Simple Check

| What Was Asked | How It's Done | ✅ |
| :-- | :-- | :-- |
| Birthday person uploads only their photo | App checks email → shows only photo page to them | ✅ |
| Birthday person can't see wishes | System blocks it at database level — returns empty | ✅ |
| Team submits wishes | Simple text form in the app | ✅ |
| Team views wishes | List shown to everyone except birthday person | ✅ |
| Reminder 10 working days before | Power Automate robot sends Teams message daily | ✅ |
| 3 working days window | API rejects wishes submitted too early or too late | ✅ |
| Auto-compile 7 days before | Power Automate triggers compilation automatically | ✅ |
| AI poem | Existing Databricks LLM writes the poem | ✅ |
| Email to manager | Power Automate Outlook connector sends it | ✅ |
| Admin adds/removes members | Admin page in the same app | ✅ |
| Holiday calendar | Admin page manages holidays in Delta table | ✅ |


***

## 🚀 How to Deploy (Super Simple)

```
Step 1: Ask Databricks admin:
        "Can I create a Databricks App and a Delta Lake schema?"
        → One email, one approval (15 min)

Step 2: Run setup notebook:
        Creates all 6 Delta tables
        → 5 minutes

Step 3: Deploy the app:
        databricks apps deploy birthday-app
        → 2 minutes

Step 4: Set up 3 Power Automate flows:
        → 15 minutes

Step 5: Done! Go-live 🎉
```

**Total setup time after approval: ~25 minutes**
**Total development time: 3 weeks**

***

## 🎯 Summary in 5 Points

1. **One platform** — The entire app (website + database + AI) lives inside Databricks that your team already uses
2. **Zero new login system** — Users log in with the same credentials they use for Databricks notebooks every day
3. **Zero new Azure resources** — No new Cosmos DB, no new Static Web Apps, nothing to provision
4. **Power Automate does the boring work** — Sends reminders, triggers compilation, emails the manager — all automatically
5. **Birthday person is always surprised** — The system enforces this at the database level, not just the UI

> **"Your team already has Databricks. The birthday app just lives inside it."** 🎂

