# Reading competitor pages

Every call is a `tvly extract '<url>' --json`. Quote URLs that contain `?` or
`&`. One extract takes up to 20 URLs, and a URL that fails lands in
`failed_results` without failing the rest, so guessed URLs can share a call
with known ones.

## Prices

A pricing read counts only when it contains the amounts in the user's currency
at the user's size. A read with plan names and no amounts, or only the entry
tier when the user's size falls in a higher tier, failed even when the extract
reports success. Work down this ladder and stop at the first read that counts.

1. **Extract the page and its markdown copy in one call**: `{url}` and
   `{url}.md`, for the canonical URL and any locale URL you rewrote. The `.md`
   copy exists on many developer-tool and docs sites and often lists every tier
   as text; a link on the page labelled for LLMs or agents points to the same
   kind of copy. Elsewhere it returns 404, a stub that says the page wasn't
   found, a copy without prices, or only the yearly prices. When a read fails
   or shows placeholders (`--`, `Loading`), extract it again at
   `--extract-depth advanced`.
2. **Ask for the currency.** When the page shows another currency, add the
   page's currency parameter at advanced depth (`{url}?currency={code}`).
3. **Follow the page's own links to where the prices live**: a consumption
   table PDF, a legal fee schedule, a help-centre fees article, a per-country
   rate page, a downloadable rate sheet (CSV or PDF), or a docs page per model
   or endpoint. Extract these at advanced depth, and record a rate sheet's
   currency and valid-from date.
4. **Search the domain for a page whose snippet shows the amounts**, the
   sub-product first (`'{sub-product} pricing'`), then the plan name:
   `tvly search '{product} {plan name} price {size}' --include-domains
   {domain} --max-results 10 --json`. A locale variant of the page can carry
   amounts the canonical URL lacks.
5. **Label it**, with the URL:
   - "contact sales" when the page says so;
   - "public, not extractable" when the read shows a price widget with no
     amount: a placeholder (`{{selectedCountry.name}}`, `$0.00`, `Loading...`),
     a slider for the user's size, or a tab that isn't the default, since an
     extract at any depth reads only the default tab. Run step 4 first;
   - "not published" for a price the user asked about that no page mentions,
     such as volume discounts;
   - "shown by location" when the page follows the reader's region: reads of
     the same URL return different currencies, even with a locale path or a
     currency parameter, or a URL without a locale always returns another
     country's currency. Stop after the plain
     read and two variants, and record any amount with its currency;
   - "in-app only" when the vendor's help says prices are shown in the app.
     App-store pages list in-app purchases without amounts.

### Billing period and tiers

Copy the period words printed next to each amount ("per month", "billed
yearly", "prepaid for 12 months"). A plan with monthly and annual prices gets
both. Never convert one period into the other.

"From" and "starts at" mean either the lowest tier of a size-based price or
the annual price divided by 12, when the page also says "save N% annually" or
has a monthly and annual toggle. A read with "from" amounts doesn't end the
ladder. Take the period from the words,
the plan ID (`plan=standard_monthly_plan`) or the fee schedule.

When the price depends on size, give the tier that contains the user's size,
its limits, and any add-on needed to reach it, and show the sum. Add any
add-on that a requested feature needs and any per-transaction fee the use
involves. A vendor that converts its USD price at the day's rate has no local
list price; mark it converted.

Write a promotional price with its end date and conditions next to the regular
price. Conditions such as a minimum term or repayment on early exit can change
the price.

### Feature tables

Extraction can drop a table's check marks, so a row reads as every plan name.
It can drop a header cell, so values shift one column right, or a row's
leading empty cells, so a feature lands in a lower plan. Count values against
headers.
When they differ, read the dated variant or a page that names the plan in
words: plan cards, FAQ text, docs pages listing features by edition, or a
`.md` copy with words instead of icons.

When two pages on the vendor's own site disagree, record both with their
URLs.

## Choosing between reads

Read every page with a dated variant, as `tavily-monitor` describes for
stale copies. Beyond the newest-date rule:

- Two reads can both have amounts and still disagree, such as a copy that
  keeps an ended sale price or a retired model. Compare the plans or models
  in each read with other evidence first, and when amounts
  differ, run ladder step 4 with the plan or model name.
- When the fresher read names plans but has no amounts, run ladder step 4 with
  those names.
- When one read has dates and no text and another has text and no dates,
  combine them, and fill a missing entry from the news search.
- The same page can be published on several domains; read each and keep the
  freshest.
- When no read agrees with the other evidence, report the price with its read
  date and name the plan that is missing.
- When every variant fails, write "freshness not checked" under Gaps, and
  search news and the vendor's blog for plans or models launched in the window
  that the page may be missing.

## Recent changes

Use the changelog's feed when the page links one, as `tavily-monitor`
describes. Some changelogs live on a feedback tool with its own feed
(`{feedback host}/api/changelog/feed.rss`). Some put the date only in the
entry URL (`/releases/{YYYY-MM-DD}`). Some headings have no year and a long
page can mix two years; infer the year from the order of entries and the
nearest dated heading, and mark it inferred.

When a page covers only the last few weeks, follow its archive links one page
at a time, without guessing archive URLs, stopping at the first page whose
oldest entry is before the window. A page over a million characters still
returns in full: save it with `-o {file}` and keep the entries dated in the
window. Do not use `--query` on these pages, because it returns undated
fragments. When entries have no dates and no feed exists, take the window
from the dated sources below and list the page as unread for dates.

Banks, consumer apps, hardware, and software vendors that post monthly
roundups or email their updates have no changelog. Look, in order, at:

- the blog index, then its product category or hub, checking that the
  category is still in use;
- the newsroom for the user's market, or its feed;
- help-centre articles about changes to fees, terms, plans or billing, where
  pricing changes are often announced first;
- the version number or date on the fee schedule or terms PDF.

Two pages that name the same plan differently point to a rename; record the
name the fresher copy uses, with its date. A change that appears only in a
sibling product's document, such as a personal account's fee schedule, goes
under Gaps as a lead.

## Home and product pages

Use the product page when the company sells several products from one domain.
A home page that returns only a cookie banner (under 1,000 characters) failed;
use the product page or the newsroom headline.
