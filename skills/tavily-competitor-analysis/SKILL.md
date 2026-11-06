---
name: tavily-competitor-analysis
description: |
  Compare a set of competitors on pricing, plans, usage limits, features, positioning and recent launches, and watch their pricing and release-notes pages for changes, using Tavily. Reads each competitor's own pricing page, rate card or fee schedule for the user's country and customer size, builds a comparison matrix, and saves those pages as a rerunnable monitor so the next run reports old and new prices. Works for software, APIs, usage-priced platforms and consumer or financial products. Use when the user asks for competitive intel, a competitor analysis, a competitor comparison or matrix, competitor pricing or fees, what a competitor shipped, how their product stacks up, or to track competitors over time. To find who the competitors are use tavily-list first. For market size use tavily-market-research. For one company in depth use tavily-due-diligence. For a single page or query on a cadence use tavily-monitor.
license: MIT
metadata:
  author: tavily
  version: "0.6.0"
  homepage: https://tavily.com
  source: https://github.com/tavily-ai/official-workflows
allowed-tools: Bash(tvly *)
---

# Tavily competitor analysis

Produce a comparison of competitors from their own pages, and leave behind a
monitor that turns the next run into a list of changes with old and new values.

Search and extract as `tavily-search` describes, and save the watched pages as
extract monitors as `tavily-monitor` describes, including its stale-copy check
and its rules for watched lines. If either isn't installed, run
`npx skills add tavily-ai/skills --skill <name>` and read its SKILL.md before
continuing. Read `references/reading-pages.md` before the first extract. It
covers prices the page extract leaves out, choosing between two reads of a
page, and products without a changelog.

## Intake

Infer from the request:

- the competitors, and the user's own product if they name it;
- the market, meaning the country or countries and the currency. Default to
  the user's own country when the request shows it, otherwise the US and USD,
  and say which;
- the customer and its size in the unit vendors price by, such as contacts,
  seats, messages a month or a business type. Price a customer measured in two
  units (employees and contractors abroad) in both. When the price needs a
  split the user didn't give (input against output tokens), state the split
  you assumed; it is an assumption about the user, never about a price. When
  the user gives no size, give the price per unit (per seat, per contact) and
  say so;
- the features, with the meaning the user intends when a term has two (SSO
  into the vendor, or the vendor acting as the identity provider), and a
  90-day window for recent changes;
- the length. When the user wants something short (a Slack thread, a slide),
  reply with the summary and matrix and still save the full report.

Ask at most 2 questions, only when the answer changes the matrix: which
features to compare when the user named none, and whether their own product
gets a column. Look for an existing `intel/` report and `monitors/` folders
for these competitors first; if they exist, this is a rerun.

When the user doesn't name competitors, build the list with `tavily-list`,
capped at 5, state how they were ranked (usage share from a named survey, for
example), and keep each competitor's domain.

## Sources

Each competitor's own site is the source of record for prices, plans and
dates, including its docs, help centre, fee schedules and price PDFs.
Third-party pages are leads, even when they quote a price.

**Resolve each product.** A product inside a larger company often keeps its
prices, release notes and announcements on different domains. Look for a
rename or acquisition in what you already read: "is now", "formerly",
"acquired by", or a home page that redirects to another brand. Confirm a hint
with a news search. Check the product is sold to the user's customer type.

**Find pages** with one call per page type and one query per competitor;
`tvly` 0.2.1 and later run up to 5 queries per call. The domain list is shared
across the queries, and each still returns its own product's pages:
`tvly search '{product A} pricing' '{product B} pricing' --include-domains
{all competitors' domains} --depth advanced --max-results 10 --json`.

- Check the locale in each result URL (`/en-AU/`, `/ie/`, `/us/`) and keep or
  rewrite it for the user's market. Where a URL has a language segment and a
  country segment (`twilio.com/en-us/sms/pricing/gb`), change the country.
  Keep the canonical URL in the ladder too, and check the currency of every
  read: a URL without a locale can be served for another country.
- Skip staging and preview hosts (`*-wip.`, `staging.`, `preview.`).
- Pages comparing the competitor with rivals (`/vs/`, `/compare/`,
  `/alternatives/`, "X vs Y" posts) are never a source for a rival's facts.
  When the vendor's compare or blog pages state a different price for itself
  than its pricing page, list the conflict under Gaps with both URLs.
