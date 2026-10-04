# BASE (base-search.net) dataset-source coverage analysis

Date: 2026-09-30
Question: which data catalogs aggregated by BASE are missing from this registry?

## Method

BASE (494,476,125 documents, 11,833 content providers as of 2026-09-30) hides its
provider table behind an Anubis proof-of-work challenge, so the table was scraped
from a real browser session (cookies from the passed challenge) via same-origin
`fetch()` against `/about/en/about_sources_date.php`. Three passes:

1. **Provider-level, BASE's own classification**: all providers with
   `search_type=Datasets` → **205 sources** (`base_datasets_providers.json`).
2. **Platform-level**: all providers on dataset platforms regardless of BASE type —
   systems `Dataverse` (51), `Figshare` (91), `Digital Commons Data` (3), `Radar` (1)
   (`base_system_providers.json`).
3. **Record-level**: BASE search restricted to document type Dataset
   (`filter[]=f_dctypenorm:7`) — top-50 "Content Provider" facet, plus ~2,000 sampled
   dataset records across 20 queries and a newest-first sample; each record's
   `dccoll:` id mapped to a provider via the full 11,833-provider index
   (`base_all_providers.json`, `base_sampled_colls.json`, `base_newest_colls.json`).

Matching: BASE provider homepage domain vs registry `link` domains in
`data/datasets/catalogs.jsonl.zst` + `scheduled.jsonl`; flagged misses verified by
hand (registry `endpoints[]`, name search, redirect aliases, re3data).

## Findings

Registry coverage of BASE's dataset landscape is **complete**. Zero new catalogs
qualified for addition.

- Type `Datasets` (205): **204/205 already registered**; 1 (UFCAT, see below) is not
  a data catalog.
- Dataverse/Figshare/DCD/Radar (146, overlapping): all registered.
- Record-level sampling (~32 distinct collections): all registered.
- 4 initial "missing" domain flags were all false positives:
  - `rmit.figshare.com` → registry `rmitfigsharecom` (link `research-repository.rmit.edu.au`,
    alias covered in `endpoints[]`).
  - `melbourne.figshare.com` → registry `melbournefigsharecom` (same pattern).
  - `aussda.at` → org homepage; catalog `data.aussda.at` registered.
  - `periodicos.ufcat.edu.br` (Biblioteca Universitária - UFCAT, BR, OJS): BASE
    mislabels this e-journal platform as type "Datasets"; its 2,154 records are
    journal articles. Out of scope.
- Dataset records also surface from providers BASE types as "Publications" etc.
  Those already registered: Zenodo, Dryad, DataCite Commons, Smithsonian DR,
  Digital.CSIC, CGSpace, ORA Oxford, ROSA P, GFZ, CESSDA, SND, e-Prints Soton,
  UvA DARE, OpenAgrar, ETH Research Collection, IIASA PURE, and ~30 university
  figshare tenants.
- **Deliberately not added** (not data catalogs — publisher/library platforms whose
  datasets live on separate registered platforms): degruyter.com, brill.com (data on
  brill.figshare.com — registered), psycnet.apa.org, science.org, pmc.ncbi.nlm.nih.gov,
  f1000research.com (data on f1000.figshare.com — registered), oecd-ilibrary.org
  (OECD data surfaces on data-explorer.oecd.org etc. — registered), library.wur.nl.

## Artifacts

- `base_all_providers.json` — full 11,833-provider table (name, URL, docs, country,
  system, type, coll id).
- `base_datasets_providers.json` — 205 Datasets-type providers.
- `base_system_providers.json` — platform-filtered lists.
- `base_sampled_colls.json`, `base_newest_colls.json` — record-sampling evidence.
- `base_missing_candidates.json`, `base_missing_system_candidates.json`,
  `base_final_missing.json`, `registry_domains.json` — intermediate diff outputs.
