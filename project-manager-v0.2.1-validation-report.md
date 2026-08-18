# Project Manager v0.2.1 Validation Report

Date: 2026-08-19
Target: `project-manager-v0.2.1-copilot-ja`

## Result

**PASS with runtime limitations.**

Standalone Backendの主要End-to-End workflowと、Agent/Skill/State/Backend contractの静的整合性はPASSした。
OpenSpec Backendは現行公式仕様とのmappingを確認したが、この実行環境ではOpenSpec CLIの一時取得がtimeoutしたため、OpenSpec CLIそのものを用いたE2E runtime testは未実施。
実際のVS Code GitHub CopilotによるSkill discovery、Auto model routing、AI credits消費もこの環境では再現できないため、最終的に短いユーザー環境smoke testが必要。

## Static validation

PASS:

- `project-manager.agent.md` frontmatter: `name`, `description`, `argument-hint`
- 7 Skillsが存在し、directory nameとfrontmatter `name`が一致
- Project Stateは `READY / CHANGE_ACTIVE / BLOCKED` のみ
- 旧State `ACCEPTANCE_PENDING`, `IMPLEMENTING` の混入なし
- OpenSpec / Standalone両Backendが共通logical operationsを提供
- `AUTO_PREFERRED` routing policy templateあり
- Standalone archive pathが明示されている
- OpenSpec archive preview/commit guidanceが分離されている

Backend logical operations checked:

- `get_current_specs`
- `create_change`
- `get_change_contract`
- `update_change_contract`
- `get_delta_specs`
- `update_delta_specs`
- `get_design`
- `update_design`
- `get_tasks`
- `update_tasks`
- `get_verification_state`
- `record_verification`
- `preview_finalization`
- `commit_finalization`

## E2E Scenario 1 — Normal LOW-centered Change

Fixture: Counter APIへ`decrement()`を追加。

Flow tested:

`READY → Change activation → LOW planning → implementation → pytest → tasks verified → merge preview → Acceptance metadata → Standalone finalization → archive → ROADMAP update → READY`

Result:

- baseline: 1 test PASS
- after implementation: 2 tests PASS
- accepted Current Spec updated
- active Change removed
- archived to `changes/archive/2026-08-19-C-001-add-decrement/`
- final State: `READY`

**PASS**

## E2E Scenario 2 — External HIGH Initial Design

Fixture: Counter APIへ`reset()`を追加。

Critical assertion:

External handoff前には、LOWによるsubstantive Delta/Design/Tasks draftを作成しない。
External Design Packageを模擬importした後にRepository Fit → LOW implementationへ移る。

Result:

- External response前: DesignはTBD、Tasksは空のplaceholderのみ
- simulated External designをimport
- LOW implementationへde-escalate
- 2 tests PASS
- accepted spec更新
- archived to `changes/archive/2026-08-19-C-002-add-reset/`
- final State: `READY`

**PASS**

## E2E Scenario 3 — Structural Replan + Finalization Recovery

Fixture: Counterへupper limitを追加。

Initial designはwrapper方式だったが、repository factとして既存callerが`Counter`を直接instantiateするため構造的に不適切、という状況を模擬。

Flow tested:

- completed task historyを保持
- existing Design structureを捨ててStructural Replan
- repair/superseding taskを追加
- revised implementation + tests
- Human Acceptanceを模擬
- Backend finalization成功後にProject-level update failureを意図的に発生
- `CHANGE_ACTIVE + FINALIZATION + Acceptance: ACCEPTED`を保持
- Human Acceptanceを再要求せずidempotent resume
- final State `READY`

Result:

- 2 tests PASS
- completed `T-001` history preserved
- `Superseded by: T-003` preserved
- archive exists after backend finalization
- recovery時の再Acceptanceなし
- final State: `READY`

**PASS**

## Issues found in v0.2.0 and fixed in v0.2.1

### 1. LOW/HIGH cannot be treated as guaranteed automatic model switching

Project ManagerのLOW/HIGHを**reasoning class**へ修正。

New policy:

- default: `AUTO_PREFERRED`
- Copilot Autoが利用可能ならtask intentを与え、actual model selectionはAutoへ委ねる
- fixed model運用 (`MANUAL_FIXED`) でHIGHが必要なら、model pickerでの切替を`COPILOT_MODEL_SWITCH` Operational Handoffとして扱う
- model切替を確認できないのに「HIGHを実行した」と記録しない

Reason:

VS Codeのsubagentはmodel指定可能だが、requested subagent modelはmain modelのcost tierを超えられないため、安価なparent modelから高価なmodelへの自動escalationをsubagentだけで保証できない。

### 2. Standalone archive semantics were ambiguous

Explicit storage rule added:

- active: `changes/<change-id>-<slug>/`
- accepted archive: `changes/archive/YYYY-MM-DD-<change-id>-<slug>/`

Finalization retry時は既存archiveを上書きせず、同じfinalization resultか確認してresumeする。

### 3. OpenSpec finalization guidance clarified

`preview_finalization`と`commit_finalization`を明確に分離。

- preview: read-only comparison/context; Current Specsを書き換えない
- commit: Human Acceptance後にspec merge + archive
- current CLI capabilityが利用可能なら、重複した人間確認を避けるため `openspec archive <change-name> --yes` を優先
- version/profile差を考慮し、実行前にcurrent CLI/generated integrationを確認

## OpenSpec compatibility review

Current official OpenSpec documentation was checked against the adapter design:

- main specs live under `openspec/specs/`
- active change artifacts live under `openspec/changes/<name>/`
- a change can contain proposal, delta specs, design, and tasks
- OPSX permits iterative editing rather than rigid phase lock
- GitHub Copilot has tool-specific command spelling and generated integration
- archive/sync behavior may vary across profiles/integration surfaces, so `backend-openspec` intentionally delegates to the currently installed OpenSpec integration instead of hardcoding a full workflow

### Runtime limitation

An isolated `npx @fission-ai/openspec@latest --version` attempt was made, but package acquisition timed out in this container. Therefore **OpenSpec CLI E2E is not claimed as tested**.

## Remaining user-environment smoke test

The following items can only be confirmed in actual VS Code + GitHub Copilot:

1. `Project Manager` custom agent is discovered from `.github/agents/`.
2. Skills under `.github/skills/` are selected as intended from their descriptions.
3. `次に進んで` performs multiple task-level actions without unnecessary stopping.
4. `AUTO_PREFERRED` behaves acceptably with the user's available Copilot models/policies.
5. If `MANUAL_FIXED` is used, HIGH recommendation correctly surfaces `COPILOT_MODEL_SWITCH` rather than pretending automatic escalation.
6. OpenSpec-installed environment and Standalone environment both select the correct backend.
7. Actual AI credit consumption is acceptable.

This should be a short smoke test, not a repeat of the full E2E validation.

## Validation conclusion

v0.2.1 is suitable as the next **user-test baseline**.

The Standalone workflow has passed the core lifecycle, External-first design path, Structural Replan, historical task preservation, Acceptance/finalization recovery, and state invariants. The remaining uncertainty is primarily GitHub Copilot/OpenSpec runtime behavior outside this container, not the repository-local lifecycle model itself.
