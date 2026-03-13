# Goal: Bittensor Subnet Analyst (24/7)

## Objective

Build and run a 24/7 system that analyzes Bittensor subnets, classifies them, and produces buy/sell (or stake/unstake) recommendations with suggested price levels or conditions. The system must handle new subnet registrations and deregistrations, keep classifications and recommendations up to date, run a daily evaluation of past recommendations, and auto-improve from that evaluation. Use the detailed blueprint in `research/research.md` for evaluation frameworks (e.g. Crucible Labs axes, Oak Research matrix, five-pillar metrics), subnet categorization, and API usage—treat it as the reference spec for your logic.

## Scope of "buy/sell"

- **Stake allocation**: when to stake or unstake TAO in which subnet (and via which validator if relevant), with target conditions.
- **Alpha tokens**: when to buy or sell subnet Alpha tokens (from pool/AMM data), with target price levels or conditions.
- **TAO (optional)**: if you include market TAO timing, define the source (e.g. spot price API) and the condition format.

Recommendations must include: target subnet (netuid), action, price or condition, and a short rationale.

## Data sources

- **Taostats API**: Subnet list and hyperparams, registration/deregistration events, subnet pool data (price, liquidity, market cap). API key in env (e.g. `TAOSTATS_API_KEY`). Never log or print the key.
- **Data Universe (SN13) API**: X (Twitter) and Reddit insights—on-demand posts by keywords, usernames, timeframes (e.g. Macrocosmos Gravity API). Use for sentiment and mentions around subnets/TAO. API key in env (e.g. `DATA_UNIVERSE_API_KEY` or Macrocosmos key). Respect rate limits (e.g. 100 req/hr for regular keys).
- **Discord**: Scrape or ingest (1) each subnet’s Discord server(s) and (2) the official Bittensor Discord. Use for announcements and community sentiment. Maintain or discover a Discord server list per subnet (e.g. from Taostats, Learn Bittensor, or a config file). If using a bot, Discord bot token in env. Respect Discord ToS and rate limits.
- **Bittensor chain / SDK (optional)**: Metagraph, emission, on-chain state as a complement to Taostats.
- **External (optional)**: TAO spot price (e.g. Coingecko or exchange API) if recommendations include TAO timing.

All API keys and secrets must be read from env only; never output them in logs, files, or messages.

## Required system capabilities

- **Ingest**: Periodically fetch subnets, registrations/deregistrations, pool data (and optionally TAO price). Ingest X and Reddit via Data Universe API. Scrape or ingest Discord (subnet servers + official Bittensor). Handle new and removed netuids.
- **Classify**: Maintain and update a classification of subnets (e.g. by emission tier, risk, use case, liquidity; optionally by social sentiment from X/Reddit/Discord). Store under `context/` or a project directory. Prefer frameworks from `research/research.md` (e.g. Crucible axes, five-pillar matrix) where applicable.
- **Recommend**: Produce buy/sell (or stake/unstake) recommendations with subnet (netuid), action, price/condition, and short rationale. Run on a defined schedule (e.g. every 6h or daily).
- **Log**: Store every recommendation with timestamp and context (e.g. `context/recommendations/` or under `context/runs/`) so daily evaluation can measure outcomes. This is required.
- **Daily evaluation**: Once per day, evaluate past recommendations (e.g. “we said buy subnet X at time T at level Y—what was the outcome?”). Write a short evaluation report (e.g. hit rate, avg return, notable misses). Save to `context/` or summarize in STATE.
- **Auto-improve**: Use the evaluation report and STATE.md to adjust the system (e.g. thresholds, filters, classification or signal logic). Document changes in STATE and apply them in the next steps. Optionally notify the operator of high-conviction recommendations via `python arbos.py send "..."`.

## Success criteria

- Subnets are kept up to date (new/deleted netuids, registration/deregistration events reflected).
- Classifications are maintained and updated as new subnets appear.
- Recommendations are produced on the defined schedule and **always logged** with timestamp and context.
- Daily evaluation runs and writes a short report (or summary in STATE).
- Improvements are decided from evaluation and documented in STATE; the system is modified in subsequent steps.

## Constraints

- Never log, print, or reveal API keys, tokens, or secrets. Use env only.
- Respect API rate limits and Discord ToS.
- Paper-only unless this goal is explicitly updated to allow live trading or staking.
- Prefer read-only use of external APIs; no destructive actions unless explicitly required.
