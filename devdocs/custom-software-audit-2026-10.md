# Audit of `software.id: custom` entries — 9 October 2026

Working note. Scope: all entity YAML records with `software.id: custom`, the goal
being to find (a) records that actually run already-registered software, and
(b) repeated platforms that justify new software definitions.

## Method

1. Full YAML scan: 43,499 entity files, **6,972 custom** (16.0%).
2. Offline clustering: registrable domains (multi-tenancy), URL path segments,
   name tokens, SaaS tenant suffixes, declared `endpoints[]` types.
3. Live probe of a stratified sample of **1,651 URLs** (up to 4 per registrable-domain
   cluster + per-catalog-type quotas): HTTP status, `<meta generator>`, `<title>`,
   word-boundary platform markers, asset paths. First pass with `requests` hit a
   LibreSSL/urllib3 timeout artifact (63% false timeouts); failed rows were
   re-probed with `curl` (24/30 spot-check rows alive), recovering 855/1073.
4. Marker hits were cross-checked against `data/software/` definitions.

## Headline numbers

| Metric | Value |
|---|---|
| Custom records (YAML) | 6,972 of 43,499 (16.0%) |
| By type | Scientific 2,836 · Indicators 1,448 · Open data 1,143 · Geo 1,041 · rest <100 each |
| Sample reachable | 1,372/1,651 (83%) |
| Sample bot-blocked (401/403) | 103 (alive, likely fine) |
| Sample unreachable or 5xx | 155 (9.4%) — **140 of them still `status: active`** |
| Est. mislabels to existing ids (extrapolated) | **~500–700 rows** |
| Multi-tenant SaaS among custom | almost none (figshare/arcgis/opendatasoft tenants ≈ 10 rows total) |

Custom is dominated by bespoke institutional web apps (React/Next.js/Angular front
ends, PHP/Java back ends) and one-off domain databases — not by undiscovered
shared platforms. The recoverable value is mostly in **re-pointing to existing ids**
plus a short list of **new definitions** below.

## A. Mislabels — records that run already-registered software

Evidence in the 1,651-row sample (generator-proven = `<meta generator>`; mark =
page fingerprint). Extrapolation is per-catalog-type, sample is 24% of the population.

| Target `software.id` | Sample rows | generator-proven | Est. full set | Example rows |
|---|---|---|---|---|
| `wordpress` | 57 | 17 | ~230 | `wwwhawaiieduclimatedataportal`, `discwiscedu`, `nipfnknbsorke` |
| `drupal` | 33 | 20 | ~120 | `clearinghouseisgsillinoisedu`, `aimistanfordedu`, `wwwahrqgov` |
| ArcGIS family (`arcgishub`/`arcgisserver`/…) | 31 | 0 | ~145 | `livingatlasarcgiscom`, `citydrydenmapsarcgiscom`, `datatorontopoliceonca` |
| `geoserver` | 7 | 0 | ~35 | `visorcatastrogubuy`, `idesergeomingobbo` |
| `mapserver` | 7 | 0 | ~30 | `pubappnorthernbeachesnswgovau`, `adinetahacentreorg` |
| `joomla` | 9 | 2 | ~30 | `wwwhealthybelgiumbe`, `archivsoccascz` |
| `tableau` | 7 | 1 | ~27 | `seriestoricheistatit`, `budgetgouvht` |
| `geonetwork` | 6 | 0 | ~25 | `geoportaligrro`, `datrasicesdk` |
| `dataverse` | 4 | 0 | ~18 | `rugnlggdcetd`, `wwwrugnl`, `wdcbgsacuk` |
| `ckan` (incl. `*.dataeye.jp` SaaS) | 3 + 5 family | 3 | ~15 | `databiowagovau`, `opendatacomuneudineit`, `miyagidataeyejp` |
| `sharepoint` / `liferay` / `typo3` / `jekyll` / `plone` / `powerbi` | 3/3/3/3/2/2 | 2 (plone) | ~10–15 each | `landcopernicuseu`, `tradelinephilippinesdtigovph`, `laotradeportalgovla` |