- Pages for a sibling product on the same domain or support site are not
  sources. A product sold only inside a bundle or a broader plan is priced by
  the plans that include it; name the bundle and what else it contains.
- For recent changes of a software product, search all the names vendors use
  in one query: `tvly search "{product} release notes OR changelog OR what's
  new OR product updates" --include-domains {domains} --depth advanced
  --max-results 10 --json` (double quotes, because of the apostrophe). When that returns no product feed, search `"{product} what's new"`
  and `"{product} changelog"` separately. Reject changelogs of a different
  product (SDKs, client libraries, API references, community betas) unless
  the API is the product.
  For banks, consumer apps and other products that rarely have a changelog,
  skip the search. Use the reference's substitutes for them and for any
  product whose search found no changelog.
- When search and the substitutes miss a page, map the product's own path,
  not the domain root: `tvly map https://{domain}/{product path}
  --instructions 'release notes' --limit 30 --json`, one page type per call.
- Watch a changelog's index or feed, never a page for one month
  (`/release-notes/2026/october`).

**Outside coverage.** Run `tvly search '{company A} announces' '{company B}
announces' --topic news --start-date {window start} --max-results 10 --json`,
adding the product name only when it differs from the company's. Event words
keep out the reviews and tutorials that otherwise fill news results. Search
`'{product} pricing change'` when a page or lead suggests one.

- For a name that is a common word or close to one, use the company's full
  name (`{parent} {product}`, `{name} {legal suffix}`) and keep only titles
  containing the exact name. When nothing is left, say outside coverage found
  nothing.
- For a product inside a large company, news returns the parent's other
  launches; rely on the product's release notes and blog.
- Outside English-speaking markets, search the company name in the local
  language, without an English event word, with `--include-domains` set to
  the country's business and trade press.
- Drop reviews, "best X" lists, comparisons and how-tos, and for a public
  company drop buybacks, earnings notices and class-action releases unless
  they concern pricing or product. Keep items whose `published_date` is in the
  window, and drop one whose URL or byline shows an older date.
- When the user asks about security incidents, read the vendor's security
  advisories or trust page and, for open-source products, its GitHub security
  advisories, instead of searching news. Report "none found" with the sources
  checked.

## Collect

1. **Resolve** each product, its domains, any rename, and whether it serves
   the user's customer type. A competitor with two relevant products gets a
   column for each.
2. **Find pages**: pricing, release notes or their substitute, and the home or
   product page, for the user's market.
3. **Read prices** with the reference's ladder. A read counts only when it has
   the amounts in the user's currency at the user's size; an entry-tier price
   doesn't answer a question about a larger size. Record each plan's name,
   amount, currency, the period printed with it, and the limits that separate
   it from the next plan. For usage pricing, record the unit (credit, TiB,
   message, seat), the price per unit and the region. When the vendor prices
   by a different unit from the user's size (emails sent against contacts
   stored), show the arithmetic.
4. **Read every page that fills a matrix cell with a dated variant** (pricing,
   rate limits, deprecations, changelogs), on every run, and choose between
   the reads as the reference describes. When plans or models retire, read
   the deprecations or lifecycle page and leave retired items out of
   "current".
5. **Read recent changes** dated in the window. List the entries that touch
   pricing, plans, limits or the matrix features, plus major launches, and
   count the rest ("and 41 smaller entries").
6. **Fill features.** Find the page that says which plan includes each
   feature, in words. For a vendor that sells modules, name the module and its
   price. When a page confirms a feature without naming a plan, write "plan
   not stated". When no page for the user's market says it, a page for
   another market can, marked with its country. A cell with no page stays
   blank.
7. **Save the baseline**, one monitor folder per competitor (below).
8. **Stop** when every column has prices at the user's size or a label from
   the reference, and every matrix cell is filled or blank.

## Monitor

Watch the URL that returned prices and the changelog's index or feed, with the
flags that read them, as `tavily-monitor` describes. Use the user's cadence
("rerun next month" is monthly), or weekly. The watched lines are:

- one per priced item (a plan and printed period, a model, a country rate):
  `{item} | {amount} {currency} | {period or unit as printed, or "unlabeled"}
  | {key limits} | {region, when the page shows one by default}`; a model's
  input and output prices are separate lines;
- for a labelled price, one per plan name with the label, so a new or removed
  plan still shows;
