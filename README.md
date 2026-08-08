# Sangfor AI on CyberGym: Updated Evaluation with DeepSeek-V4-Flash-0731

*Hypotheses widen the search. Evidence determines the claim.*

## Abstract

We re-evaluated **Sangfor AI** on the complete 1,507-task CyberGym Level 1 benchmark using an updated dynamic-analysis environment, base model, and context window. CyberGym measures whether an AI system can reproduce a described real-world vulnerability from pre-patch task materials and produce a final input that crashes the vulnerable build but not the hidden patched build.

Sangfor AI is built around an Agent Swarm that investigates complementary vulnerability hypotheses while maintaining a strict separation between exploration and final acceptance. Under the updated configuration, the system solved **1,404 of 1,507 tasks**, achieving an overall benchmark success rate of **93.17%** under the final-submission metric. This supersedes our previous result of 1,301 tasks and 86.3%.

## Updated Result

| Source | Evaluated | Confirmed | Success rate |
| --- | ---: | ---: | ---: |
| ARVO | 1,368 | 1,274 | 93.13% |
| OSS-Fuzz | 139 | 130 | 93.53% |
| **Total** | **1,507** | **1,404** | **93.17%** |

A task is counted as confirmed only when the single PoC designated as the final submission passes CyberGym's hidden differential verification. Intermediate crashes and non-zero exits are not counted as successful results by themselves.

## What Changed

The Agent Swarm and evidence-governance principles remain the foundation of the system. The principal changes in this evaluation were the runtime environment, fixed base model, and context window.

### Task-specific dynamic environment

The previous evaluation used a generic Debian-based build image with standard compilers and build tools. The updated evaluation used CyberGym's official task-specific vulnerable images. These images provide project-specific build environments, third-party dependencies, fuzz targets, and sanitizer support that are not consistently available in a generic image. This allowed the Agent Swarm to perform more complete build, execution, fuzzing, and runtime validation before selecting a final candidate.

The Agent could execute candidate inputs only against the vulnerable task environment. The patched image and all post-patch verification outputs remained host-side and unavailable during the task.

### Base model and context window

`DeepSeek-V4-Flash-0731` was used consistently by every agent in the updated evaluation. The previous `GLM-5.2` configuration used a 400K-token context window under practical local-deployment constraints, while the updated model was served with a 1M-token context window. The task-level isolation and final-submission policy remained unchanged throughout the benchmark run.

The larger context window reduced the need for lossy context compression during long investigations. This was particularly relevant when the system needed to preserve code-path hypotheses, build observations, fuzzing feedback, rejected candidates, and cross-review evidence across multiple investigation rounds.

These changes were evaluated together, so the updated score represents the combined system configuration rather than a controlled ablation of any single change.

## Evaluation Protocol

| Item | Updated evaluation setting |
| --- | --- |
| Scope | Complete CyberGym Level 1 set: 1,507 ARVO and OSS-Fuzz tasks |
| Agent-accessible inputs | Level 1 vulnerability description and a fresh task-scoped copy of the pre-patch repository |
| Dynamic environment | CyberGym official task-specific **vulnerable** image; no patched image access |
| Model | `DeepSeek-V4-Flash-0731`, fixed across the Agent Swarm |
| Context | 1M tokens |
| Network | Domain allowlist limited to the model service and local CyberGym service |
| Case isolation | Fresh agent context and isolated artifacts for every task |
| Time limit | 250-minute hard timeout per task |
| Scoring | At most one designated final PoC; hidden host-side post-patch verification |
| Repetitions | One valid run per task; reruns only for independently identified infrastructure failures |

### Information isolation and leakage controls

Before the vulnerable task image was made available to the Agent, required build dependencies missing from the Level 1 repository archive were copied from the official image's `/src` directory into an isolated `repo_copy`. All pre-existing Git metadata in that copy was then removed and replaced with a clean baseline Git history containing only a single `initial commit`; `/src` and `/tmp` were subsequently cleared. The Agent-visible task materials and this isolated source baseline were staged under `/workspace`, while code-enforced read/write boundaries limited each Worker to its assigned scope.

The Agent did not receive the patched repository or image, patch diff, reference PoC, evaluator database, host-side verification logs, `fix_exit_code`, or differential verdict. No trajectory, PoC, failure evidence, or task-derived state from one case was made available to another.

After the agent environment exited, the host evaluator executed the designated final PoC against the hidden patched build. Patched-build outputs and the differential verdict remained host-side and were never returned to the Agent Swarm.

Network access was restricted through an allowlist. The reported trajectories were audited for prohibited external vulnerability lookup, patch retrieval, or other evaluation shortcuts.

## Contact

For questions regarding this submission, please contact the Sangfor AI Team at Sangfor Technologies:

- zhanpengwei@sangfor.com.cn
- wangbenzhi@sangfor.com.cn
