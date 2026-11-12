# GGL: Graph-Guided Localization for Local SWE Agents on Gemma 4

> **Status: work in progress.** This repository holds the design and starter
> configuration for an entry to the Gemma 4 Developer Agent Competition (Kaggle).
> No benchmark results are reported yet. Adapters are not trained yet.
> The final competition submission is due **December 2, 2026**.

Small open models can write plausible patches but often edit the wrong code.
GGL separates **where to edit** from **what to write**:

1. A read-only **localizer** sub-agent turns the issue into a ranked shortlist of
   functions. It uses embedding search (`search_similar_code`), bounded call/dependency-graph
   expansion (`get_code_neighbors`, `get_code_subgraph`), and lexical evidence extracted from
   the issue text.
2. A **coder** agent reads only the shortlisted spans, reproduces the bug, makes a
   minimal edit and runs nearby tests.
3. A **budget controller** (implemented in the prompt, using `get_status`) splits the
   shared 12-hour budget across tasks and makes sure every task gets a submitted patch.

For post-training, the functions changed by a gold patch serve as free localization
labels, so a LoRA localizer can be trained from issue–patch pairs alone, without
collecting agent trajectories.

The paper (paper track) describes the method and the planned evaluation in full.

## Layout

```
agent.yaml                     # root coder agent
sub_agents/localizer.yaml      # read-only localizer, exposed to the coder as an AgentTool
sub_agents/localizer.md        # localizer procedure (kept beside its config: no ../ includes)
prompts/coder.md               # coder instructions + budget rules
configs/sampling.yaml          # generation parameters
skills/extract_evidence/       # parses issue text: traceback frames, paths, identifiers, errors
skills/find_tests/             # finds the tests nearest to an edited file/symbol
adapters/                      # (planned) loc_lora/, coder_lora/ — PEFT LoRA adapters
```

The structure follows the competition's submission format (`submission.zip` with
`agent.yaml` at the root). **Config field names are a draft** and still need checking
against the competition's `HARNESS_README.md` and the ADK Agent Config spec.

## Skills

Both scripts are plain Python 3 with no dependencies and print JSON.

```bash
python skills/extract_evidence/scripts/extract_evidence.py issue.txt
python skills/find_tests/scripts/find_tests.py path/to/module.py [symbol]
```

## Roadmap

- [ ] Verify config fields against `HARNESS_README.md`; run the A0 baseline (shell-only agent)
- [ ] Localizer prompt + graph tools (ablations A1–A3)
- [ ] Budget controller (A4)
- [ ] Gold-patch label extraction from SWE-Gym / SWE-smith; train `loc_lora`
- [ ] Optional `coder_lora` via SFT on released trajectories
- [ ] Run on E4B and 31B; report resolve rate and localization recall@k

## Author

Swetank Mishra