Caveats:
- ArcGIS hits must be split per row (ArcGIS Online items → `arcgishub` or portal
  ids; "Item Replacement" titles are ArcGIS Online item pages).
- `geoserver`/`mapserver` markers may be the WMS behind a custom viewer — confirm
  the *portal* stack before re-pointing.
- `zenodo` (10), `figshare`, `eprints`, `samvera` hits are largely false positives
  (outbound links/DOIs on the page), not installations.
- 29 rows with `sdg` in URL/name (e.g. `pslmsdgsdatagovpk`) need a manual look —
  only some are actual OpenSDG instances.

Generator-proven rows are safe to re-point mechanically; marker-only rows need the
per-site check above or a second probe against the platform's capability URL
(`hunt.py probe --software <id>`).

## B. New software definitions worth adding

Multi-instance products currently without an id (ordered by expected payoff):

1. **`hugo`** — static site generator behind many `Datasets list`/project catalogs;
   `jekyll` already exists as `cms_or_app_framework`, Hugo has no id. 10 sample rows
   (~40 est.).
2. **`confluence`** — Atlassian Confluence used as a data catalog (3 sample rows,
   clear first-party product page → qualifies under the registry rule even at low
   count).
3. **`gatsby`** — same rationale as Hugo (6 sample rows), lower urgency.

Domain infrastructure (single-org but recognizable products, `domain_data_infrastructure`
precedent: PANGAEA, USGS ScienceBase, ESA Science Archive):

4. **NASA CMR / Earthdata** (`cmr`) — open source (github.com/nasa/cmr), powers
   `search.earthdata.nasa.gov`, Worldview, and DAAC archives; NASA alone contributes
   44 custom rows (heasarc, pds, gcmd…). Strongest domain candidate.
5. **EBI OLS** (`ols`) — Ontology Lookup Service, open source (EBISPOT/ols4), run by
   EBI and third parties; `ols4wwwebiacuk` plus related rows.
6. Optional, iconic single-instance resources — add only if the maintainers want
   per-resource ids: UCI ML Repository, gnomAD (Broad), EPA CompTox, WRDS, CiteSeerX.

Not recommended:
- **Looker Studio / Webflow / Wix / Squarespace** — generic site/dashboard builders,
  keep `custom`.
- **WTO portal suite** (18 rows) — internally built family, no shared fingerprint
  found (different stacks per portal); keep `custom`.
- **`satudata` as one id** — Indonesia's Satu Data is a *program*, not a platform:
  the 57 regional portals run mixed stacks (WordPress, Next.js, some CKAN). Handle
  as a per-row mislabel sub-project instead.
- **expasy.org / ebi.ac.uk / re.kr / res.in families** — per-resource bespoke
  databases; keep `custom` (individually they could follow the optional path in #6).

## C. Dead links

155/1,651 sample rows unreachable or 5xx (9.4%), 140 still marked `active`.
If representative, ~650 custom rows point at dead or unreachable sites (some are
geo-blocks from a US/EU vantage point — `.ru`, `.cn`, some `.gov` — so verify before
flipping status). Worth a dedicated liveness pass before the next release.

## Recommended next steps

1. Re-point generator-proven rows (CKAN×3, Drupal×20, WordPress×17, Joomla×2,
   Plone×2 from this sample) — cheapest, zero-risk wins.
2. Add `hugo`, `confluence` (+ optionally `gatsby`, `cmr`, `ols`) definitions and
   run the instance hunts per `docs/software-taxonomy.md`.
3. Full-population sweep of the remaining 5,321 unprobed custom rows with the same
   curl-based fingerprinter to convert estimates into exact lists.
4. Liveness pass over custom rows with `status: active`.

## Artifacts

Ephemeral analysis outputs: `/tmp/custom_yaml.json` (all 6,972 custom records with
paths), `/tmp/probe_results.json` + `/tmp/retry_results.json` (probe evidence),
`/tmp/mislabel_ids.json` (sample-proven mislabel row ids grouped by target id),
`/tmp/hits.json` (marker hits). Regenerate with the steps in *Method*.