- the newest 10 changelog entries, `{YYYY-MM-DD} | {title}`, using the first
  sentence cut at 80 characters when entries are paragraphs under a date
  heading, and only this product's entries on a page that covers several.

For a price labelled "shown by location", watch only the plan-name line,
since its currency changes between reads.

Copy the limits field from one place on the page in its own words, and
compare only the amount, currency, period and the numbers in the limits, so
an agent that phrases the line differently doesn't report a change.

On a rerun, read each monitored URL with its stored flags and a fresh
variant, and go back to the reference's ladder only when that read fails.
Diff as `tavily-monitor` describes. A pricing read with plan names but no
amounts failed, unless that item was stored with a label. A labelled item
that now shows an amount is `now published`. Replace each snapshot and save
the report.

## Parallel work

With subagents, start one per competitor in the same turn, with its domains,
the market, the customer size, the features, the window and its monitor
folder. Each returns its matrix column and section with a source URL for every
cell; write the matrix and summary yourself. If a call returns `blocked due
to excessive requests`, wait 60 seconds and retry one call at a time.

## Deliverable

Show the report in the reply, or the summary and matrix when the user asked
for something short. Save the full report to `intel/{title}.md`, with a title
of 3 to 6 words in sentence case, kept the same on reruns, and a dated copy of
each run as `intel/{title} {YYYY-MM-DD}.md`.

```markdown
# Competitive intel: {competitors}

Focus: {areas covered}
Market: {country, currency} · Customer: {customer type and size}
Window for recent changes: {start} to {end}
As of: {YYYY-MM-DD}
Previous run: {YYYY-MM-DD, or none}

## Changes since last run
- {competitor}: {item}, {old} to {new} | added | removed | now published ({url}, read {old date} and {new date})

## Summary
{1 to 2 paragraphs: how the competitors differ on price at the user's size and
on focus, the launches that matter, and where the user's product leads or
trails}

## Comparison matrix
| | {Competitor A} | {Your product} (you) |
|---|---|---|
| Pricing model | {per seat, usage, flat, per transaction} | |
| Price at {customer size} | {amount, currency, period, region, arithmetic} | |
| Plans | {one row per plan type the user asked about} | |
| Add-ons and fees at {customer size} | {add-ons the asked features need, per-transaction fees} | |
| Current offers | {price, end date and conditions} | |
| Usage limits | {units and window} | |
| {feature} | {plan that includes it} | |
| Free tier or trial | | |
| Who can buy it | | |
| Sources | | |

## {Competitor}
- Site: {domains}
- Positioning: {headline and segment, from the home or product page}
- Plans: {name, amount, currency, period as printed, key limits}
- Prices read from: {url and flags, read YYYY-MM-DD}
- Recent changes: {YYYY-MM-DD or YYYY-MM: entry} ({url})
- Outside coverage: {dated news items}
- Unread pages: {URLs that failed, and why}

## Coverage
Pages read: {per competitor}
Monitors: {folder paths}
Gaps: {labelled prices, freshness not checked, failed pages, pages that disagree}

## Rerun inputs
workflow: tavily-competitor-analysis
competitors: {name, domains, one per line}
market: {country, currency}
customer: {customer type and size}
features: {list}
window_days: {N}
monitors: {folder paths}
```

On a first run, leave out Changes since last run and say a baseline was
saved. Drop matrix rows that don't apply. With more than 4 competitors, put
competitors in rows. When columns are in different currencies, say so above
the matrix and don't convert.

## Quality bar

- Every price comes from the competitor's own site for the user's market, with
  its currency, period, region where it varies, and read date. When the
  vendor publishes the market's price only in another currency, use it and
  name the currency. Prices from another country's page or from a personal or
  consumer product of the same company are not used.
- The period is the one printed with the amount. A "from" amount is not a
  monthly price until a page states the period for that plan.
- A price the vendor converts at the day's exchange rate is marked converted.
- A total you compute (seats times price, a month times 12) is shown as
  arithmetic, never as a billing option the vendor doesn't offer.
- A missing price carries one of the reference's labels and is never
  estimated.
- A feature cell names the plan that includes it, from text that says so. A
  table row listing every plan name is not evidence.
- Every change in Changes since last run comes from a monitor diff with old
  and new values.
- Recent changes carry the competitor's published date. Write a month-only
  date as `YYYY-MM`, and mark one in the window's first month as possibly
  before the window. Mark a forum date as an activity date, and a change with
  no published date with the date it was read.
