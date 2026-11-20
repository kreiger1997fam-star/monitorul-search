# Plan: Monitorul Oficial search website (pilot)

This is the plan, written in simple words. **I will not build anything until you approve it.**

---

## 1. What I already checked on the official website

I looked at the real site (politely, only a few pages) so the plan matches how it really works:

| What | What I found |
|---|---|
| `robots.txt` | Allows all bots. Nothing is forbidden. |
| Edition pages `monitorul.gov.md/ro/monitor/<id>` | The text lists every act in **Parts I, II and III** (number, type, title, act number and date), grouped by issuer. It also shows which parts exist for that edition. |
| Newest edition today | ID **3380** = Nr. 577-580 from 20.11.2026. ID 3381 does not exist yet. |
| January 2024 | ID 2830 = 29.12.2023 and ID 2840 = 11.01.2024. So the first 2024 edition is between 2831 and 2840. In total about **550 editions** to collect. |
| PDF viewer `.../view/pdf/<id>/part/<n>` | The viewer page loads the PDF of that part. The collector can read the PDF text the same way the viewer does. |
| **Part IV** (announcements) | About 30 pages per edition. There are **no headings**. Each announcement simply starts with the issuer's name in **bold** (company, town hall, bailiff, bank, etc.). |
| **Contents** ("Cuprins") | Repeats the Parts I–III list, plus the **page number** of each act. It does **not** list the Part IV announcements. |
| legis.md | Protected by a Cloudflare "are you a robot" check. So I will **not** collect anything from legis.md. The website will only **link** to it. |

---

## 2. What will be built

### A. The collector (a small Python program)

For each edition, from the newest one back to January 2024, it will:

1. Open the edition page and read the list of acts in Parts I, II and III.
2. Open the **Contents** through the viewer, only to get the page number of each act.
3. Open **Part IV** through the viewer and split it into separate announcements, using the bold issuer name at the start of each one.
4. Save everything into **`data/items.json`**.

For every act or announcement it saves:

| Field | Example |
|---|---|
| Edition number | Nr. 565-568 |
| Edition date | 2026-11-13 |
| Part | III |
| Issuer or company | Ministerul Finanțelor |
| Type | Ordin |
| Title | Ordin cu privire la modificarea Clasificației bugetare |
| Number / date of the act | nr. 148, 5 noiembrie 2026 |
| Page | 44 |
| Link | official viewer page of that part |

**Part IV titles.** Announcements in Part IV have no official title. So the "title" will be the **first sentence** of the announcement, shortened to about 200 characters. The **type** is guessed from keywords:
lichidare, reorganizare, insolvabilitate / faliment, licitație, concurs (job), adunare generală, executor judecătoresc, citație / hotărâre judecătorească, or "altele" (other).

**Errors.** If one part cannot be read (for example the PDF is not online yet), the edition is marked "incomplete" and is tried again the next day.

### B. Privacy and copyright rules (built into the code)

- **Part V is never opened.** The program refuses any Part V link, even by mistake.
- **No PDFs are stored or republished.** PDFs are read in memory and thrown away. The website links to the official viewer.
- **Part IV can also contain private people's names** (for example debtors in bailiff notices, sometimes with a birth date and address). To protect them:
  - Only the short first sentence is kept, never the full text.
  - Phone numbers, e-mails, IDNP numbers, IBANs, birth dates and street addresses are removed.
  - Names of **private persons** are replaced with "[persoană fizică]". Company, town-hall and bailiff names stay, because they are what people search for.

### C. Being polite to the website

- Checks `robots.txt` before starting.
- Waits **2 seconds** between every request.
- Identifies itself with a clear bot name, for example: `MonitorulSearchBot/1.0 (non-commercial pilot)`.
- If the site gives errors, it slows down and retries a few times, then stops.
- About 5 requests per edition, so the **first full run takes about 2 hours**. After that, each daily run takes about 1 minute.

### D. The website (folder `docs/`, in Romanian)

One simple page that works well on a phone:

