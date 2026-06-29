# Bank Client Exposure Frameworks

## Overview

Banks that advise, custody, finance, or distribute investments to wealthy clients need a way to see beyond product wrappers and understand the client’s true economic exposure. A client may hold direct equities, mutual funds, PMS mandates, AIFs, private equity funds, structured notes, and credit products at the same time, yet many of those wrappers can point to the same issuer, sector, style factor, or liquidity risk bucket [cite:111][cite:117][cite:119].

This document explains how a bank can move from simple portfolio mapping to a mature internal framework for client exposure management. It covers portfolio mapping, look-through design, governance, internal operating processes, reporting packs, exception handling, and the special challenges posed by private markets and AIF structures [cite:111][cite:113][cite:117].

## Why portfolio mapping matters

Traditional account statements answer the legal question of what a client owns in each account. They do not answer the more important risk question: what is the client economically exposed to after aggregating all wrappers and looking through pooled vehicles [cite:111][cite:117].

This distinction matters because concentration risk often hides inside apparently diversified portfolios. A client can own a large direct bank stock position, a large-cap mutual fund that also owns the same bank, a PMS strategy tilted to financials, and a structured note linked to a banking basket, all while believing the portfolio is spread across many products [cite:111][cite:117][cite:119].

A good bank framework therefore treats wrappers as packaging and exposure as the true analytical unit. The operating objective is to convert all client holdings into a single household-level exposure graph that can be monitored, tested, and reported consistently [cite:113][cite:117].

## The core design principle

The central design principle is simple: separate the legal ownership layer from the economic exposure layer. The legal layer stores the accounts, wrappers, mandates, and securities owned by the client, while the economic layer stores issuer, sector, geography, factor, leverage, and liquidity exposure after look-through logic is applied [cite:111][cite:113].

This separation is necessary because the same position can be represented in different legal forms but contribute to the same economic risk. It is also necessary because some products are fully transparent, some are partially transparent, and some require proxy treatment and confidence scoring instead of exact decomposition [cite:87][cite:93][cite:117].

## Portfolio mapping model

A practical portfolio map starts by linking the client or household to all related entities and accounts. Those entities can include individuals, family trusts, HUFs, companies, family office SPVs, and advisory mandates, each of which may hold investments through different custodians or wrappers [cite:111][cite:123].

The next layer is the wrapper map. Typical wrappers include direct equities, direct debt, mutual funds, ETFs, discretionary PMS, advisory portfolios, AIFs, private equity funds, venture funds, private credit funds, insurance-linked wrappers, and structured products [cite:94][cite:117].

Under each wrapper sit positions and then, where possible, underlying exposures. That means a large-cap mutual fund might resolve to a list of listed issuers, while a private equity fund might resolve to portfolio companies or strategy proxies, and a structured note might resolve to payoff-linked issuer baskets or synthetic factor exposures [cite:87][cite:93][cite:117].

## Data model and schema logic

A robust schema should allow the bank to represent the portfolio at multiple levels of abstraction without losing lineage. A clean minimum model includes client, entity, account, instrument, position, look-through exposure, issuer group, factor map, policy limit, alert, and report snapshot tables [cite:79][cite:111][cite:117].

The most important fields are the fields that support aggregation and auditability. These include `as_of_date`, reporting currency, market value in base currency, exposure percentage, source system, look-through source, confidence score, valuation date, last updated time, manual override reason, and exposure path from client to underlying exposure [cite:79][cite:87][cite:93].

A useful additional field is `economic_exposure_type`, which distinguishes direct ownership, fund look-through, synthetic exposure, estimated proxy, and unfunded commitment. That allows the bank to report not just what the client is exposed to, but how reliably that exposure was derived [cite:87][cite:92][cite:93].

## Householding framework

The first internal framework is householding. Before the bank can monitor exposure correctly, it must identify which individuals, trusts, entities, and mandates should be aggregated into a single economic unit for reporting and control [cite:111][cite:123].

Householding usually depends on beneficial ownership, family linkage, reporting mandate, and product relationship. Without this step, concentration risk becomes artificially diluted across accounts and the bank may underestimate both client risk and internal sales concentration [cite:111][cite:118].

A mature householding framework also includes ownership percentages and report scopes. A family trust owned 40 percent by the client may need a different aggregation treatment from a fully controlled family office SPV, and those rules must be consistent over time [cite:111][cite:123].

## Security master and issuer hierarchy

The second framework is the instrument and issuer master. Every instrument must map to a consistent issuer identity, sector, geography, asset class, and parent group so that the bank can roll up exposures reliably [cite:111][cite:114][cite:119].

