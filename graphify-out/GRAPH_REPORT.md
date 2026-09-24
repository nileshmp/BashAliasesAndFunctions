# Graph Report - .  (2026-09-24)

## Corpus Check
- Corpus is ~4,138 words - fits in a single context window. You may not need a graph.

## Summary
- 66 nodes · 110 edges · 8 communities (6 shown, 2 thin omitted)
- Extraction: 84% EXTRACTED · 15% INFERRED · 2% AMBIGUOUS · INFERRED: 16 edges (avg confidence: 0.88)
- Token cost: 70,111 input · 0 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Shell Environment & Utilities|Shell Environment & Utilities]]
- [[_COMMUNITY_Service Lifecycle Management|Service Lifecycle Management]]
- [[_COMMUNITY_Session Recording & Resume|Session Recording & Resume]]
- [[_COMMUNITY_Provider & Model Selection|Provider & Model Selection]]
- [[_COMMUNITY_Encrypted Credential Storage|Encrypted Credential Storage]]
- [[_COMMUNITY_Git Branch Deletion|Git Branch Deletion]]
- [[_COMMUNITY_Model Catalog Config|Model Catalog Config]]
- [[_COMMUNITY_Service Manager Entry Point|Service Manager Entry Point]]

## God Nodes (most connected - your core abstractions)
1. `_pi_handle_providers()` - 12 edges
2. `pi()` - 8 edges
3. `_pi_handle_resume()` - 7 edges
4. `_app.is_running` - 7 edges
5. `app.start` - 7 edges
6. `_pi_parse_args()` - 6 edges
7. `_pi_run_new_session()` - 6 edges
8. `APP_MAP` - 6 edges
9. `app.stop` - 6 edges
10. `_pi_model_menu()` - 5 edges

## Surprising Connections (you probably didn't know these)
- `Credential Injection (spec 3.2)` --references--> `_pi_handle_providers()`  [AMBIGUOUS]
  pi_customization_specs.md → /private/tmp/claude-501/-Users-nilesh-work-personal-BashAliasesAndFunctions/720c9523-288b-45a0-9c45-028fd4ee2aec/scratchpad/stage/pi_customization.sh
- `_app.resolve` --semantically_similar_to--> `_pi_model_menu()`  [INFERRED] [semantically similar]
  service_manager.sh → /private/tmp/claude-501/-Users-nilesh-work-personal-BashAliasesAndFunctions/720c9523-288b-45a0-9c45-028fd4ee2aec/scratchpad/stage/pi_customization.sh
- `Provider User Workflows (spec 3.2)` --references--> `_pi_handle_providers()`  [INFERRED]
  pi_customization_specs.md → /private/tmp/claude-501/-Users-nilesh-work-personal-BashAliasesAndFunctions/720c9523-288b-45a0-9c45-028fd4ee2aec/scratchpad/stage/pi_customization.sh
- `Robust Recording and Naming (spec 1.1)` --references--> `_pi_parse_args()`  [INFERRED]
  pi_customization_specs.md → /private/tmp/claude-501/-Users-nilesh-work-personal-BashAliasesAndFunctions/720c9523-288b-45a0-9c45-028fd4ee2aec/scratchpad/stage/pi_customization.sh
- `_app.csv` --semantically_similar_to--> `_pi_parse_args()`  [INFERRED] [semantically similar]
  service_manager.sh → /private/tmp/claude-501/-Users-nilesh-work-personal-BashAliasesAndFunctions/720c9523-288b-45a0-9c45-028fd4ee2aec/scratchpad/stage/pi_customization.sh

## Import Cycles
- None detected.

## Hyperedges (group relationships)
- **Encrypted Provider and Model Selection Flow** — stage_pi_customization_pi, stage_pi_customization_pi_handle_providers, stage_pi_customization_pi_get_password, stage_pi_customization_pi_decrypt_providers, stage_pi_customization_pi_provider_menu, stage_pi_customization_pi_model_menu, stage_pi_models_conf_model_catalog [EXTRACTED 1.00]
- **Port-as-Ground-Truth Process Liveness Tracking** — stage_service_manager_app_csv, stage_service_manager_app_port_pids, stage_service_manager_app_group_pids, stage_service_manager_app_is_running, stage_service_manager_app_running_desc, stage_service_manager_app_start, stage_service_manager_app_stop, stage_service_manager_pid_sidecar_state [EXTRACTED 1.00]
- **Functions Corrected for Zsh 1-Based Indexing** — stage_pi_customization_zsh_one_indexed_array_invariant, stage_pi_customization_pi_provider_menu, stage_pi_customization_pi_model_menu, stage_pi_customization_pi_parse_args, stage_pi_customization_pi_handle_resume [EXTRACTED 1.00]

