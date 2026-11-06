# Lead-gen sources

Discovery sources and query shapes by signal, industry and region. Use
`--depth advanced --max-results 10 --json` on every search. Each row is one
source type for the stop rule.

## Discovery by signal

| Signal | Source | Query shape | Filters |
|---|---|---|---|
| VC funding | Funding news, then weekly roundups | `'<city> <industry> raises Series A'` | `--topic news --start-date <window>` |
| VC funding, AI companies | aifundingtracker.com weekly tables, which carry HQ and region | `'Weekly AI Funding'` | `--include-domains aifundingtracker.com --start-date <window>` |
| VC funding, France | Maddyness weekly roundups | `'levées de fonds French Tech semaine'` | `--include-domains maddyness.com --start-date <window>` |
| PE investment, acquisitions | The industry's trade-press deal roundups (dentistry: Group Dentistry Now monthly roundups, Becker's quarterly and state-by-state lists) | `'<industry> deal roundup <month> <year>'` | `--include-domains <trade press>`, no `--topic news` |
| PE, credit facilities, M&A in logistics | Logistics trade press | `'<industry noun> recapitalization'`, `'<industry noun> acquires'` | `--include-domains dcvelocity.com,logisticsmgmt.com --start-date <window>` |
| Mergers, US law firms | Fairfax Associates' quarterly merger releases, which give lawyer counts for both firms and the deal date | `'Fairfax Associates law firm mergers Q<n> <year>'` | none; extract the PDF |
| Mergers and core-system changes, credit unions | The NCUA table of proposed mergers, which names the merging and continuing credit union with dates; core vendors' quarterly client releases | `'NCUA proposed credit union mergers comments'`, `'credit union selects <core vendor>'` | `--include-domains ncua.gov`; `--topic news --start-date <window>` |
| New locations, warehouses and plants | News, with the industry noun in the query | `'<industry noun> announces opening new warehouse facility'`, `'<industry noun> opens new distribution center'` | `--topic news --start-date <window>` |
| New plants and expansions, US manufacturing | State economic-development newsrooms, which state the jobs created and retained | `'<industry> manufacturer expansion jobs <state>'` | `--start-date <window>`; extract the state release |
| Market entry, replatforming, Nordic e-commerce | Swedish trade press; listed companies' releases on mfn.se; platform vendors' customer stories | `'expanderar till nya marknader lanserar e-handel i <land>'` | `--include-domains ehandel.se` |
| Hiring, US startups | Ashby, Greenhouse and Lever | `'"<exact title>" <industry>'` | `--include-domains jobs.ashbyhq.com,job-boards.greenhouse.io,jobs.lever.co` |
| Technology project starting | Implementation partners' newsrooms and the vendor's project job titles | `'Go-Live <technology> <industry>'`, `'<technology> Einführung Projektleiter'` | `--include-domains <partner domains> --start-date <window>` |
| Product trait | Job posts that describe the work | `'"<role>" "<trait phrase>"'` | the hiring filters for the region |
| Rare signal, any industry | Industry list pages and sector trackers (healthtechjobs.uk for UK healthtech with HQ and stage; the Inbound Logistics Top 100 3PL list) | `'<industry> headquartered in <region> list'` | no date filter; check each company's signal afterwards |

## Notes by source

- **Funding news.** A city in the query returns articles that state the HQ.
  Very large rounds usually mean the company is above a mid-market headcount
  cap. Strategic investments and debt count as funding; say which in the row.
- **Funding queries that return vendors.** A funding query about the buyer's
  industry often returns startups selling to it. Put the buyer's own noun
  and deal words in the query, or search the buyer's trade press.
- **PE and M&A wording.** Services industries announce deals as "partners
  with", "recapitalization", "welcomes" or "affiliation", not "raises". Deal
  lines name the buyer's HQ, which is often in another state than the
  business it bought.
- **Undated trade press.** Becker's and Group Dentistry Now pages come back
  without `published_date`, so `--topic news` searches skip them. Extract
  Becker's at `--extract-depth advanced`; basic depth fails and advanced
  returns the byline date. Becker's state-by-state lists state each group's
  HQ. Monthly roundups republish older deals.
- **Sources that date postings.** healthtechjobs.uk ("Posted <date>").
- **Recruiter wording.** "Unser Mandant", "pour notre client" and "our
  client" mark a recruiter posting.
- **Technology.** For a migration service, a finished go-live is a negative
  signal and an announced project or a project-lead job post is the positive
  one. Vendors' customer-story sections favour large global brands; partner
  newsrooms name mid-market customers. A competitor's newsroom
  (`'<company> selects <competitor>'`) shows who already chose it.
- **Regulators.** For US credit unions, merger notices on
  `downloads.regulations.gov` carry branch addresses and balance sheets,
  which separate same-name credit unions in different states. A credit union's "assets under management" is
  not total assets.
- **State economic-development releases.** A release about a plant states
  the jobs it creates and retains, which is the best size figure for a
  private manufacturer whose plant staff are not on LinkedIn.
- **Registries for revenue.** UK: Companies House. Data-vendor revenue for
  private companies can be off by an order of magnitude.
- **Company profiles.** `tavily.com/knowledge/company/...` pages appear only
  for queries that name a company, so they are not a discovery source. Their
  funding lists can trail recent rounds by months, and companies with many
  staff off LinkedIn show a fraction of their headcount.
- **Vendor-written list pages.** Take names only; AI-written lists contain
  companies from other countries and invented location counts.