This sounds mundane but it is one of the most important control layers. If one system labels an instrument by ticker, another by ISIN, and another by product code, then concentration reports become unreliable because the same economic exposure is fragmented across multiple identifiers [cite:111][cite:119].

For private markets, the issuer hierarchy may need an extra operating-company layer. A PE fund can own a holding company that in turn controls an operating company, and the bank may need to map both to understand effective sector and geography exposure [cite:87][cite:93].

## Look-through framework

The third framework is the look-through engine, which is the heart of exposure monitoring. Its job is to transform wrappers into underlying economic exposure by using available holdings data, payoff decomposition, portfolio-company schedules, or proxy buckets when direct data is unavailable [cite:87][cite:93][cite:117].

A listed equity fund can often be decomposed through actual constituent holdings and weights. A discretionary PMS may require position files from the manager. A structured note may need payoff mapping into underlying reference assets, barrier terms, and leverage assumptions. A PE or AIF fund may need investment schedules, capital account statements, quarterly reports, or proxy mapping by strategy and geography [cite:87][cite:89][cite:92][cite:117].

The key design choice is to avoid treating look-through as binary. Instead, the bank should maintain levels such as full look-through, partial look-through, proxy look-through, and no look-through, with confidence scores and freshness flags attached to each result [cite:87][cite:92][cite:93].

## Handling private equity and AIF exposures

Private markets require a separate overlay because they are structurally different from listed products. Valuations are lagged, disclosures are uneven, liquidity is constrained, and underlying positions may be known only quarterly or with substantial delay [cite:87][cite:89][cite:93].

For buyout funds, look-through is often built from an investment schedule at the portfolio-company level, along with fair value, sector, geography, and leverage data. For venture funds, the data is usually thinner and may only support company-level and stage-level exposure with broad sector tags. For private credit funds, the right look-through unit is often the borrower or loan tranche rather than the fund itself [cite:87][cite:88][cite:92].

For AIFs, the bank should model according to strategy rather than applying one generic template. Credit AIFs should be broken down by loan, maturity, and recovery profile where possible; real estate AIFs should be broken down by property or development project; infrastructure AIFs should be broken down by concession, project, or contracted asset; and mixed strategy AIFs may need hybrid proxy treatment [cite:87][cite:89][cite:94].

Where exact holdings are not available, the bank should use proxy buckets such as “India mid-market buyout,” “late-stage venture healthcare,” or “private credit senior secured.” Those proxies should be marked clearly as estimated exposures and should feed policy limits differently from high-confidence look-through exposures [cite:87][cite:93].

## Concentration and limit framework

Once the exposure graph exists, the bank needs a concentration framework that defines what counts as excessive exposure. This usually includes single issuer limits, related-group limits, sector limits, country limits, factor limits, liquidity concentration limits, and private asset concentration limits [cite:111][cite:114][cite:117][cite:119].

The limit matrix should reflect client type and mandate. A sophisticated family office may tolerate higher single-name concentration than a retirement income mandate, but both should still be governed by explicit rules, documentation, and exception handling [cite:111][cite:117].

A mature framework also distinguishes between hard limits and soft limits. Hard limits should block new trades or force escalation immediately, while soft limits should trigger review, client discussion, or risk acceptance without necessarily prohibiting the position [cite:111][cite:118].

## Factor and correlation framework

Concentration is not only about repeated issuers. Many portfolios become crowded because they repeat the same factor or macro exposure through different instruments, such as quality-growth bias, duration sensitivity, financials beta, small-cap illiquidity, or India domestic cyclical concentration [cite:113][cite:117].

A factor framework therefore complements issuer and sector reporting. It aggregates exposure by style, beta, duration, leverage, currency sensitivity, commodity sensitivity, and sometimes thematic flags so that the bank can identify hidden common drivers of drawdown risk [cite:113][cite:117].

This is especially important where product wrappers differ but underlying behavior converges under stress. A direct equity basket, a concentrated fund, and a convertible structure may appear different in form while reacting similarly to the same macro shock [cite:113][cite:117].

## Liquidity and valuation framework

Liquidity is a separate framework because client exposure is not defined only by mark-to-market size. A position in a daily dealing mutual fund behaves very differently from a locked-up AIF, a PE commitment, or a structured note with limited secondary liquidity [cite:87][cite:113][cite:117].

Banks therefore need to classify each exposure by liquidity bucket, notice period, redemption terms, transferability, expected liquidation horizon, and valuation lag. This helps distinguish portfolios that are concentrated but manageable from portfolios that are concentrated and difficult to exit [cite:87][cite:117].

