# Players: identity, owners, figures and deals

Read this when building the players table.

## Identify each player

Identify each player as `tavily-entity` does, by its name plus its own domain. In every later query, add the market term, or the parent's name for a unit. When a filing or funding record shows another name, search that name too.

A `tavily.com/knowledge/company/...` card in search results is a lead. Use its legal name and rounds as search terms, and confirm owner and funding on the company's own page, a filing or the funding announcement. Never cite its revenue.

## Figures by owner type

Find each player's owner first; it decides where the figures are.

| Owner | Where the figures are | Query |
|---|---|---|
| Listed company | Its filing; see `filings.md` | `'{company} {product} revenue {year}'` |
| Unit of a listed parent | The parent's annual report, by segment | `'{parent} annual report {year} {unit} net sales'` |
| Acquired by a listed company | The acquirer's next quarterly or annual filing | `'{acquirer} {target} revenue' --include-domains sec.gov` |
| Family, founder or foundation | The company's annual sales release | `'{company} financial year {year} sales {currency}'` |
| State-owned | The state agency's announcements and official statistics | local-language query with the agency's name |
| Subsidiary or joint venture with its own accounts | The national company registry, usually quoted by trade press | the registry's own terms for revenue and year-end, in its language |
| VC-backed | Funding announcements; revenue only as self-reported run-rate | see below |
| Taken private | The last annual filing and the deal announcement | last public annual revenue with its year, plus price and date |

When a parent's segment mixes the unit with other businesses, report the
unit's operating figures and write that its revenue is not disclosed.

## VC-backed funding

Search `'{company} {market} total funding raised Series'` with no filter. When
it shows no round from the last 12 months, also run `'{company} {market}
raises' --topic news --time-range year`. Funding trackers disagree on totals,
so give the sum of the rounds you found, or a total with its source, and say
which. Strategic investments by a customer or partner count as deals.

## Deals, entries, exits and insolvencies

Search the category, not each player, with `--topic news --time-range year`
and a concrete noun for the category: `'{market} startup acquired'`,
`'{market} acquires'`, `'{market} maker acquires'`.

- In a shrinking market, search insolvencies first:
  `'{market} supplier bankruptcy {year} {player}'`, and the local-language
  term for insolvency.
- One news query naming 3 or 4 players with `exits`, `enters`, `insolvency` or
  `non-renewal` finds entries and exits.
- The largest player's annual report lists deals in the segment.
