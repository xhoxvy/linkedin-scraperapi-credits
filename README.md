# LinkedIn Data Extraction: Which Fields Still Return Data, What 10,000 Profiles Cost in Credits, and How ScraperAPI's Plans Line Up

LinkedIn's Position, Experience and Education fields have been restricted since November 13, 2025. Bright Data's field-status documentation puts the fill rate on those three at roughly 2%, down from 50–64%, and notes they now come from cache rather than a fresh scrape. Name, headline, current company, follower count and About text still refresh on request, so most recruiting and sales lists remain buildable, just without a dependable work-history column.

Below: what you can still pull today, what a LinkedIn page costs in ScraperAPI credits at 1,000, 10,000 and 100,000 profiles, which of its plans covers which volume, and where LinkedIn-specific data APIs and do-it-yourself Playwright scrapers run into trouble.

> **Quick pick.** A LinkedIn page costs **30 credits** on ScraperAPI. The free plan's 1,000 monthly credits cover about 33 profiles; the 5,000 credits you get in the first seven days cover about 166, which is enough to measure your real fill rate before paying anything. Past that, buy the tier whose monthly allowance covers your page count at 30 credits each.
>
> 👉 [See ScraperAPI's current plan allowances and per-credit prices](https://www.scraperapi.com/?fp_ref=coupons)

## What LinkedIn Still Returns, and What It Stopped Returning

Profiles, company pages, job listings and posts are still scrapeable. Work history, education and the structured position field are not, at any volume worth planning around.

The restriction is a platform-level change, not a vendor limitation, so it applies whether you use a managed API, a LinkedIn dataset, or your own browser automation. Bright Data's documentation is unusually specific about it: since November 13, 2025 those three fields dropped to roughly 2% fill and are served from cache; `current_company.industry` has been unsupported since August 7, 2025; every other field still triggers a live scrape. Because the cached fields don't refresh, paying for a bigger plan or a premium proxy tier won't make them reappear.

One practical consequence: a tool that advertises "experience, education and skills" without mentioning the restriction is either serving you stale cache or hasn't re-tested since late 2025. The LinkdAPI-based command-line scraper on GitHub, for example, lists experience, education and skills as scrapeable with no note about the change. That doesn't make it useless, but it does mean the claim needs checking before it goes into a pipeline.

The check itself is cheap. Run 20–50 profile URLs through whatever tool you're evaluating and count how many come back with non-null experience entries. Around 2% means you're seeing the live platform. Anything near 60% means you're looking at cache, and you should ask how old it is.

Response times also move for reasons outside the vendor's control. Bright Data explains that LinkedIn changes how pages are served, collectors get updated to match, and an update can change how long a single scrape takes even when the success rate holds. If you're building anything production-grade, plan for asynchronous jobs and webhook delivery rather than assuming a fixed latency.

## ScraperAPI's Plans and the 30-Credit LinkedIn Rate

ScraperAPI bills by request count, not bandwidth, and it prices harder domains higher. LinkedIn sits at the top of its published list at **30 credits per request**, against 1 credit for a standard page, 5 for Amazon and 25 for Google or Bing. Requests that need bot protection bypassed (Cloudflare, DataDome, PerimeterX) add 10 credits each.

Credit allowances and thread counts scale with each tier. The figures below are the ones ScraperAPI states in its pricing FAQ and documentation; the pricing page carries the current allowance and price for every plan, which is where the links go.

| Plan | Concurrency | When credits run out | Current pricing |
| --- | --- | --- | --- |
| Free | 5 threads | Resets next cycle; 1,000 credits/month, 5,000 in the first 7 days | [Start on the free plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | Set by tier | Auto-upgrade to the next plan, otherwise pauses | [Check Hobby's allowance](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | Set by tier | Auto-upgrade, otherwise pauses | [Check Startup's allowance](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | Set by tier | Auto-upgrade, otherwise pauses | [Check Business's allowance](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling | Up to 200 threads | Auto-upgrade or pay-as-you-go with a monthly cap | [Check Scaling's allowance](https://www.scraperapi.com/?fp_ref=coupons) |
| Professional | Set by tier | Auto-upgrade or pay-as-you-go with a monthly cap | [Check Professional's allowance](https://www.scraperapi.com/?fp_ref=coupons) |
| Advanced | Set by tier | Pay-as-you-go only | [Check Advanced's allowance](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | Custom; sales quotes 300+ threads and up to 50M credits for custom trials | Pay-as-you-go plus custom terms | [See Enterprise terms on the pricing page](https://www.scraperapi.com/?fp_ref=coupons) |

That auto-upgrade versus pay-as-you-go split is the part people miss. On Hobby through Professional, running out of credits either bumps you to the next plan or stops your scraping until renewal. On Scaling and above you can keep going at a fixed per-credit rate inside a spending cap you set yourself, and the cap is a limit rather than a pre-charge. If your LinkedIn volume is spiky, that difference matters more than the headline allowance.

Allowances go well into the millions of credits on the upper tiers; one plan on the pricing page shows 10,500,000 credits with 300 threads. Annual plans hand you the full year's credits upfront and let you renew early for a fresh allocation, which suits a one-off backfill of 100,000 profiles better than a monthly subscription does.

Turning credits into pages is simple arithmetic:

| Profiles per month | Credits at 30/page | What it implies |
| --- | --- | --- |
| 33 | 1,000 | Free plan, exactly |
| 166 | 5,000 | First-week trial credits |
| 1,000 | 30,000 | First paid tier territory |
| 10,000 | 300,000 | Mid-to-upper paid tier |
| 100,000 | 3,000,000 | Annual plan or pay-as-you-go |

Add 10 credits per request if LinkedIn forces an extra bypass on your traffic, which pushes 10,000 profiles to 400,000 credits. Divide your plan price by its credit allowance once to get your per-credit rate, then multiply by 30 for a single profile. That number is the one to compare against per-record pricing elsewhere.

👉 [Compare credit allowances across ScraperAPI's plans](https://www.scraperapi.com/?fp_ref=coupons)

## Running a LinkedIn Pull Through ScraperAPI

You send a URL to the API endpoint with your key and ScraperAPI handles proxies, retries and blocking. The parameters that matter for LinkedIn:

- `url` for the profile, company or job page you want
- `render=true` when the page needs JavaScript to fill in
- `output_format=markdown` or `text` to skip HTML parsing
- `country_code` to send the request from one of 100+ locations
- `premium` to force residential or mobile proxies, `ultra_premium` for the hardest targets
- `device_type` to choose desktop or mobile rendering

Typical responses come back in 1–5 seconds; heavier pages like job listings or long profiles usually land within 10. Exceeding your plan's concurrency returns a 429 rather than a bill, and the request is turned away instead of queued, so backoff logic belongs in your client. For recurring syncs, DataPipeline endpoints let you schedule jobs and push results to a webhook.

The output is page content, not a LinkedIn-shaped JSON object. You get the HTML, markdown or text and write the extraction yourself. That's the real trade-off against a LinkedIn-specific dataset API: no field schema to learn, but a parser you own and have to maintain when LinkedIn's markup moves. `max_cost` per scrape is worth setting, since a mistyped URL that lands on LinkedIn costs 30 times what a normal page does.

Worth knowing: ScraperAPI's own LinkedIn tutorial avoids login walls entirely, scrapes the public guest job endpoints, and warns that pulling data from behind a login "can have legal implications and may violate LinkedIn's terms of service."

👉 [See the LinkedIn endpoint parameters and a full example payload](https://www.scraperapi.com/solutions/linkedin-scraper?fp_ref=coupons)

## Four Ways to Get LinkedIn Data, Side by Side

| Route | What you get | Cost signal | Where it breaks |
| --- | --- | --- | --- |
| Managed scraping API (ScraperAPI) | Raw page content as HTML, markdown or text; you parse | 30 credits per LinkedIn page | No login, so the restricted fields stay restricted; you maintain the parser |
| LinkedIn-specific scraper API (Bright Data) | Structured JSON, NDJSON, CSV or Parquet per data type | Pay per successful record; published rates start at $0.75 per 1K records, and pre-collected LinkedIn datasets start at $250 per 100K records | Same platform-level field restrictions; discovery jobs run asynchronously and take longer |
| Third-party profile API wrappers (LinkdAPI CLI) | Structured profile JSON including claimed experience, education, skills and contact fields | Its own credit system; README suggests 5–10 concurrent requests on trial, 30–50 on enterprise | Claims fields that LinkedIn restricted in November 2025; verify fill rate before committing |
| Self-hosted Playwright with your own account | Structured JSON and CSV, including the contact-info overlay | Your compute, your proxies, your account risk | 2FA, session expiry, DOM changes returning nulls, and connection-degree caps |

The last row deserves a number. The Playwright project's own README reports a company page listing 247 associated members where it extracted 181, because LinkedIn limits employee visibility by connection degree. That gap exists in every login-based approach and grows the further your target is from your network.

Bright Data is the closest structural alternative for pure LinkedIn work: 5,000 free credits a month for new accounts, a `POST` to a dataset endpoint with a per-data-type ID, synchronous results when a job finishes inside a minute and HTTP 202 with a snapshot ID otherwise, and up to 1 GB of input per async job. It's the option to price against if LinkedIn is your only data source and you want records rather than pages.

## Account Risk, in the Tool Authors' Own Words

None of this is legal advice, and the projects below say it better than a summary can.

The Playwright extractor's README states: "Scraping LinkedIn directly may violate their terms of service. This tool is for educational purposes." It also warns about credential security and rate limits, and caches a `session.json` after a manual 2FA entry so you don't re-verify every run. The LinkdAPI CLI's disclaimer limits the tool to educational and research use and puts responsibility for LinkedIn's terms, GDPR and CCPA compliance, and any required permissions on the user.

Login-based scraping carries a second, non-legal cost: the account itself. ScraperAPI's tutorial sidesteps it by using a burner account and a proxy, which tells you what the failure mode looks like. Unauthenticated scraping avoids the account question entirely and gives up the login-gated fields, which as of November 2025 are mostly cached anyway.

## Picking a Plan by Volume

Match the plan to the page count, not to the feature list:

- **Under ~30 profiles a month, or still validating a use case:** stay on the free plan. 1,000 credits and 5 threads are enough, and the 5,000 first-week credits let you test a real sample.
- **A few hundred to a few thousand profiles a month:** the smallest paid tier that covers 30 credits per page, with auto-upgrade on so a busy month doesn't stall mid-job.
- **10,000+ profiles a month, on a schedule:** Scaling or Professional, with pay-as-you-go enabled and a monthly cap so a spike doesn't force a plan jump you didn't plan for.
- **One large backfill:** an annual plan, since the year's credits arrive upfront and can be spent on any timeline.
- **Company pages and job listings rather than profiles:** the same 30-credit domain rate buys far more fields per request, which lowers your cost per data point considerably.

Three things to confirm before you pay. Set `max_cost` per scrape so a LinkedIn URL can't quietly cost 30 credits when you expected one. Set the pay-as-you-go spending cap if your tier supports it, since that cap is enforced rather than advisory. And keep the 7-day no-questions-asked refund window in mind: it's long enough to run a 500-profile sample through the real endpoint and count your fill rates before committing to an annual term.

If your list is names, headlines, current employers and follower counts, most of the value is still there at 30 credits a page.

👉 [Start on the free plan and size up from your real credit burn](https://www.scraperapi.com/?fp_ref=coupons)