Valuation governance is closely linked. For private assets, the bank should store both valuation date and report date so users can see when the economics are current and when they are stale. A six-month-old PE mark should not be given the same analytical weight as a listed security price from the prior close [cite:87][cite:92][cite:93].

## Monitoring and control framework

A bank needs a control process that runs both pre-trade and post-trade. Pre-trade controls ask whether a proposed recommendation or trade will breach issuer, sector, factor, liquidity, or product-suitability thresholds. Post-trade controls ask whether changes in market value, fund disclosures, or private valuations have created new concentration after the fact [cite:111][cite:114][cite:118].

A typical monitoring stack includes daily position loads, periodic look-through refreshes, exception rules, stale-data checks, and breach scoring. The purpose is not merely to generate alerts, but to prioritize them so that advisors, risk teams, and product governance teams focus on the breaches that matter most [cite:111][cite:117][cite:118].

Effective alerts often include severity, driver, confidence level, affected client scope, aging, and recommended next action. For example, a high-severity alert could be a client whose aggregate exposure to a single financial group exceeds policy through direct equity, a bank-themed fund, and a structured note, while the underlying look-through data is fully current [cite:117][cite:118][cite:119].

## Internal operating model

A mature internal model usually follows the three-lines-of-defense principle. The front office owns the relationship and the recommendation, risk and compliance independently challenge the resulting exposure and suitability, and internal audit tests whether the framework is operating as designed [cite:111][cite:123].

In practice, the front office should be accountable for client context, portfolio rationale, and recommendation logic. The risk function should own concentration methodology, limit calibration, scenario analysis, and challenge. Compliance should own product governance, disclosure discipline, and conduct concerns. Operations and technology should own data integration, reconciliation, and report generation [cite:111][cite:117][cite:123].

This separation matters because the same desk that sells or recommends a product should not be the final authority on whether the client is appropriately diversified. Independent challenge is one of the strongest protections against sales-driven concentration creep [cite:111][cite:123].

## Exception handling and governance

No exposure framework works without documented exceptions. Some clients intentionally want concentrated positions, founder stock, private business overlap, or high-conviction thematic allocations, and the bank must be able to distinguish informed concentration from uncontrolled drift [cite:111][cite:117].

A strong exception framework records the policy breached, the rationale for temporary or permanent exception, who approved it, the mitigation plan, the review date, and whether the concentration is client-initiated or advisor-initiated. This turns exceptions into controlled governance items rather than silent policy failure [cite:117][cite:118].

Exception governance should also differentiate between data exceptions and exposure exceptions. If the issue is missing look-through data, the action may be to classify the exposure as low-confidence and restrict further sales until the data gap is closed. If the issue is real concentration, the action may be to rebalance, hedge, or obtain client acknowledgement and committee approval [cite:87][cite:92][cite:117].

## Product governance framework

Exposure control should begin before the product is sold. Banks need a product governance framework that defines target market, risk features, liquidity profile, transparency level, and concentration interactions for every product on the shelf [cite:117][cite:123].

This is particularly important for thematic funds, structured notes, concentrated PMS strategies, illiquid AIFs, and products with embedded leverage. A product may be suitable on a standalone basis but unsuitable when layered onto an already crowded household portfolio [cite:117][cite:118].

A product governance view should therefore include a “concentration compatibility” assessment. In simple terms, the bank should ask whether this product meaningfully diversifies the client or merely repackages an existing bet in a different wrapper [cite:117][cite:118].

## Stress testing and scenario analysis

Static concentration reporting is not enough. Banks need scenario analysis to show how portfolios behave under issuer-specific shocks, sector drawdowns, interest-rate changes, correlation spikes, liquidity freezes, or delayed private valuations [cite:113][cite:117].

Scenario analysis is valuable because portfolios that seem diversified in normal periods may converge under stress. A bank-heavy equity basket, a financial-sector fund, subordinated financial credit, and a banking-linked note can all suffer together in a credit stress event even if their wrapper labels differ [cite:113][cite:117].

A strong stress framework should cover both public-market shocks and private-market lag effects. In private markets, the issue is often not just immediate drawdown but stale valuation, delayed recognition of losses, and inability to exit while public proxies are already falling [cite:87][cite:113].

## Reporting stack

A mature bank does not use one universal report. It uses a reporting stack tailored to the audience while preserving one underlying source of truth. The key report families are client-facing reports, advisor diagnostics, risk-management dashboards, product governance reports, and committee packs [cite:79][cite:117].

The client report should be simple, visual, and decision-oriented. It should show top household exposures, where overlap occurs across wrappers, liquidity buckets, major concentration alerts, and recommended actions in plain language [cite:77][cite:79][cite:81].

