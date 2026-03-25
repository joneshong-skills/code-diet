# Code Diet Decision Framework — Reference Examples

Source: 2026-03-17 search shared layer refactoring

## Real Examples: Extract vs Keep

### Extracted (correct decisions)

| Pattern | Modules | Decision | Result |
|---------|---------|----------|--------|
| Qdrant availability check + hybrid_search + fallback signal | finance, taskflow, dailyos | `→shared` `search_with_fallback()` | -60 lines boilerplate |
| `EMBEDDING_DIM = 1024` constant | embedding.py, omlx_bridge.py, sparse_tokenizer.py | `→shared` `search_constants.py` | Single source of truth |
| `SERVICE_AVGDL` dict | sparse_tokenizer.py, fallback_search.py | `→shared` `search_constants.py` | Eliminated duplication |
| `f"%{q}%"` raw ILIKE | finance, taskflow, dailyos | `→adopt` `build_ilike_conditions()` | CJK search quality boost |
| `apply_recency_boost()` | memvault scoring, generic search | `→shared` `scoring_stages.py` | Reusable by future modules |
| `cosine_similarity()` | memvault scoring_pipeline | `→shared` `scoring_stages.py` | Pure math, zero coupling |
| Cross-encoder reranking | 6 modules | `→shared` `rerank_utils.py` | Generic type-agnostic |

### Kept in Module (correct decisions)

| Pattern | Module | Decision | Reason |
|---------|--------|----------|--------|
| HyDE query expansion | memvault | `✓ keep` | Uses local LLM, memvault-specific prompt |
| `_DOMAIN_SIGNALS` routing | memvault | `✓ keep` | Signal words are memvault tag vocabulary |
| Weibull time decay | memvault | `✓ keep` | Needs confidence→tier mapping, access_count |
| Trust boost scoring | memvault | `✓ keep` | Imports source_tracker (memvault-specific) |
| MMR diversity | memvault | `✓ keep` | Coupled with memvault's embedding format |
| Noise filter | memvault | `✓ keep` | Memory-specific noise patterns |
| Warm/cold tier search | intelflow | `✓ keep` | ReportArchive table is intelflow-specific |
| Module-specific ILIKE queries | all | `✓ keep` | Each module has different ORM models/columns |
| FinanceSearchResult schema | finance | `✓ keep` | Has amount/wallet_id unique to finance |

### The "Tempting but Wrong" Category

| Pattern | Why it looks extractable | Why extraction would be wrong |
|---------|------------------------|------------------------------|
| SearchPipelineBuilder class | 6 modules do search | Only 3 need the convenience fn; memvault/intelflow have unique pipelines |
| Unified SearchResult type | All search results have score + content | finance has amount, taskflow has full TaskResponse — forcing generics leaks abstraction |
| AbstractSearchService base | All modules have search() | memvault has 7-stage pipeline, finance has 20-line ILIKE — forcing them into same base is a straitjacket |
| Query expander to shared | Could benefit intelflow | intelflow queries are specific enough; HyDE + domain routing are deeply memvault-specific |

## Smell Tests for Over-Abstraction

1. **Does the shared code import from any module?** → Over-abstracted (shared should never depend on modules)
2. **Does the generic type parameter need >2 constraints?** → Probably domain-specific
3. **Would the shared function need >5 optional parameters?** → Probably a sign you're merging different concerns
4. **Is the "shared" code only called by 1 module?** → Move it back
5. **Does extracting it make the module code HARDER to read?** → Not worth it
