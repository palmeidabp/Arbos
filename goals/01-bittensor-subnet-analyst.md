# Goal: Bittensor Subnet Analyst (24/7)

Use the below program to evolve a system S that analyzes Bittensor subnets, classifies them, and produces buy/sell (or stake/unstake) recommendations with suggested price levels or conditions. Be efficient: the agent must not re-fetch data from external APIs on every step—build a data-fetch machine first, then have the agent read from its output.

You are given:
C = { Taostats API (subnets, registrations, pools, Alpha prices), Data Universe SN13 (X, Reddit insights via Macrocosmos Gravity API), Discord (subnet servers + official Bittensor), Bittensor SDK/chain (optional), TAO price API (optional), research/research.md (evaluation frameworks: Crucible axes, Oak matrix, five-pillar metrics) }

Scope of recommendations: stake allocation (when/where to stake TAO), Alpha token buy/sell (from pool data), and optionally TAO market timing. Each recommendation must include target subnet (netuid), action, price/condition, and short rationale. Paper-only unless this goal is later updated to allow live trading or staking. Never log or reveal API keys or secrets; respect rate limits and Discord ToS.

Initial state (build first)
S₀ = 24/7 Bittensor subnet analyst, built in two parts:

(1) Data-fetch machine (build and deploy first)
    - A separate process that fetches all data the analyst needs: subnets, registrations/deregistrations, pool data; X and Reddit via Data Universe; Discord (subnet servers + official Bittensor). Handles new and removed netuids.
    - Runs outside the agent loop (e.g. cron, pm2, or a long-lived stream). Writes results to a fixed location (e.g. context/data/ or context/ingest/) with timestamps so the agent knows what is fresh.
    - Deploy it and keep it running. The agent must not duplicate this work: when the agent wakes up, it reads from these files, not from live APIs. This keeps Chutes/LLM usage low.

(2) Analyst (runs in the agent loop)
    - Reads pre-fetched data from the machine’s output; classifies subnets (e.g. Crucible/Oak from research; emission tier, risk, use case, liquidity, optional social sentiment).
    - Produces buy/sell (stake/Alpha/TAO) recommendations on a schedule (e.g. every 6h or daily) with price/condition and rationale; logs every recommendation with timestamp and context (e.g. context/recommendations/) for evaluation.
    - Runs daily evaluation of past recommendations (outcome vs suggestion; hit rate, avg return, notable misses); auto-improves from evaluation (adjust weights, thresholds, or the data-fetch machine; document in STATE; optionally notify operator via `python arbos.py send "..."`).
    - Stores working state and progress in context/STATE.md and under context/ (e.g. classifications, last run timestamps, next actions); updates STATE.md frequently after each major phase so the next step can resume and pick up where it left off.
    - Before ending a step, set `context/.next_step_delay` to the number of seconds until the next run (e.g. 3600 for 1h, 21600 for 6h) to stay within Chutes rate limits; during build phase immediate iteration is fine.

Run this loop continuously
loop t = 1..∞
    S_t = design_or_modify(S_{t-1})   # implement or update the data-fetch machine and/or the analyst (schedules, logic, weights)
    O_t = run(S_t)                    # run S: read pre-fetched data, classify, recommend, log, run daily eval (do not re-fetch from APIs here)
    P_t = measure(O_t)                # eval: recommendation accuracy, hit rate, PnL vs suggestion, drawdown, regime behavior
    Δ_t = reflect(S_t, P_t)           # find weaknesses (e.g. machine not fresh enough, over-weighting sentiment, under-weighting liquidity)
    S_{t+1} = improve(S_t, Δ_t)       # design a new design; update machine or analyst; document in STATE
end

Then iterate.