- A **search box** with instant results as you type. It ignores accents, so "hotarare" also finds "Hotărâre".
- **Filters:** part, type, issuer, date from / to.
- Results are sorted **newest first**. You see the first 50, and more load as you scroll down.
- Each result has buttons:
  - **"Vezi în Monitorul Oficial"** opens the official viewer.
  - **"Caută pe legis.md"** opens legis.md for that act (Parts I–III only, because announcements are not on legis.md).
- Plain HTML and JavaScript. Nothing to install. Hosted for free with **GitHub Pages**.

The data for the site is split by year (for example `docs/data/2026.json`), so a phone first downloads only the newest data and loads older years only when needed.

### E. RSS feeds for your keywords

- You write your keywords in one simple file, **`feeds.json`**, for example `["lichidare", "TVA"]`.
- Every day one feed is made per keyword, for example `docs/rss/lichidare.xml`.
- Each feed has the 100 newest matching items. You can add it to any RSS reader app.

### F. The daily automatic update (GitHub Actions)

- Runs every day at **08:00 Moldova time**.
  - GitHub only understands UTC time, and Moldova changes between summer and winter time. So the job starts at both 05:00 and 06:00 UTC, and the program only works on the run that falls at 08:00 Moldova time.
  - Note: GitHub sometimes starts scheduled jobs 10–30 minutes late.
- Collects **only new editions** (and retries "incomplete" ones). Then it rebuilds the site data and RSS feeds and **commits** the changes.
- Has `permissions: contents: write` and a **"Run workflow"** button (`workflow_dispatch`) so you can start it by hand.

### G. README

A short guide in simple English: what the project does, how to turn on the website, how to change keywords, and how to run the update by hand.

---

## 3. Files that will be created

```
PLAN.md                     this plan
README.md                   simple guide
requirements.txt            the Python libraries the collector needs
feeds.json                  your RSS keywords
collector/collect.py        downloads and reads the editions
collector/build_site.py     makes the website data and the RSS feeds
data/items.json             all collected items (the main data)
data/state.json             which editions are done or incomplete
docs/index.html             the website
docs/app.js                 search and filters
docs/style.css              phone-friendly design
docs/data/<year>.json       search data for the website
docs/rss/<keyword>.xml      RSS feeds
.github/workflows/daily.yml the daily automatic update
```

---

## 4. The order of work

1. **You approve this plan** (or tell me what to change).
2. I write the collector and **test it on the last 10 editions only** (IDs 3371–3380).
3. **I show you the results:** how many items per part, some example rows (including Part IV and the privacy masking), and a screenshot of the website on a phone screen.
4. I run the **full collection** back to January 2024 (about 2 hours).
   If you want to check the test results yourself before this step, tell me when you approve. Otherwise I continue straight after showing you the results.
5. I build the website, RSS feeds, daily job and README. I test them, then commit and push to the branch `claude/compassionate-hamilton-8xtng5`.

---

## 5. Two things you will need to do on GitHub (I will explain step by step)

1. **Make the daily job run.** GitHub only runs scheduled jobs from the main branch. When everything is ready, the work must be merged into `main`. I can open a pull request for you if you ask.
2. **Turn on the website:** Settings → Pages → "Deploy from a branch" → `main` → `/docs`. Your site will then be at
   `https://kreiger1997fam-star.github.io/monitorul-search/`

Your repository is public, so GitHub Pages is free.

---

## 6. Questions for you

You can simply answer "OK to all defaults" if you agree.

1. **Bot name.** Should the bot name include a link to your GitHub project, so the website owners can see who is visiting? The link shows your GitHub username to them.
   *Default: no link, only `MonitorulSearchBot/1.0 (non-commercial pilot)`.*
2. **Private names in Part IV.** Replace private persons' names with "[persoană fizică]"?
   *Default: yes.*
3. **RSS keywords.** Which keywords do you want, besides "lichidare" and "TVA"? You can change them later in `feeds.json`.
   *Default: "lichidare", "TVA".*
4. **legis.md button.** I cannot test legis.md links automatically, because it blocks bots. The button will open a legis.md search for the act's type, number and date. After the site is built, please open one button on your phone and tell me if it works.
