# Company filings and prospectuses

Read this when a player is listed, files outside the US, or has filed an IPO
prospectus.

## Filings

Extract the filing with a `--query` that names the product or segment and its
unit, such as `--query '{product} revenue {country}'`.

- US filers: under `--include-domains sec.gov`, pick the annual report by the
  fiscal year-end in its URL, such as `-20251231.htm`. Foreign filers on
  sec.gov file a 20-F.
- Other filers: search the annual report or shareholder letter on the
  company's investor site, with the reporting currency in the query. Query in
  the filer's language.
- Private companies that issue bonds publish quarterly results; search them as
  for a listed company.
- Convert currency at the average rate the filer states for that period. If
  it states none, give the rate and its source in Defaults assumed.
- Label every fiscal year by its end, such as FY3/2026, and keep calendar and
  fiscal years in separate rows.
- When a reported segment mixes the market with other products, report the
  shipped volume or label the revenue as broader than the market.

## Prospectuses

Search for one only when a player has filed for an IPO in the last two years,
with `'{company} DRHP'`, `'{company} prospectus'` or the local-language term,
filtered to the securities regulator's domain. A prospectus often contains a
commissioned industry report with its definition and method, and figures for
the listed competitors. Extract it without `--query`, save the output to a
file, and search it for the industry section and the peer tables.

## When an extract succeeds without the numbers

Tables stored as images and documents over a million characters come back
without the figures you asked for. Extract without `--query`, save the text
and search it. Otherwise search the figure in news that quotes the filing and
mark it quoted.
