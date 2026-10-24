# web scraping n8n: The Official Node, Real Credit Costs, and How to Pick a ScraperAPI Plan Without Overpaying

If you search "web scraping n8n," you're usually trying to answer one question: how do I turn n8n into a scraper that actually survives real websites — ones with JavaScript, CAPTCHAs, and bot protection — without babysitting a broken HTTP Request node?

The short answer: n8n alone can fetch pages and parse HTML, but it has no answer for anti-bot systems. That's why most working setups pair n8n with a scraping API. The cleanest pairing right now is ScraperAPI, which ships an **official n8n community node** (`n8n-nodes-scraperapi-official`) that handles proxy rotation, user-agent rotation, CAPTCHA bypass, and JS rendering inside a single node — no HTTP Request gymnastics, no manually wiring API params into URLs.

> **Quick pick:** For most people testing the waters, start with the free trial — 5,000 credits, 7 days, no credit card required. If your workflow runs on standard HTML pages, Hobby at $49/month covers 100,000 standard requests. If you're scraping Google, Amazon, or protected sites at scale, the credit math changes completely (details below). 👉 [Start with the 7-day free trial and 5,000 credits](https://www.scraperapi.com/?fp_ref=coupons)

## Why the built-in HTTP Request node fails at web scraping in n8n

The HTTP Request node works fine for open APIs and simple static pages. It breaks down the moment a site checks whether you're a browser: you get a CAPTCHA page, a 403, or an empty shell because the content loads through JavaScript.

n8n's own strengths are scheduling (cron-like triggers), connectors to hundreds of destinations (Sheets, databases, Slack, email), and a visual workflow editor. None of that solves "the target site blocked me." A scraping API fills exactly that gap: your n8n workflow sends one request, and the API's infrastructure rotates through tens of millions of residential and mobile IPs, solves CAPTCHAs, renders JavaScript when needed, and retries failed requests before returning HTML or structured JSON.

Two ways to connect it in n8n:

1. **The official community node** — search "ScraperAPI" in the Nodes panel, click Install, paste your API key as a credential, done.
2. **The hosted MCP server** — for agentic workflows, point n8n's MCP Client Tool at `https://mcp.scraperapi.com/mcp` with HTTP Streamable transport and Bearer Auth using your API key.

There's also a plain HTTP Request route (calling `api.scraperapi.com` directly with your key), which works, but the official node gives you a settings UI for the parameters instead of query strings — less error-prone once you start combining render, geo, and proxy flags.

## Setting up the ScraperAPI node in n8n

Setup takes about two minutes:

1. Log into your ScraperAPI dashboard and copy your API key.
2. In n8n, open the Nodes panel, search for ScraperAPI, and install `n8n-nodes-scraperapi-official`.
3. Add the ScraperAPI node to your workflow, click **Credential to connect with → Create New Credential**, paste the key, and Save. n8n auto-tests the credential.
4. Pick a resource, enter the target URL, set optional parameters, and execute.

The node exposes four resources, and this is where it goes beyond what most n8n scraping tutorials cover:

- **API** — scrapes a single URL. Parameters include `render` (JS rendering, default off), `countryCode` (two-letter ISO code like US/GB/DE), desktop/mobile device selection, `premium` and `ultraPremium` proxy tiers (mutually exclusive), and `outputFormat` (markdown, text; JSON and CSV for supported auto-parse sites).
- **Structured Data Endpoint** — returns clean parsed JSON for supported sites: Amazon (product/search/offers), Google (search/jobs/news/shopping/maps), eBay, Walmart, and Redfin.
- **Crawler** — multi-page crawling jobs. You must set either `maxDepth` or `crawlBudget` (a hard cap on credits the job may consume — a genuinely useful safety net), regex rules for which links to include or exclude, and a `callbackUrl` webhook that receives both successful and failed results plus a job summary. Scheduling options: once, hourly, daily, weekly, monthly.
- **AI Parser** — build a reusable parser from a few example URLs, then apply it to any page with the same layout to get structured JSON.

