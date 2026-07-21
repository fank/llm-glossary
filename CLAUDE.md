# llm-glossary

Astro/Starlight glossary for LLM self-hosting terms. Content lives in `src/content/docs/*.md`; verify with `npx astro build` before committing.

## Entry style

- One entry per term: `**Term** — definition.` in 1–3 terse sentences. Define what the term *is* plus the one practical consequence that matters for self-hosting — no paper summaries, methodology details, or benchmark numbers.
- **Bold every searchable name.** Any term, system, or buzzword a reader might encounter and look up (ATSInfer, PagedAttention, xHC-Flash, …) gets bolded — including names introduced mid-entry, not just the headword.
- Link the primary term to its source (arXiv paper, docs, or GitHub repo): `**[Term](https://arxiv.org/abs/…)**`.
- Cross-reference other glossary pages with relative links: `[KV cache](../inference/)`.
- Match the length and tone of neighboring entries in the file; place new entries next to related ones.
