---
name: tavily-lead-gen
description: |
  Build a qualified prospect list from an ideal customer profile (ICP), using Tavily. Finds companies that match an industry, size, geography or a product trait, then checks each one for dated buying signals (funding or PE investment, hiring for a role, new locations, market entry, acquisitions or mergers, a launch, a technology project starting) and for negative signals such as a competitor already in place. Each company gets a fit grade from a stated rule, with a watch list, a disqualified section and a record that lets a rerun return only new companies. Use when the user wants leads, prospects, target accounts, an account list for outbound, companies that fit an ICP or show a trigger, or a refresh of an earlier lead list. For a plain table of matches with no qualification, use tavily-list. For enriching one account or a CSV of companies the user already has, use tavily-entity.
license: MIT
metadata:
  author: tavily
  version: "0.6.0"
  homepage: https://tavily.com
  source: https://github.com/tavily-ai/official-workflows
allowed-tools: Bash(tvly *)
---

# Tavily lead gen

Produce a ranked list of companies that match the user's ICP. Each qualified
company (grade A or B) carries at least one dated buying signal with the URL
that shows it, and a fit grade from the rule below.

Discover and deduplicate companies as `tavily-list` describes, identify each
one as "Identify the entity first" in `tavily-entity` describes, and search
and extract as `tavily-search` describes. If any isn't installed, run
`npx skills add tavily-ai/skills --skill <name>` and read its SKILL.md before
continuing. Discovery sources by signal, industry and region are in
[references/sources.md](references/sources.md); read it before searching.

## Intake

Split the request into gates and signals, and write both in the deliverable
header before searching.

- **Gates** are what a company must be. Write each as a test a page can pass
  or fail.
  - Size is a range with both bounds, in the unit the industry uses:
    employees, locations, revenue, funding stage, or total assets for banks
    and credit unions.
  - Geography means headquarters unless the user says footprint or office.
  - Traits separate buyers from look-alikes in the same industry, such as
    operators that run their own sites versus brokers that resell them.
    Confirm a trait on the company's product page or in a job post that
    describes the work. An exclusion the user states ("not X") is a gate,
    and a company that is X is disqualified.
- **Signals** are dated events that make the company worth contacting now:
  funding; a PE investment or recapitalization that leaves the company
  operating under its own name; hiring for a role, or a completed hire when
  the user says "hired"; new locations; entering a country the company didn't
  sell into before; acquiring another company or absorbing it in a merger; a
  launch; a technology project starting. A company qualifies with any one of
  the signals the user asked for.
- **The product-fit signal** is the event that shows the need the user's
  product meets, added even when the user didn't name it: a new site for an
  equipment seller, a migration project starting for a migration service.
  It never qualifies a company alone, and it can be the second event of an
  A.
- **Negative signals** lower a lead or rule it out: a finished project in the
  area the user sells; a deployment of, or contract with, a competitor's
  product in the last two years; the company being bought by or merged into
  another operating company, including a platform a PE firm already owns;
  and, when the user wants a company's first hire
  for a role, someone already in that role.
- **Flags** are conditions the user asked to mark, such as accounts already
  on a named competitor. A flag goes in the Flag column and never changes
  the grade, even when it would otherwise be a negative signal.

Default to a cap of 25 qualified companies. Expect 5 to 25 calls per
qualified company, more when the signal is rare; tell the user the cap and
the expected call count, and use subagents if available for the checks when
the cap is above 10. Windows run 90 days for open roles and launches and 180 days for
every other signal. Compute each window start once and pass it as
`--start-date`.

Ask at most 3 questions, chosen from these:

- Which signals count, when the user gave no trigger. Offer the two that fit
  their product; if the user doesn't answer, use them as the requested
  signals.
- Which size range, when the user gave none and the industry runs from single
  sites to national chains. Offer a range that matches who buys the product.
  When the industry's deals cross borders, ask in the same question whether
  the region means headquarters or footprint.
- Which titles count, when hiring is a signal. A role counts when it owns the
  function and reports to an executive or the board, whatever its title or
  level says. List the adjacent titles that pass that test (for Head of
  Data, also Director of Data Platform and Head of Data Engineering).
- Which companies to exclude and which products count as competitors:
  current customers, prior leads, the user's competitors, and companies that
  already use a product that rules them out.

## Collect