The advisor report should include more diagnostic depth, such as product overlap matrices, look-through gaps, confidence scores, suitability notes, and “what changed since last review” sections. The risk report should go further into policy breaches, stale data, scenario results, and unresolved exceptions [cite:79][cite:81][cite:117].

## Recommended report template

A practical monthly or quarterly report can be structured as follows:

1. Household overview with total tracked assets, liquid versus illiquid split, and net new inflow or outflow [cite:77][cite:79].
2. Wrapper allocation summary showing direct holdings, mutual funds, PMS, AIFs, private funds, cash, debt, and structured products [cite:77][cite:81].
3. Look-through exposure summary showing top issuers, top sectors, top geographies, and top factors after aggregation [cite:79][cite:81][cite:117].
4. Overlap matrix showing where direct holdings and pooled products duplicate one another [cite:79][cite:81].
5. Liquidity map showing daily, monthly, quarterly, locked, and commitment-based buckets [cite:87][cite:117].
6. Breach and exception register showing policy exceptions, approvals, age, and action owner [cite:117][cite:118].
7. Data quality section showing stale valuations, low-confidence mappings, and unresolved look-through gaps [cite:87][cite:92][cite:93].
8. Recommended actions such as trim, switch, hedge, diversify, monitor, or accept with exception [cite:79][cite:81][cite:117].

## Metrics worth tracking

A bank should not measure only returns and allocation percentages. It should also track concentration score, repeated-issuer count, top-10 exposure share, liquidity concentration, low-confidence look-through share, stale valuation share, unresolved exception count, and product duplication ratio [cite:113][cite:117].

These metrics are powerful because they create management visibility into the quality of the portfolio and the quality of the underlying data. A portfolio can be profitable and still be poorly controlled if concentration, stale marks, and proxy exposures are rising beneath the surface [cite:113][cite:117].

## Major failure patterns this framework is designed to prevent

The framework exists to prevent a recurring set of failure patterns that show up across private funds, alternative investments, and integrated bank platforms. Those patterns include poor fee and expense disclosure, conflicts in private fund operations, stale or incomplete performance reporting, weak transparency into underlying holdings, excessive leverage, and poor control of liquidity and valuation risk [cite:100][cite:101][cite:106].

Another common failure pattern is the inability to see concentration across businesses. A bank may separately view the client as a wealth client, a fund investor, a borrower, or a structured-products buyer, but the client’s real exposure only becomes visible when all those relationships are aggregated into a single graph [cite:111][cite:118][cite:119].

## Operating cadence

A strong framework needs a repeatable rhythm. Daily processes should capture positions, trades, market values, basic breach checks, and urgent alerts. Weekly processes should focus on data quality, stale look-through remediation, and unresolved exceptions. Monthly processes should produce household concentration reports and governance reviews. Quarterly processes should update policy calibration, stress tests, and product shelf reviews [cite:111][cite:117][cite:118].

This cadence is important because exposure drift is continuous. Even with no new trades, market moves, private valuation updates, and new fund disclosures can change concentration materially over time [cite:114][cite:117].

## Practical implementation roadmap

A useful implementation sequence is to start with the listed portfolio and direct holdings, then layer in transparent funds, then discretionary mandates, then structured products, and finally private markets and AIFs. This phased approach reduces complexity while preserving the core design principle from day one [cite:79][cite:87][cite:117].

A sensible roadmap often looks like this:

- Phase 1: Client master, householding, listed holdings, issuer map, basic concentration reporting [cite:111][cite:119].
- Phase 2: Mutual fund and ETF look-through, advisor overlap dashboards, monthly household report [cite:79][cite:81].
- Phase 3: PMS, structured notes, suitability rules, exception workflows, product governance linkages [cite:117][cite:118].
- Phase 4: PE and AIF overlays, proxy logic, confidence scoring, private-market committee review [cite:87][cite:92][cite:93].
- Phase 5: Scenario analytics, enterprise-wide aggregation across lending and trading relationships, management committee dashboards [cite:113][cite:118][cite:119].

## What good looks like

A good framework produces a simple and defensible answer to a difficult question: “What is this client truly exposed to, how certain is that answer, and what should the bank do about it?” The best systems do not pretend that every exposure is perfectly known; instead, they combine transparency where possible, estimation where necessary, and governance everywhere [cite:87][cite:93][cite:117].

The strongest sign of maturity is not a flashy dashboard. It is the ability to trace any reported concentration number back to a source, a method, a date, an owner, and a documented decision path. That is what turns portfolio mapping into a real control framework [cite:111][cite:117][cite:123].