## Communities (8 total, 2 thin omitted)

### Community 0 - "Shell Environment & Utilities"
Cohesion: 0.12
Nodes (8): nilesh.sh script, AWS_PROFILE, CLICOLOR, ENV, PATH, PROMPT, PYENV_ROOT, PYTHONPATH

### Community 1 - "Service Lifecycle Management"
Cohesion: 0.23
Nodes (16): parse_git_branch(), Zsh Environment Bootstrap, _app.csv, _app.group_pids, _app.is_running, app.list, app.log, APP_MAP (+8 more)

### Community 2 - "Session Recording & Resume"
Cohesion: 0.32
Nodes (12): pi_customization.sh script, pi(), _pi_handle_resume(), _pi_parse_args(), _pi_run_new_session(), _pi_run_resumed_session(), _pi_sanitize_filename(), Helper-Function Refactoring (spec 2) (+4 more)

### Community 3 - "Provider & Model Selection"
Cohesion: 0.31
Nodes (9): loadenv(), _pi_add_new_provider(), _pi_get_password(), _pi_handle_providers(), _pi_model_menu(), _pi_provider_menu(), Credential Injection (spec 3.2), Provider User Workflows (spec 3.2) (+1 more)

### Community 4 - "Encrypted Credential Storage"
Cohesion: 0.83
Nodes (4): In-Memory-Only Credential Handling, _pi_decrypt_providers(), _pi_encrypt_providers(), Encrypted Provider Storage (spec 3.1)

### Community 5 - "Git Branch Deletion"
Cohesion: 0.67
Nodes (3): git.delete.all.branches(), git.delete.local.branch(), git.delete.remote.branch()

## Ambiguous Edges - Review These
- `_pi_handle_providers()` → `Credential Injection (spec 3.2)`  [AMBIGUOUS]
  pi_customization_specs.md · relation: references
- `Per-Provider Model Catalog` → `Credential Injection (spec 3.2)`  [AMBIGUOUS]
  pi_customization_specs.md · relation: conceptually_related_to

## Knowledge Gaps
- **15 isolated node(s):** `nilesh.sh script`, `PATH`, `PYTHONPATH`, `PROMPT`, `CLICOLOR` (+10 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **2 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **What is the exact relationship between `_pi_handle_providers()` and `Credential Injection (spec 3.2)`?**
  _Edge tagged AMBIGUOUS (relation: references) - confidence is low._
- **What is the exact relationship between `Per-Provider Model Catalog` and `Credential Injection (spec 3.2)`?**
  _Edge tagged AMBIGUOUS (relation: conceptually_related_to) - confidence is low._
- **Why does `_pi_handle_providers()` connect `Provider & Model Selection` to `Session Recording & Resume`, `Encrypted Credential Storage`?**
  _High betweenness centrality (0.288) - this node is a cross-community bridge._
- **Why does `loadenv()` connect `Provider & Model Selection` to `Shell Environment & Utilities`?**
  _High betweenness centrality (0.223) - this node is a cross-community bridge._
- **Why does `Zsh Environment Bootstrap` connect `Service Lifecycle Management` to `Session Recording & Resume`?**
  _High betweenness centrality (0.189) - this node is a cross-community bridge._
- **Are the 2 inferred relationships involving `_pi_handle_providers()` (e.g. with `loadenv()` and `Provider User Workflows (spec 3.2)`) actually correct?**
  _`_pi_handle_providers()` has 2 INFERRED edges - model-reasoned connections that need verification._
- **Are the 2 inferred relationships involving `_pi_handle_resume()` (e.g. with `True Session Continuation (spec 1.2)` and `_app.is_running`) actually correct?**
  _`_pi_handle_resume()` has 2 INFERRED edges - model-reasoned connections that need verification._