1. **Discover from the signal.** Pick the discovery source for each signal,
   industry and region from the reference table. In regulated industries
   (banks, credit unions, healthcare providers), start from the regulator's
   lists and filings, which give dated events and size in one place. Outside
   English-speaking countries, search job titles and local trade press in the
   local language, and funding news in English with the city
   (`'<city>-based <industry> raises Series B'`). Record each discovery angle
   with the count of candidates it added.
2. **Switch to the industry when a signal is rare.** After 10 calls, if
   discovery for a signal has produced fewer than 3 candidates whose page
   states that signal and suggests they pass the gates, build a list of
   companies in the segment instead (industry list pages, sector job
   trackers, VC portfolio job boards, and the "Similar companies" list on the
   profiles of your best leads) and check each one in step 5. After the first
   10 to 20, divide the hits by the companies checked. Below 10%, tell the
   user the expected number of leads and stop once that list is checked.
3. **Deduplicate** by website host with `www` removed, or by registry ID (a
   charter or license number) for regulated companies. Skip companies on the
   exclusion list and companies a previous run already recorded (see
   Reruns). When there are more candidates than the budget can check, check
   first the ones whose discovery page already states a gate (size, city,
   stage), and list the rest under Coverage.
4. **Resolve identity and check gates.** Identify each candidate as
   `tavily-entity` describes: the name plus its own website host, which goes
   into every later query. Look up 5 companies per call by passing one query
   per company to a single `tvly search` (it takes up to 5 queries and runs
   each separately): `'"<company>" company profile headquarters'`. A single
   query that ORs several names returns results for only one of them. Give a
   name that is a common word its city or product.
   - The `tavily.com/knowledge/company/...` results carry website, legal
     name, headquarters and LinkedIn URL in the snippet. Extract a profile
     for headcount and growth, funding rounds, M&A, detected technology or
     similar companies. Match it to the candidate by website host first; for
     chains, clinics and common names it often belongs to another company
     with the same name, so fall back to the company's own site. Detected
     technology is a lead to check, not a signal.
   - Profile headcount counts LinkedIn members, so treat it as a floor. A
     matched profile whose floor is above the upper bound disqualifies; a
     floor inside the range confirms only the lower bound, so in industries
     with many staff off LinkedIn (plants, clinics, warehouses) find a stated
     figure for the upper bound. A
     floor below the lower bound calls for a search of
     `'"<company>" employees'` (or the local word) for a stated figure; with
     none, and a floor under a tenth of the lower bound, record the company
     as Dropped even when it has a signal.
   - Take stage from the funding article itself, extracting the articles in
     one batch. Take revenue from the company registry or annual report, and
     assets from the regulator; the reference lists registries by country.
   - Read the boilerplate of every page you open for the company; press
     releases state size and HQ in their last paragraph. Before calling a
     gate unconfirmed, run one search for it.
   - When sources disagree on a gate, a regulator filing or registry wins,
     then the company's own site, then a dated news item, then data vendors,
     then the profile. Vendors that copy LinkedIn are not independent of it.
     Check HQ on the company's own contact or about page. Disqualify when the
     best source, or two independent sources, contradict the gate; otherwise
     the gate counts as unconfirmed.
5. **Check signals with one query per company.** Pass up to 5 single-company
   queries to one `tvly search`; a query that names several companies reads
   as "no signal" for all but one. Run
   `'"<company>" <domain stem>' --topic news --start-date <window start>`,
   where the domain stem is the website host without its suffix. It covers
   funding, deals, launches and new locations. When the stem is a common
   word, equals the name, or is a surname that news never prints, use the
   full legal name with the industry word instead.
   - When the reference marks the industry's trade press as undated, run the
     query again on those domains without `--topic news` and check each date
     on the page against the window.
   - When hiring is a signal, search the title on the region's job sources
     with the company name and domain stem, and take the jobs page link from
     the company's site or profile. For a completed hire, search
     `'"<company>" names <title>' --topic news`.
   - When the user wants a first hire for a role, run
     `'"<company>" "<title>"' --include-domains linkedin.com` for someone
     already in it.
   - When the user sells a product that replaces an incumbent, or asked to
     flag users of a competitor, check three places: the competitor's own
     site (`'"<company>"' --include-domains <competitor domains>`) for case
     studies and press, the company's job posts that ask for experience with
     the product (`'"<company>" "<competitor>"'` on the region's job
     sources), and the profile's detected technology. A query that pairs
     the company with competitor names returns the competitors' own
     comparison pages. When the budget is short, check A and B rows only
     and write "not checked" in the others.
   - For a merger, the continuing company is the lead and the absorbed one
     carries a negative signal. The regulator's merger notice or the notice
     of the members' vote says which is which.

   Record each signal, its date and the URL that states it. Check that the
   page is about this company and not its parent or a sister brand. One event counts once, even when it is both an acquisition and a new location; a site's announcement, groundbreaking and opening are one event, dated by the milestone the user asked for (an opening when they said "opening"). A store in a country where the company already sells online is a new location, not market entry.
