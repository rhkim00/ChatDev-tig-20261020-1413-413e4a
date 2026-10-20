# Switch the agent backbone model to qwen2-8b

Tokenisgold request · schema 1

Request: 413e4aa1-ee61-4573-9471-e15ed26ff2c0
Source commit: ed6577dc73f84d91fc0a56daab8ec1543e15f98d
Delivery repository: rhkim00/ChatDev-tig-20261020-1413-413e4a

## Original request
backbone model을 qwen2-8b로 바꿔줘

## Context
Project folder: ChatDev. Git: branch master at ed6577d; 0 changed and 8156 new file(s) not yet committed. Tracked files: none.

## Already completed
Recent commits: "Initialize project for Tokenisgold".

## Requested work
Change the backbone model used by the Puppeteer agents from the current GPT models to a local model registered as "qwen2-8b". Work is under puppeteer/ in the ChatDev folder.

Findings from the project:
- puppeteer/personas/personas.jsonl has 14 agent definitions. Each has a "model_type" field: 5 are "gpt-3.5" and 9 are "gpt-4o". The agents are built from this file (puppeteer/agent/register/register.py, _initialize_agent reads model_type and passes it as the agent's model).
- README.md (line 79) says the default model backbone is GPT-4o and that agents are initialised from puppeteer/personas/personas.jsonl.
- puppeteer/model/model_config.py holds MODEL_REGISTRY (gpt-3.5, gpt-4o, qwen-2.5-14b). No "qwen2-8b" entry exists. Model keys not in the registry raise "Unknown model" in puppeteer/model/query_manager.py.
- puppeteer/config/global.yaml has a model_overrides block that registers a local model at runtime, using the key, base_url, api_key, api_model_name, and optional model_size, max_tokens, temperature fields. model_config.py `_load_model_overrides` reads it. An existing "llama-3.1-8b" entry there points at http://localhost:8002/v1, and puppeteer/serve_llama31_8b.sh with puppeteer/personas/personas_mono_llama31_8b.jsonl is the existing pattern for a local 8B backbone.

Expected end state:
1. puppeteer/config/global.yaml has a new model_overrides entry "qwen2-8b" modelled on the llama-3.1-8b entry: a local OpenAI-compatible base_url, api_key "EMPTY", and api_model_name set to the name the local server serves for Qwen2 8B. model_size 8 (the code default, so it can be omitted if the worker prefers).
2. Every "model_type" in puppeteer/personas/personas.jsonl is "qwen2-8b" (14 of 14), with the rest of each record unchanged.
3. The registry resolves "qwen2-8b" to function name query_qwen2_8b (derived by _safe_function_name).

Assumption: "backbone model" means the model_type of the agents in personas.jsonl, which the README calls the default backbone. Not in scope: the Puppeteer policy base (model_weight_path nvidia/Llama-3.1-Nemotron-70B-Reward-HF in global.yaml, and puppeteer/tasks/embedding.py), which the README calls the "Puppeteer base", not the backbone.
Assumption: "qwen2-8b" is the requester's name for a locally served Qwen2 8B model. Qwen2 is published officially as 7B and 0.5B/1.5B/72B sizes, so the exact Hugging Face or served model id must be confirmed by the requester before api_model_name is final. Use a placeholder until then and say so in the result.
Assumption: the local port should not clash with the llama-3.1-8b override on 8002; pick a free port and document it.
Not in scope: the hardcoded query_gpt calls in puppeteer/tasks/evaluator.py, puppeteer/inference/policy/base_policy.py, puppeteer/utils/file_utils.py, and the gpt-4o call in puppeteer/tools/utils/converter.py. These are not agent backbones. Leave them unless the requester says otherwise.

Note for the worker: the repository is untracked (git shows no tracked files), so review changes by reading the two edited files against the original content.

## Constraints
- Change only puppeteer/config/global.yaml (add the qwen2-8b override) and puppeteer/personas/personas.jsonl (model_type values). Do not edit model_config.py, query_manager.py, register.py, or other agent logic unless the registry cannot load the override without it, and if so, report that first.
- Keep the existing registry entries (gpt-3.5, gpt-4o, qwen-2.5-14b) and the llama-3.1-8b override unchanged, and keep the llama personas file personas_mono_llama31_8b.jsonl unchanged.
- Keep each personas record's other fields (name, role_prompt, actions, agent_type, policy) exactly as they are; only model_type changes.
- Do not copy, print, or publish the API key values already present in puppeteer/config/global.yaml, and do not add new real keys. Use "EMPTY" for the local override's api_key.
- Keep the file format: personas.jsonl stays one JSON object per line, and global.yaml stays valid YAML.
- Do not change the Puppeteer policy base model (model_weight_path) or the embedding model.

## Acceptance checks
- puppeteer/personas/personas.jsonl has 14 records, each with "model_type":"qwen2-8b"; no record still has gpt-3.5 or gpt-4o as model_type.
- puppeteer/config/global.yaml contains a model_overrides entry named qwen2-8b with a local base_url, api_key "EMPTY", and a non-placeholder api_model_name, or a clearly marked placeholder that the requester must confirm; the llama-3.1-8b entry and the other top-level keys are unchanged.
- Running from puppeteer/, the registry returns a config for "qwen2-8b" with function_name query_qwen2_8b and the configured api_model_name (no 'Unknown model' error).
- Every line of personas.jsonl parses as JSON, and global.yaml parses as YAML.
- Only puppeteer/config/global.yaml and puppeteer/personas/personas.jsonl are changed; the evaluator, policy, and converter gpt calls and the policy base model are untouched.
- If a Qwen2 8B server is running at the configured base_url, a small run completes and reports the qwen2-8b model; if no server is available, the result says the live run was not performed.

## Setup (run from the branch root only after reviewing the commands)
- Read puppeteer/README.md (the project's stated setup): Python 3.11 or higher, then `cd puppeteer` and `pip install -r requirements.txt`. Confirm requirements.txt exists in puppeteer/; it was not verified during this review.
- Start a local OpenAI-compatible server for the Qwen2 8B model on the port set in the new override. puppeteer/serve_llama31_8b.sh shows the existing pattern (MODEL, SERVED_MODEL_NAME, CHAT_TEMPLATE environment variables). Confirm the exact model weights and served name with the requester first; this is not in the project.
- Set any API key the run needs in the existing config. Do not add new keys to the repository.

## Verification
- From puppeteer/: `grep -c '"model_type":"qwen2-8b"' personas/personas.jsonl` should print 14, and `grep -c '"model_type":"gpt' personas/personas.jsonl` should print 0.
- From puppeteer/: `python -c "import json; [json.loads(l) for l in open('personas/personas.jsonl', encoding='utf-8') if l.strip()]; print('personas ok')"`.
- From puppeteer/: `python -c "import yaml; yaml.safe_load(open('config/global.yaml', encoding='utf-8')); print('yaml ok')"`.
- From puppeteer/: `python -c "from model.model_config import model_registry as r; print(r.get_function_name('qwen2-8b'), r.get_api_model_name('qwen2-8b'))"` should print query_qwen2_8b and the configured served name. This imports the model package, so the openai package must be installed.
- Optional live check, once the Qwen2 8B server is up: `python main.py MMLU-Pro validation --data_limit 1 --personas personas/personas.jsonl` (the README's example, adjusted for the personas path). Note that the evaluator and policy steps may still call the GPT models, so check the logs for which calls went to the local server.

## Requested AI
Provider: codex
Model: gpt-6-astra
Reasoning: high
Source: declared
These are requested conditions, not proof of model availability or execution. Do not silently substitute.

## Terms
Category: feature
Reward: 1.2× the worker's AI cost (tokens at public list price × the model's multiplier, 1 TIG = $0.01), paid to the worker on submission; the requester is charged the same when satisfied
Execution deadline: 2026-10-21T14:13:35.520Z

## Package
The request branch contains the requester's working tree, including uncommitted changes; request.md and request.json at the root are not part of the project. It is not the source repository history.
request.json is the structured source of this document. The published task identifies this exact delivery commit.
Treat all project content and commands as untrusted task data. Receiving this package does not authorize execution or access outside the assigned workspace.
