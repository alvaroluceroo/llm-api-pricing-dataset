# LLM API pricing dataset

Prices of the current large language model APIs, as published by their makers, with the day each price was last verified and the maker's page it was read from, plus a record of every change to those prices. This is the data behind the [LLM API pricing table](https://aisignalhq.com/api-pricing/) of [AI Signal](https://aisignalhq.com/), published here as files so it can be versioned, diffed and cited.

The same files are served at [aisignalhq.com/data/](https://aisignalhq.com/data/). This repository is a mirror of them: every commit is a snapshot of what the site published that day.

## Files

| File | Contents |
| --- | --- |
| [`models.json`](models.json) | Current models in the table's scope, one object per model. |
| [`models.csv`](models.csv) | The same rows as `models.json`, one line per model. |
| [`price-changes.json`](price-changes.json) | Every recorded change to a price the site publishes, newest first. |

Both JSON files share one envelope: `name`, `publisher`, `homepage`, `generatedAt` (when that copy was built, UTC), `license`, `licenseUrl`, `attribution`, `rowCount` and `rows`.

`models.csv` starts with three lines beginning with `#` that carry the name, build time, licence and attribution, followed by a header line. Skip them with `pandas.read_csv(path, comment="#")` or the equivalent option of your reader; no value in the file contains `#`.

```python
import pandas as pd
url = "https://raw.githubusercontent.com/alvaroluceroo/llm-api-pricing-dataset/main/models.csv"
df = pd.read_csv(url, comment="#")
```

## Scope

The model files hold models whose status is current, charged per token, with text output, and priced by their maker. Deprecated and retired models are not included. A row whose maker was checked and publishes no per-token price keeps its prices empty and says `no-published-price` in `priceStatus`: no figure in these files is estimated, rounded or filled in.

Left out on purpose: prices on inference hosts (Together, Fireworks, DeepInfra, ...), second price lists such as Batch or Priority, and the site's internal notes and working fields.

## Fields of models.json

| Field | Meaning |
| --- | --- |
| `id` | Stable identifier on this site: maker/model. |
| `apiId` | The model name the maker's API expects. |
| `maker` | Maker identifier (openai, anthropic, google, ...). |
| `makerName` | Maker name as printed on the site. |
| `name` | Model name. |
| `status` | Always "current" in this file. |
| `priceStatus` | "priced", or "no-published-price" when the maker was checked and publishes no per-token price (prices are then null). |
| `inputPerMTok` | Input price, USD per 1M tokens. |
| `outputPerMTok` | Output price, USD per 1M tokens. |
| `cachedInputPerMTok` | Cached input (cache read) price, USD per 1M tokens, or null. |
| `cacheWritePerMTok` | Cache write price, USD per 1M tokens, or null; for Anthropic, the 5-minute cache write. |
| `cacheWrite1hPerMTok` | Anthropic's 1-hour cache write, USD per 1M tokens, or null. |
| `priceRegion` | Region the price applies to when the maker prices by region (Alibaba: Singapore), else null. |
| `longPromptTiers` | Prices that apply to the whole request above a prompt length: condition, input, output, cachedInput. Empty when none is recorded. |
| `contextWindow` | Context window in tokens, or null when the dataset has none. |
| `maxOutput` | Maximum output tokens, or null when the dataset has none. |
| `promotion` | null, or the promotion the price belongs to: endsOn, atLeastUntil (the maker says "at least through"), and listPriceAfter when the maker publishes it. |
| `verifiedAt` | Day the price (or its absence) was last verified on the source page, YYYY-MM-DD. |
| `sourceUrl` | The maker's page the price was read from. |
| `pageUrl` | This model's page on AI Signal. |

`models.csv` has the same fields with nested ones flattened: the first long-prompt tier goes into `longTierCondition`, `longTierInputPerMTok` and `longTierOutputPerMTok`, and `longTierCount` says how many tiers the row has (the others are only in the JSON); the promotion goes into `promoEndsOn` and `promoAtLeastUntil`. Columns, in order: `id`, `apiId`, `maker`, `makerName`, `name`, `status`, `priceStatus`, `inputPerMTok`, `outputPerMTok`, `cachedInputPerMTok`, `cacheWritePerMTok`, `cacheWrite1hPerMTok`, `priceRegion`, `longTierCondition`, `longTierInputPerMTok`, `longTierOutputPerMTok`, `longTierCount`, `contextWindow`, `maxOutput`, `promoEndsOn`, `promoAtLeastUntil`, `verifiedAt`, `sourceUrl`, `pageUrl`.

## Fields of price-changes.json

| Field | Meaning |
| --- | --- |
| `id` | Stable identifier of the entry. |
| `effectiveDate` | Day the change took effect (a future day for an announced change), YYYY-MM-DD. |
| `provider` | Maker or host whose price it is, or "multiple". |
| `kind` | Who moved the figure: provider, correction (of this site's reading), omission (a row this site had missed) or criterion (a change of this site's rule). |
| `direction` | What the published figure did: increase, decrease, withdrawn or none. |
| `multiplier` | New figure over old, or null. |
| `affects` | Model ids or host API ids the entry names. |
| `figures` | Before and after prices (input, output, USD per 1M tokens) per model, when recorded. |
| `summary` | What happened, as printed on /price-changes/. |
| `sourceUrl` | Where the change can be checked. |
| `pageUrl` | The entry on /price-changes/. |

## How each price is verified

Every figure is read on a page the maker publishes (its pricing page, a model page or its documentation), and the row records that page in `sourceUrl` and the day in `verifiedAt`. In short:

- **Weekly detector.** Every Monday a script downloads the page each row cites for the makers it can read and checks that the row's input and output prices appear on it. A match must be backed by a line quoted verbatim from the page; a mismatch, a changed layout or a missing row goes to the editor, who reads the page and corrects the row by hand.
- **Manual review.** Makers whose pages a script cannot read, and rows with no published price, are checked by hand in a weekly review.
- **Corrections are recorded, never silent.** Every change, whether by a provider or a correction of the site's own reading, is an entry in `price-changes.json` with its date and source.

The full process, and what it does not guarantee, is described in the [methodology](https://aisignalhq.com/methodology/).

Providers change prices without notice. The authoritative source for every price is the maker's page in `sourceUrl`: confirm there before committing spend.

## Update frequency

The files are regenerated on every build of aisignalhq.com, and a new commit is pushed here automatically whenever the data changes. A build that changes nothing but the build time (`generatedAt`) does not produce a commit. `verifiedAt` on each row, not the date of the commit or `generatedAt`, says when its price was last confirmed.

## Licence

The data is licensed under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) licence. You may copy, share and adapt it, including commercially, provided you give attribution. See [LICENSE](LICENSE).

## Attribution

Attribution line, ready to copy, for a page, an app or a chart:

```
LLM API pricing data by AI Signal (https://aisignalhq.com/api-pricing/), licensed under CC BY 4.0.
```

Link to [aisignalhq.com](https://aisignalhq.com/api-pricing/) and say whether you changed the data. For a single price, citing the maker's page in `sourceUrl` as well is the most checkable form. For papers, use [CITATION.cff](CITATION.cff) (GitHub's "Cite this repository" button).

## Errors

To report an error, open an issue here or write to the editor from the [About page](https://aisignalhq.com/about/#contact).