6. **Grade** each company with the fit rule.
7. **Stop** at the first of these: the cap of A and B companies; the low hit
   rate in step 2; or two discovery angles in a row that add no A or B
   company, counted only
   after you have tried at least 3 rows of the reference table and checked
   the candidates those angles found. When you stop below the cap, say how
   many you found and which segment ran dry. Never fill the gap with C rows.
   Companies that qualify in the last batch beyond the cap go under Overflow.

## Dating and checking evidence

- The signal date is the date of the event. An article that reports an
  earlier go-live, round or deal dates the earlier event, and a roundup's
  publish date is not the event date: search the event for its first report.
  A scheduled event, such as a vote next month, is dated by its announcement.
- A snippet's date can belong to a sidebar or related link. Read the date
  from the opened page: the byline, the dateline, a "Published" or "Updated"
  line in any language, or a date in the URL path. An image path
  (`/2026/08/`) dates the article only when the image is the article's own,
  since sites reuse photos. When the page has no date, use the
  `published_date` of the `--topic news` result and write "(index date)"
  after it, after checking the page's year: `--start-date` lets undated pages
  through, and the index can give an old article a current date.
- A quarterly or monthly release that lists several deals dates them only to
  its period. When that period crosses the window start, find each deal's own
  date or treat the date as unconfirmed.
- A LinkedIn post URL's 19-digit activity ID encodes its time: the ID shifted
  right by 22 bits is milliseconds since 1970. Decode the post that holds
  the text you cite. Job IDs do not encode a date.
- Plans count only with a date and a commitment: an announced opening date, a
  signed lease, a filed permit. Sponsored or advertiser-written pages are not
  evidence.
- A job post counts when it is open on the date checked and, if its posting
  date is known, was first posted or reposted inside the window; give the
  latest posting date. A posting that can't be extracted is a lead, not a
  signal. Ashby, Greenhouse and Lever postings carry no date:
  look for the same post on a source that dates it (the reference lists
  them), or record "open as of {date checked}" and rank it below dated
  signals of the same grade.
- An open Greenhouse or Lever posting has an "Apply for this job" section and
  an open Ashby posting an "Application" tab. A closed Greenhouse posting
  redirects to "Current openings", a closed Ashby posting returns 404, and
  Built In says the job is removed. A closed posting is not a signal.
- Skip job posts from recruiters, staffing firms and consultancies hiring
  their own staff unless the page names the employer. A company seen only in
  a "similar jobs" sidebar is a candidate, not a signal.
- Match every job board, LinkedIn page and news item to the company by its
  website host; board slugs, LinkedIn slugs and generic names collide with
  other companies. Never guess a slug or a website.
- Use the company's current name and site; mergers and rebrands leave old
  ones in profiles and roundups. When a parent and a subsidiary differ, the
  row describes the one that passes the gates and names the other.

## Fit rule

- **A**: every gate confirmed, and two separate in-window events, each a
  requested signal or the product-fit signal, at least one of them
  requested. Two of the same type count when they differ in date or place.
- **B**: every gate confirmed and one requested signal in its window.
- **C**: a requested signal in its window but a gate unconfirmed or in
  conflict, no website found, or a B lowered by a negative signal.
- **Watch**: every gate confirmed and no requested signal in the window, or
  the only event is the product-fit signal, or a C lowered by a negative
  signal. Watch rows don't count toward the cap.
- **Dropped**: a gate unconfirmed and no requested signal. List the name
  under Coverage.
- **Disqualified**: the best source contradicts a gate, or a negative signal
  removes the reason to buy.

