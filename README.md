# scrape javascript rendered page: Skip the Headless Browser Maintenance, and See What Each Page Actually Costs

You searched for this because your scraper probably returned a beautiful, empty HTML shell. The content loaded fine in your browser, but your script got nothing. That means the data you want is being rendered by JavaScript after the initial HTML arrives, and plain HTTP requests will never see it.

There are three real fixes: read the data that's already buried in the raw HTML, call the hidden API the page itself uses, or actually render the page in a browser. Below I'll walk through all three, and then look closely at the third option through the lens of ScraperAPI, a scraping API with JavaScript rendering built in, because once you decide you need real rendering, the question stops being "how do I code this" and becomes "what does each page actually cost me." That second question is where most reviews go quiet.

> **Quick verdict:** If your target pages need real browser rendering and you're doing roughly 1,000–100,000 pages a month, ScraperAPI's Hobby plan at **$49/month** (or $44.10/month billed annually) covers 100,000 credits with `render=true` included in every plan. You can test it first: a 7-day trial with 5,000 credits, no credit card required. 👉 [Check ScraperAPI's plans and start the free trial](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Three ways to scrape a JavaScript-rendered page

Short answer: try the cheap tricks first, and only fall back to actual rendering when they fail. Modern sites built with Next.js, Nuxt, or similar frameworks often embed a complete JSON dump of the page's data in the raw HTML, inside script tags like `<script id="__NEXT_DATA__" type="application/json">` for Next.js or `__NUXT_DATA__` for Nuxt. Redux apps frequently expose `window.__INITIAL_STATE__`, and Apollo clients leave a `window.__APOLLO_STATE__` object sitting in the page. If your target has one of these, you can parse the JSON directly from the first HTTP response. No browser, no rendering, one credit instead of ten.

The second route is finding the API the page calls behind the scenes. Open your browser's network tab, reload, and look for the XHR/fetch requests returning JSON. Many sites will serve that endpoint directly if you send the right headers, though some will return 403 unless you also carry the session cookie from a manual browser visit.

The third route, real rendering, is what you're left with when the site sets JavaScript-based tokens, runs aggressive anti-bot checks, or hydrates content that never appears in any embedded JSON. Most tutorials cover the coding side of this exhaustively: launch Playwright or Puppeteer, wait for a selector, extract from the rendered DOM. What they skip is the operational cost. Running headless browsers at any volume means maintaining a browser fleet, rotating residential proxies so you don't get IP-banned, solving CAPTCHAs, and retrying failures. That's a part-time job with a server bill attached.

## How ScraperAPI handles the rendering for you

Instead of you running the browser, ScraperAPI runs it. You send one HTTP request with your target URL and an API key, and add `render=true` when the page needs JavaScript rendering. Their infrastructure spins up a headless browser, waits for the content, and returns the finished HTML. Proxy rotation across a pool they advertise at 40 million+ IPs (residential, datacenter, and mobile, across 50+ countries), CAPTCHA solving, header management, and automatic retries all happen on their side.

For React, Vue, and Angular single-page apps specifically, a developer who ran the service in production for a year reported that the rendering "handles React/Vue/Angular SPAs consistently," in contrast to competitors where JS rendering sometimes means a bare headless Chrome that breaks on half of sites. Success rates were reported around 95–99% for most targets, with auto-retry handling transient failures.

Integration effort is genuinely minimal: add a URL parameter to your existing request code. There's no Playwright config, no proxy pool to babysit, no CAPTCHA-solving subscription to manage.

One parameter worth knowing early: `premium=true` routes your request through harder-to-block residential and mobile pools for protected targets, and `session_number` keeps the same IP across a sequence of requests. More on the cost implications of those in a moment.

## Full plan and pricing table

ScraperAPI currently lists seven tiers on its pricing page. Every plan includes JavaScript rendering, premium residential and mobile IPs, CAPTCHA and anti-bot handling, JSON auto-parsing, automatic retries, unlimited bandwidth, and a 99.9% uptime target. Annual billing takes 10% off the monthly rate.

| Plan | Monthly price | Annual (per month) | API credits | Concurrent threads | Get started |
| --- | --- | --- | --- | --- | --- |
| Hobby | $49 | $44.10 | 100,000 | 20 | [Start with Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [Start with Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [Start with Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling (most popular) | $475 | $427.50 | 5,000,000 | 200 | [Start with Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [Start with Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [Start with Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Contact sales for Enterprise](https://www.scraperapi.com/pricing?fp_ref=coupons) |

A few details that don't fit in the table: Hobby and Startup are limited to US and EU geotargeting only; country-level targeting starts at Business. Hobby's analytics dashboard only shows the last 30 days of usage. Dedicated account management and Slack-channel support are Enterprise features.

## What a rendered page actually costs you

This is the part the pricing page's headline numbers obscure, so let's do the math properly.

Every request consumes credits, but not at a flat rate. A standard page costs 1 credit. Rendered requests cost roughly 10 times that, since you're paying for a headless browser session. And specific hard targets have their own published rates: Amazon pages cost 5 credits, Google and Bing (including subdomains) cost 25, LinkedIn costs 30. If a request runs into bot protection like Cloudflare, Datadome, or PerimeterX and ScraperAPI bypasses it, that adds 10 credits on top.

Stack those multipliers and the gap between nominal and effective pricing gets wide. On the Startup plan, one credit works out to about $0.000149 on paper. Add `render=true` plus `premium=true` against a protected target, and a single successful scrape can consume 20–30 times that headline figure. One independent review ran the numbers on Amazon search pages with rendering enabled and estimated that a million credits might realistically cover 40,000–100,000 actual pages rather than a million. On the Business plan, a plain page costs about $0.10 per thousand, while a rendered one runs about $1.00 per thousand, a useful anchor for budgeting.

Three tools keep this from turning into a surprise bill:

- **Domain Cost Estimator**, built into the dashboard, tells you the exact credit cost for any URL before you burn quota on it.
- **`max_cost`** caps how many credits any single request can consume.
- **Pay-as-you-go** (on Scaling and above) lets you keep scraping past your credit limit at a fixed per-credit rate, up to a monthly cap you set yourself.

If you only take one action from this article, run your target URLs through the cost estimator before committing to a plan. 👉 [Open the pricing page and size up your real per-page cost](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Limits you only find out about later

Several restrictions sit in the documentation rather than the marketing page, and each one has bitten real users:

- **Concurrency overages return 429 errors, not a queue.** When you hit your thread limit, requests are rejected outright rather than delayed. Hobby's 20 concurrent threads can bottleneck a time-sensitive job like price monitoring, and you'll need to manage thread limits in your own code.
- **Unused credits don't roll over.** What you don't spend in a billing cycle is gone when the cycle renews.
- **Sessions top out around an hour.** `session_number` keeps one IP for a single scraping run, which works fine for most extraction jobs but rules out long-running logged-in workflows or day-long ad verification.
- **No fixed IPs.** You can't pin a specific IP or ASN; you choose parameters and trust the rotation logic.
- **No login-state scraping.** Each request runs through a different proxy by default, so authenticated scraping needs workarounds.

None of these are dealbreakers for the typical "grab product data off a JS-heavy storefront" use case. They become real problems for account management, ad verification, or high-frequency monitoring.

## When a ScraperAPI alternative makes more sense

Honesty requires saying this plainly: at $49/month, ScraperAPI has the most expensive entry tier in its category, and there are cheaper doors into the same problem.

ScrapingBee starts at $19/month and scores comparably in recent reviews (4.0 versus ScraperAPI's 4.0 on Huzzler), making it a reasonable pick for general scraping tasks where you're not hammering search engines. Firecrawl, also $19/month, rates higher (4.5) and outputs clean Markdown, which matters if you're feeding pages to AI agents; ScraperAPI has no native Markdown output and no agent-protocol integrations like MCP. ScrapeOps acts as a proxy aggregator across multiple upstream providers and tends to cost less per request if you're willing to handle rendering yourself. Bright Data is more powerful overall but significantly more complex and expensive, firmly enterprise territory. And if you're comfortable managing proxy rotation and browser clusters yourself, raw residential proxies plus your own Playwright setup cost less per request at high volume, at the price of considerable engineering time.

Where ScraperAPI holds its ground is the combination of rendering reliability, published per-site credit rates (so costs are predictable rather than mysterious), and structured data endpoints for Amazon, Google Shopping, and Walmart that return parsed JSON and save you writing parsers.

## How to test it before paying anything

The free path is enough to validate whether ScraperAPI can scrape your specific JavaScript-rendered page:

1. Sign up for the free plan, which includes 1,000 API credits and up to 5 concurrent connections.
2. Grab your API key from the dashboard and check your target URL's credit cost with the Domain Cost Estimator.
3. Send a test request with `render=true` and inspect the returned HTML for your data.
4. If you need more volume to evaluate, the 7-day trial adds 5,000 credits with no credit card required.

If your test pages render correctly at a per-page credit cost you can live with, the pricing table above tells you which tier fits your monthly volume. Divide your monthly page count by the per-page credit cost, then match it to the plan. 👉 [Sign up free and run your first rendered request](https://www.scraperapi.com/?fp_ref=coupons)

## FAQ

**Does one ScraperAPI request always cost one credit?**
No. Standard pages cost 1 credit, Amazon 5, Google and Bing 25, LinkedIn 30, and bypassing Cloudflare-style bot protection adds 10. Rendered requests cost roughly 10x a standard request.

**Can it handle React, Vue, or Angular single-page apps?**
Per a long-term production user's assessment, yes: the rendering handles these SPAs consistently, unlike some competitors' bare headless Chrome implementations.

**Is there a free plan?**
Yes. Signing up gets you 1,000 credits per month with up to 5 concurrent connections, no credit card required, plus a 7-day trial with 5,000 credits.

**Can I scrape pages that require logging in?**
Not reliably. Sessions persist for roughly an hour via `session_number`, and there's no built-in persistent login state. Purpose-built tools serve that use case better.

## Which setup should you actually pick

Match your situation to the cheapest workable option:

- Your target embeds `__NEXT_DATA__` or similar JSON, or exposes a callable API → skip rendering entirely. A few lines of parsing code beats any paid service.
- You need real rendering, 1,000–100,000 pages/month, and your time is worth more than $49 → ScraperAPI's Hobby plan, starting with the free 1,000 credits to validate your target. 👉 [Start the free trial and test your target pages today](https://www.scraperapi.com/pricing?fp_ref=coupons)
- You're feeding an AI agent and want Markdown output → Firecrawl's approach fits that better.
- You're pushing 500K+ requests per day → self-managed proxies plus your own browser cluster will cost less, assuming you have the engineering capacity to run it.

One last check before you commit to any paid tier: run three to five of your real target URLs through the Domain Cost Estimator, not just the homepage. Per-page credit costs vary wildly by site, and that single step is the difference between a predictable monthly bill and a credits-within-a-week surprise.