One parameter worth flagging: `autoparse` returns parsed JSON on supported sites and **does not increase the request cost**. `sessionNumber` lets you reuse the same proxy session across requests, but it can't be combined with premium or ultra-premium proxies.

If you'd rather not build from scratch, ScraperAPI maintains pre-built n8n templates — including daily price-drop digests for Amazon, Walmart, and Google that you can import and adapt.

## ScraperAPI plans: full comparison

All plans include the same core feature set: JS rendering, premium residential and mobile proxies, CAPTCHA and anti-bot bypass, JSON auto parsing, rotating proxy pools, custom headers and sessions, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee. Support is priority-level across the board. Annual billing takes 10% off every paid plan.

| Plan | Monthly price (annual billing) | Credits | Concurrent threads | Geo-targeting | Get it |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | 1,000/mo | 5 | US & EU | [ Sign up free](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 ($44.10) | 100,000 | 20 | US & EU | [ Get Hobby](https://scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 ($134.10) | 1,000,000 | 50 | US & EU | [ Get Startup](https://scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 ($269.10) | 3,000,000 | 100 | Country-level | [ Get Business](https://scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | $475 ($427.50) | 5,000,000 | 200 | Country-level | [ Get Scaling](https://scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 ($877.50) | 10,500,000 | 300 | Country-level | [ Get Professional](https://scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 ($1,777.50) | 21,500,000 | 500 | Country-level | [ Get Advanced](https://scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | 22,000,000+ | 500+ | Country-level | [ Contact sales](https://scraperapi.com/pricing?fp_ref=coupons) |

New accounts also get a **7-day trial with 5,000 credits and no credit card required**, on top of the permanent free plan's 1,000 monthly credits. And there's a 7-day no-questions-asked refund policy on paid plans, so the first week of a subscription is effectively a test window. 👉 [Try ScraperAPI with 5,000 free credits](https://www.scraperapi.com/?fp_ref=coupons)

## The credit system nobody explains properly

This is the part that trips people up, and most n8n scraping tutorials skip it entirely: **credits are not pages.** ScraperAPI charges per request, but the per-request cost varies by domain and parameters.

Base domain multipliers:

| Target type | Credits per request |
| --- | --- |
| Standard page | 1 |
| Amazon, Walmart, eBay | 5 |
| Google, Bing (all subdomains) | 25 |
| LinkedIn | 30 |
| Cloudflare, Datadome, PerimeterX, Akamai bypass | +10 |

Parameter add-ons stack on top: `render=true` costs +10, `premium=true` +10, `screenshot=true` +10, `ultra_premium=true` +30. Combos are non-linear — `premium` + `render` together cost +25 (not +20), and `ultra_premium` + `render` costs +75. Meanwhile, `country_code`, `device_type`, `output_format`, `session_number`, `autoparse`, and `wait_for_selector` are all free.

What does that mean in practice? Run the numbers for a Hobby plan (100,000 credits):

| Use case | Actual pages you can scrape |
| --- | --- |
| Standard HTML | 100,000 |
| JS rendering (×10) | 10,000 |
| Amazon (×5) | 20,000 |
| Google SERP (×25) | 4,000 |
| Ultra-premium + JS (×75) | 1,333 |

So that "100,000 credits" is 100,000 pages for a lightweight news site, and about 1,300 pages for a Cloudflare-protected site with full rendering. Know your target before you pick a plan. ScraperAPI gives you three tools to stay on top of this: the Domain Cost Estimator in your dashboard (look up any URL's exact cost before scraping), the API Playground for testing targets at small scale, and a `max_cost` parameter to cap any single request. You can also query `https://api.scraperapi.com/account/urlcost` with your key and target URL to get the exact price programmatically — handy for checking costs from inside an n8n workflow before a big run.

Another detail worth knowing: **you're only billed for successful requests** — HTTP 200 and 404 responses — plus requests you cancel yourself before the 70-second window closes. Failed retries, blocks, and timeouts don't burn credits. Every response includes a `sa-credit-cost` header showing exactly what that request consumed, so you can audit real usage from n8n.

## Which plan actually fits an n8n workflow

- **Just prototyping in n8n?** The free plan (1,000 credits/month, 5 concurrent connections) plus the 7-day 5,000-credit trial is genuinely enough to build and test a full workflow. No credit card, and you can cancel any paid plan anytime without a charge.
- **Scheduled standard-HTML jobs** — product pages on simple sites, blogs, RSS-like sources. Hobby at $49/month is the right size. The 30-day analytics window and US/EU-only geotargeting won't matter for most individual workflows.
- **Amazon/eBay/Walmart monitoring** — multiply carefully. Amazon's 5× multiplier means Hobby's 100K credits is really 20,000 Amazon pages; Startup's 1M credits is 200,000. If your n8n workflow checks thousands of products daily, Startup is the honest starting point.
- **Google/Bing SERP tracking** — the 25× multiplier is brutal. 1M credits = 40,000 searches. Business ($299) or above is where SERP workflows live.
- **Expecting volume spikes?** Hobby through Startup have no overage option — credits run out, scraping pauses until renewal (or you opt into auto-upgrade, which bumps you to the next plan automatically). Scaling, Professional, and Advanced include Pay-As-You-Go: you keep scraping past your limit at a fixed per-credit rate, protected by a monthly spending cap you set, so a runaway crawler can't produce a surprise invoice.

For context on the competitive field: ScrapingBee maintains a popular open-source n8n no-code scraper template built around its AI Web Scraper API, and webscraping.ai lists an entry plan at $29/month for 250,000 credits with its own credit multipliers (JS rendering 5, residential proxies 10–25). Neither publishes a native official n8n node with Crawler and AI Parser resources the way ScraperAPI does — though the HTTP Request approach works with any of them, so if your targets are cheap-domain, low-volume, a smaller provider's entry plan can be the frugal choice. 👉 [Compare all ScraperAPI plans and current prices](https://scraperapi.com/pricing?fp_ref=coupons)

## FAQ

**Can I use ScraperAPI with n8n's AI Agent nodes?**
Yes. Besides wiring the ScraperAPI node into an AI Agent's Tool input (with a chat model and memory node alongside), ScraperAPI runs a hosted MCP server. Configure n8n's MCP Client Tool with the endpoint, HTTP Streamable transport, and your API key as the Bearer token, and your agent gets scraping as a tool call.

**What happens when I hit my credit limit mid-month?**
On Hobby through Professional, you can opt into auto-upgrade to move to the next plan automatically. Scaling, Professional, Advanced, and Enterprise can continue on Pay-As-You-Go at a fixed rate. Otherwise scraping pauses until your credits renew. Annual plans can re-subscribe before the next billing date to refresh their allowance.

**Does the free trial need a credit card?**
No. The 7-day trial gives 5,000 credits with no card required, and if that's not enough for testing, support can grant additional test credits.

**Is there a refund if it doesn't work for my target sites?**
Yes — 7-day no-questions-asked refund, handled through support. Combined with the free trial, you can validate your actual credit burn on your real target URLs before any money is at risk.

## Before you pick a plan

Do one thing first: run your three or four most important target URLs through the Domain Cost Estimator. If they're mostly standard pages, Hobby will feel generous. If they're Google, LinkedIn, or Cloudflare-walled, the multiplier math above tells you which tier actually matches your monthly volume — and it's often two tiers higher than the credit count suggests.

If your n8n workflows are mostly standard HTML with occasional JS rendering, start on Hobby's annual billing ($44.10/month) and keep `render=true` reserved for pages that need it. If you're building SERP or e-commerce monitoring that runs daily, Startup or Business with Pay-As-You-Go is the safer architecture — the spending cap means your crawler can't outspend your budget at 3 a.m. 👉 [Start your ScraperAPI trial and check your targets' real credit costs](https://www.scraperapi.com/?fp_ref=coupons)