A negative signal moves a company down one level (A to B, B to C, C to
Watch). Write the reason in the Fit cell (`C: size unconfirmed`,
`C: acquired`). Rank A, then B, then C; within a grade, dated signals
first, newest first. If the user gives their own scoring, use it and write it
in the header.

Before saving, grade each A and B row again from its own cells. Name the two
events behind each A and check that each has an event date in its window and
a type from the header. Apply every negative signal written in the row. A
negative-signal check run as one query over several companies doesn't count
as checked.

## Parallel work

With subagents, give discovery to one agent per reference row. Split identity
and signal checks by batches of 10 companies. Give each agent the gates, the
signals, the window start dates, the exclusions and competitors, the query
shapes and the row fields. Run at most 3 agents at once; when a call returns
"blocked due to excessive requests", wait 60 seconds and retry it once. Merge
and deduplicate before grading.

## Deliverable

Show the list in the reply and save it to `leads/{title} {YYYY-MM-DD}.md`,
where the title is 3 to 6 words in sentence case that name the ICP; add
`-2`, `-3` when a file with that name exists. When the user wants a CSV,
write the A, B and C rows to the same name with `.csv`, using a CSV writer.

```markdown
# Leads: {ICP in a few words}

Gates: {industry; size range and unit; geography, HQ or footprint; traits}
Signals: {requested signal types}; product-fit: {signal}; windows: {days per type}
Flags: {conditions the user asked to mark, or none}
Fit rule: A = gates + two in-window events; B = gates + one requested signal; C = requested signal with a gate unconfirmed, or B lowered by a negative signal
As of: {YYYY-MM-DD}
Rows: {A} A, {B} B, {C} C, {Watch} watch ({N} upgraded from watch), {Disqualified} disqualified
Stop: {hit the cap | low hit rate | two angles added nothing}: found {N} of {cap}; {segment} ran dry

| Fit | Company | Website | Size | HQ | Signal | Signal date | Signal source | Negative signal | Flag | Gate sources |
|---|---|---|---|---|---|---|---|---|---|---|

## Overflow
{Companies that qualified beyond the cap, same columns; omit when empty}

## Watch
| Company | Website | Why it is worth watching | Last event and date | Source |
|---|---|---|---|---|

## Disqualified
| Company | Website | Failed gate or negative signal | Evidence |
|---|---|---|---|

## Coverage
Discovery angles: {each source or query, with the count it added}
Unchecked candidates: {names never checked, or none}
Dropped: {names with the unconfirmed gate}
Likely gaps: {segments these searches could not reach}

## Seen
| Company | Website or registry ID | Status | Last checked |
|---|---|---|---|

## Rerun inputs
workflow: tavily-lead-gen
industry: {industry}
size: {range and unit}
geography: {geography, HQ or footprint}
traits: {traits, or none}
signals: {requested signal types, with role titles or technology}
product_fit: {signal}
flags: {conditions, or none}
competitors: {products, or none}
window_days: {days per signal type}
cap: {N}
```

## Reruns

Seen holds every company this run and earlier runs checked, one row each,
with a status: A, B, C, Overflow, Watch, Dropped, Disqualified, Customer or
Excluded.

On a rerun, read the latest file and keep its Rerun inputs unless the user
changes them; write any change into the new block. Skip every company in
Seen except Watch rows last checked more than 7 days ago and Dropped rows
last checked more than 90 days ago; run steps 4 and 5 on those again.
Unchecked candidates under Coverage are eligible. Add new customers as
Customer and new exclusions as Excluded. The user's new cap counts new
companies only; count upgraded Watch companies in the Rows line. Carry every
Seen row forward with its latest status.

## Quality bar

- Every website host was read from a page the agent opened or a profile
  snippet, never typed from memory. A regulated company with no website is
  identified by its registry ID.
- Every signal has the event's own date, or an index date marked as one, and
  a URL that names this company.
- Size, location counts, revenue and assets come from an opened page, a
  registry, a regulator filing or a matched profile, not another site's
  snippet.
- A job post cited as a signal was extracted and showed an open application on
  the date checked.
- A disqualified company names the gate or negative signal and the page that
  shows it.
- Include only published business information: company pages, press
  releases, job posts, case studies. Never add a personal email or phone
  number. When the user asks for a contact, give the name and title from the
  company's own team page or a press release, with no contact details.
