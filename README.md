# AI Fiscal Analysis System

Natural-language analysis of Brazilian electronic invoices (NF-e CSV exports), built as the capstone project for the I2A2 *Intelligent Agents with Generative AI Networks* course (Canada, 2025).

You load a CSV or a ZIP containing the header and item exports, ask a question in Portuguese, and the agent computes the answer with pandas over the full dataset instead of guessing from whatever rows happened to fit in the prompt.

## The problem this solves

The default way to put an LLM in front of a dataframe is to paste a sample of rows into the prompt. It works in a demo and fails in use: the model answers correctly about the thirty rows it was shown and confidently wrong about the forty thousand it was not. Totals are the first thing to break, and a wrong total in a fiscal report is worse than no report.

## How it works

1. **Column normalization.** NF-e exports arrive with inconsistent headers across sources, so column names are lowercased, accent-stripped and separator-normalized before anything else touches them. Monetary and quantity columns are coerced to numeric and dates to datetime with `errors="coerce"`, so bad cells become null instead of killing the load.
2. **A custom ZeroShot agent instead of a prompt dump.** The agent gets a `PythonAstREPLTool` holding the complete `cabecalho_df` and `itens_df` in its locals. The prompt carries the schema, the record counts, the period covered and a bounded sample, and the instruction is explicit: compute over the full dataframes, cite exact values, explain the method used.
3. **Streamlit front end** for loading files, browsing an overview and asking questions, with a few aggregate charts (value by issuing state, top products).

## Notes from building it

The agent kept returning its reasoning without ever emitting a final answer, which is the classic ZeroShot parsing failure. The fix was in the prompt `SUFFIX`, forcing the `Final Answer:` token into position rather than hoping the model would produce it.

Giving an agent a Python REPL over your dataframes is what makes the answers accurate, and it is also the reason this is a single-user local tool. The model writes code that executes in the process. That is acceptable when you are the only user, running it against your own files, and it is not something to expose to anyone else without a sandbox.

## Stack

Python · pandas · LangChain (`ZeroShotAgent`, `PythonAstREPLTool`) · LLaMA via Groq · Streamlit

## Running it

```bash
uv sync
uv run streamlit run app.py
```

The Groq API key is entered in the sidebar at runtime.

## Scope and limitations

- Inference runs through the **Groq API**, so invoice contents leave the machine. Fine for the public course dataset, not suitable for a client's real fiscal data without moving to local inference first.
- Course scope: one analysis module, no test suite, no auth, no multi-user state.
- NF-e **CSV exports only**. No XML parsing and no SEFAZ integration.
- Fiscal code semantics (CFOP, NCM) come from the model, not from a lookup table, so treat any interpretation of a code as a starting point rather than an answer.

A sample generated report is included as `Relatorio_Analise_Fiscal_IA.pdf`.

## Licence

MIT. See [LICENSE](LICENSE).
