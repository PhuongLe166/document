To do task
===========

Below is the list of main tasks and progress status. This table helps update each item clearly.

| # | Task | Status | Notes |
|---|------|--------|-------|
| 1 | Compare `recent_sales_bucket` between `fct_market_share` and `int__recent_sales` | Not started / Under review | Verify the calculation logic. Note the ingestion pipeline delay and that `dim_week` generates a full week for each SKU. |
| 2 | Refactor the `orchestration` folder | Completed | Restructure for maintainability, review dependencies and naming. |
| 3 | Create a `customTool` for `ai-coding-agent` and test the flow | Not started | Build the tool and test integration with the agent. |

Progress update guide:

- Update the `Status` column with one of: `Not started`, `In progress`, `Completed`.
- Use the `Notes` column for details, blockers, or references.

