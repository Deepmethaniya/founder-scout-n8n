# Founder Scout — find the founders of newly funded startups, automatically

An n8n workflow that finds German startups that just raised money, identifies their founders from public sources, and saves each founder's **verified** LinkedIn profile to Notion — with the source and the exact evidence behind every name.

[![Watch the 44-second demo](https://img.youtube.com/vi/X-VGw0CgbSk/maxresdefault.jpg)](https://youtu.be/X-VGw0CgbSk)

▶ **[Watch the 44-second demo on YouTube](https://youtu.be/X-VGw0CgbSk)** — the workflow, a live run, and the results in Notion (founder names blurred for privacy).
▶ **[Full report with method, results and cost](https://claude.ai/artifact/RZfwdHFW8JVUAzQ7GbwrSz#01bca24e-c390)** — 

---

## Results

| | |
|---|---|
| Companies in a real run (13 Nov 2026) | **16** |
| Completed fully automatically | **10 of 16** (62.5%) |
| Founders found | **19**, each with a source and quoted evidence |
| Verified LinkedIn profiles | **12 of 19** |
| Guessed links / broken rows | **0** |
| Manual research time, measured | **9.25 min** per company |
| Projected for 100 companies | **~15.4 h → ~5.3 h** of human time (~10 h saved) |
| Running cost | **€0** — free up to ~290 companies a month |

Company-level results (no personal data): [`results/run_2026-11-13_summary.csv`](results/run_2026-11-13_summary.csv)

---

## How it works

![Founder Scout workflow](docs/workflow-diagram.png)

1. **Get funding rounds** — tech.eu API, German rounds from the last 90 days
2. **Keep new, early-stage companies** — skips companies already in Notion; keeps Pre-seed to Series A (and "Other" rounds up to €15M)
3. **Find the official website** — accepted only if the domain contains the company name
4. **Find founder names** — the company's **Impressum** (German law requires it to name the managing directors) plus funding news; AI only extracts the names written there
5. **Find the LinkedIn profile** — searches `linkedin.com/in` for the name + company
6. **Verify with fixed rules** — name, company and a founder/CEO role must all match; then save to Notion

Companies with no website or no named founder are **not dropped or guessed** — they are saved as *research by hand*, pre-filled with the tech.eu link, city and investors.

---

## Design decisions

- **AI only reads, never decides.** The AI's one job is to pull names out of text it is given. Website choice, stage filtering and LinkedIn verification are fixed rules.
- **Never guess a link.** A website must contain the company name; a LinkedIn link is taken only from a real search result, and only a profile (`/in/`) link.
- **"Verified" has a strict meaning:** name + company + founder/CEO role match, and no second profile fits equally well. Otherwise it is saved as *Unverified*, with the reason and every candidate.
- **Traceable rows.** Every founder carries the page the name came from and the quoted words that show the role.

**Why it was rebuilt:** an audit of the first version (one AI agent doing everything) found only **14 of 30 rows correct** — guessed links, broken rows, one wrong person, and a Series C company passing as early stage. Version 2 moved every decision into fixed rules.

---

## Time saved and cost (projection for 100 companies)

| Work | By hand | With Founder Scout |
|---|---|---|
| ~62 companies completed automatically | 574 min | 0 min of a person's time |
| ~38 companies flagged for manual research | 351 min | 228 min (pre-filled, ~6 min each) |
| ~44 unverified founders to check | — | 88 min (~2 min each) |
| **Total, person's time** | **925 min (15.4 h)** | **316 min (5.3 h)** |

**Tavily** (web search) costs 1 credit per basic search; the workflow averages ~3.4 per company → ~350 credits per 100 companies.

| Tavily plan | Credits / month | Price | ≈ Companies / month |
|---|---|---|---|
| Free | 1,000 | $0 | ~290 |
| Pay-as-you-go | per use | $0.008 / credit | ~$2.75 per 100 companies |
| Project | 4,000 | $30 / month | ~1,170 |

The AI model runs on OpenRouter's free tier. Pricing as of Nov 2026 — [Tavily docs](https://docs.tavily.com/guides/api-credits).

---

## Setup

1. **Import** `workflow.json` into n8n (⋯ → *Import from file*).
2. **Add credentials:**
   - **Tavily** → nodes `FindWebsite`, `SearchImpressum`, `SearchNews`, `SearchLinkedIn`
   - **OpenRouter** → node `AI model (OpenRouter)` (default: a free NVIDIA Nemotron model)
   - **Notion** → nodes `GetKnownCompanies`, `SaveFounder`, `SaveNoWebsite`, `SaveNoFounder`
3. **Create a Notion database** with these columns, then replace `YOUR_NOTION_DATABASE_ID` in the four Notion nodes:

   | Column | Type |
   |---|---|
   | Company | Title |
   | Website, Amount EUR, Person, Role, LinkedIn, Verified, Source URLs, Notes, Data status | Text |
   | Stage, Round status | Select |
   | Round date, Date found | Date |

4. **Test** with `Limit` = 3 → *Execute workflow*. Then raise `Limit` and switch on the **Run daily 07:00** trigger (change the time in that node).

All settings — days back, country, stages, amount cap, batch size, verification rules — are explained in the **sticky notes** on the canvas.

---

## Known limits

- **37.5% of companies had no website found** — they go to manual research. Next step: a second search with "GmbH".
- **Common names** (several profiles share a name) stay *Unverified* by design — a human picks the right one.
- **tech.eu returns at most 100 rounds per request** — turn on pagination for longer periods.
- **Free tiers** give occasional slow or empty AI replies (retries are on) and 1,000 searches a month.

---

## Privacy

This repository contains the method and aggregate numbers only — **no founder names or LinkedIn links**. The workflow uses public sources (company Impressum pages, funding news, public search results) and never logs in to or scrapes LinkedIn.

---

## Built with

n8n (self-hosted) · tech.eu funding API · Tavily · OpenRouter · Notion

*Built by Deep Methaniya — [deepmethaniya.eu](https://www.deepmethaniya.eu) · [LinkedIn](https://www.linkedin.com/in/deep-methaniya/)*
