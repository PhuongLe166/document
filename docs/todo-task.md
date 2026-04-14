To do task
===========

Below is the list of main tasks and progress status. This table helps update each item clearly.

| # | Task | Status | Notes |
|---|------|--------|-------|
| 1 | Compare `recent_sales_bucket` between `fct_market_share` and `int__recent_sales` | Not started / Under review | Verify the calculation logic. Note the ingestion pipeline delay and that `dim_week` generates a full week for each SKU. |
| 2 | Create a `customTool` for `ai-coding-agent` and test the flow | Not started | Build the tool and test integration with the agent. |
| 3 | Extract `generate_dbt_docs` from the Dify workflow and port it into a Claude agent script | Not started | Pull the existing `generate_dbt_docs` logic out of the Dify workflow, then rebuild it as a Claude agent script. |


Progress update guide:

- Update the `Status` column with one of: `Not started`, `In progress`, `Completed`.
- Use the `Notes` column for details, blockers, or references.

---

Task details
------------

### 1. Compare `recent_sales_bucket` between `fct_market_share` and `int__recent_sales`

Steps:
1. Locate the `recent_sales_bucket` definition in both `fct_market_share` and `int__recent_sales` models.
2. Diff the SQL logic (window functions, joins, filters, bucket thresholds) and record any divergence.
3. Account for the ingestion pipeline delay — identify the cut-off timestamp each model relies on.
4. Inspect how `dim_week` expands a full week per SKU and confirm both models handle the expansion consistently.
5. Run both models on the same snapshot and compare `recent_sales_bucket` values per SKU / week.
6. Document discrepancies and propose a reconciliation (align logic, add tests, or flag as expected).

### 2. Create a `customTool` for `ai-coding-agent` and test the flow

Steps:
1. Define the tool contract: name, description, input schema, output schema, and error cases.
2. Scaffold the `customTool` inside the `ai-coding-agent` project following the existing tool structure.
3. Implement the tool logic and wire it into the agent's tool registry.
4. Add unit tests covering happy path, invalid input, and failure modes.
5. Run an end-to-end agent session that invokes the tool and verify the expected behavior.
6. Update the agent documentation / README with usage examples.

### 3. Extract `generate_dbt_docs` from the Dify workflow and port it into a Claude agent script

Steps:
1. Open the Dify workflow and export the `generate_dbt_docs` node graph (prompts, tools, variables, branching).
2. Map each Dify node to its Claude equivalent: system prompt, tool calls, control flow.
3. Scaffold a Claude agent script (Python / TypeScript SDK) with the same inputs and outputs as the Dify workflow.
4. Port prompts verbatim first, then refactor for Claude-native patterns (tool use, prompt caching).
5. Reproduce the dbt docs generation locally against a sample dbt project and diff the output against the Dify version.
6. Add logging, retries, and error handling for long-running generations.
7. Document how to run the script and hand it off / integrate with the existing pipeline.
