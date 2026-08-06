**You:**

コンテキストエンジニアリングとは何なのでしょうか？別スレッドで教えてもらった、プロジェクトファイルの開発状況などを記録した共有ファイルを保存する方法に近いのでしょうか？

* * *

**ChatGPT:**

はい、かなり近いです。  
ただし、**「プロジェクト状況を記録した共有ファイルを置くこと」自体がコンテキストエンジニアリングなのではなく、そのような情報をAIに適切な形・量・タイミングで渡す仕組み全体**を指します。

## コンテキストエンジニアリングとは

LLMが回答や作業を行う際に参照する情報、つまりコンテキストを設計・管理することです。

たとえば、GitHub Copilotに開発を依頼する場合、AIが必要とする情報には次のようなものがあります。

* このプロジェクトの目的
* 現在の仕様
* 実装済みの機能
* 未実装の機能
* 技術的な決定事項
* コーディング規約
* 過去に発生した問題
* 今回編集してよいファイル
* 今回の作業で達成すべきこと

これらを、毎回すべて会話で説明するのではなく、**AIが安定して参照できる場所に整理し、必要な情報だけを渡す**のがコンテキストエンジニアリングです。

## 共有ファイルとの関係

以前お話しした、プロジェクトフォルダ内に現在の状況をまとめたファイルを置く方法は、典型的なコンテキストエンジニアリングです。

たとえば次のような構成です。

```
project/
├─ README.md
├─ .github/
│  ├─ copilot-instructions.md
│  ├─ prompts/
│  └─ agents/
├─ docs/
│  ├─ project-context.md
│  ├─ feature-brief.md
│  ├─ implementation-plan.md
│  ├─ technical-decisions.md
│  ├─ current-status.md
│  └─ known-issues.md
└─ src/
```

特に、以前議論していた「プロジェクトの現時点の仕様や実装段階をまとめた共有ファイル」は、次の役割を持ちます。

```
current-status.md
```

```Markdown
# Current Status

## Current goal
COMMON不整合修正フェーズを完了し、
次にCALL引数整合性の解析機能を実装する。

## Implemented
- program unit単位のfparser2解析
- COMMONブロック収集
- COMMON不整合候補の出力
- 合成fixtureによるテスト

## In progress
- CALL側とcallee側の引数対応付け
- 暗黙型変数の型推定

## Not yet implemented
- CALL引数修正の自動適用
- Absoftとの数値回帰比較
- Python移行フェーズ

## Important constraints
- 実際の社内Fortranコードは利用不可
- 合成fixtureのみで検証する
- 自動修正前に承認候補を提示する
```

Copilot Chatの会話を切り替えた場合でも、最初にこのファイルを参照させれば、比較的少ない説明で作業を再開できます。

## プロンプトエンジニアリングとの違い

両者は似ていますが、対象範囲が違います。

### プロンプトエンジニアリング

AIへの指示文を工夫することです。

```
src/analyzer/call_checker.pyを調査し、
CALL引数の型不一致検出を実装してください。
既存テストを壊さず、修正前に実装方針を提示してください。
```

### コンテキストエンジニアリング

その指示を正しく実行するために、AIへ何を参照させるかを設計することです。

```
参照:
- docs/current-status.md
- docs/technical-decisions.md
- docs/call-analysis-spec.md
- tests/fixtures/call_mismatch/
- src/analyzer/call_checker.py
```

つまり、

> プロンプトエンジニアリングは「何をしてほしいか」の設計  
> コンテキストエンジニアリングは「何を知った状態で作業してもらうか」の設計

と考えると分かりやすいです。

## 単に大量の情報を渡せばよいわけではない

コンテキストエンジニアリングで重要なのは、情報量を増やすことではありません。

むしろ、情報を渡しすぎると次の問題が起きます。

* 古い仕様と新しい仕様が混ざる
* 関係のないコードまで調査する
* 会話や入力トークンが増える
* Copilotのクレジット消費が増える可能性がある
* 重要な制約が埋もれる
* AIがどの文書を優先すべきか分からなくなる

したがって、プロジェクト全体を1つの巨大なファイルにまとめるよりも、役割別に分ける方がよいです。

| ファイル | 主な役割 |
| --- | --- |
| `project-context.md` | プロジェクト全体の目的・背景 |
| `current-status.md` | 現在地点、次の作業 |
| `feature-brief.md` | 対象機能の要求仕様 |
| `implementation-plan.md` | 実装手順 |
| `technical-decisions.md` | 技術選定と判断理由 |
| `known-issues.md` | 未解決問題、注意事項 |
| `handoff.md` | 会話や担当を切り替える際の引き継ぎ |

## あなたのプロジェクトに当てはめると

Fortranリファクタリング・Python化ツールでは、次のような流れがコンテキストエンジニアリングになります。

```
人間
  ↓ 仕様・判断を記録
docs/project-context.md
docs/current-status.md
docs/technical-decisions.md
  ↓ 必要なものだけ参照
GitHub Copilot
  ↓ 実装・テスト・レポート
ソースコード / テスト / 実装結果
  ↓ 状況を更新
docs/current-status.md
```

さらに社内AIと連携する場合は、

```
プロジェクト内部の共有コンテキスト
        ↓
社外秘情報を除去・抽象化
        ↓
社内AIへ渡す相談用コンテキスト
        ↓
社内AIの回答
        ↓
Copilotが参照できる文書へ整理
```

という流れになります。

この場合、以下は分ける方が安全です。

```
docs/internal/project-context.md
docs/exchange/ai-consultation-context.md
docs/exchange/ai-consultation-response.md
```

`project-context.md`をそのまま外部や別AIへ渡すのではなく、**相談用に再編集した文書を生成する**という考え方です。

## コンテキストエンジニアリングに含まれるもの

共有Markdownファイル以外にも、次のようなものが含まれます。

* `.github/copilot-instructions.md`
* カスタムエージェント
* `.prompt.md`テンプレート
* 対象ファイルの明示
* 会話終了時の引き継ぎ文書
* コード検索やRAG
* 実行ログやテスト結果の要約
* 過去の技術判断の記録
* AIに読ませる優先順位
* 長い文書の要約・分割
* 古い情報の削除や更新
* 機密情報のフィルタリング

したがって、以前検討していた共有ファイル方式は、

> **ファイルベースのコンテキストエンジニアリング**

と呼べるものです。

特にGitHub Copilotのように、複数のチャットやAgent、Planモードをまたいで開発する場合には、会話履歴だけに依存せず、**プロジェクトフォルダ側を正本にする**ことが重要です。AIの会話は一時的な作業場所、Markdownファイルは継続的なプロジェクト記憶、と役割を分けると安定します。

* * *

**You:**

docs/current-status.mdやdocs/technical-decisions.mdなどのファイル名を指定してくださっていますが、このファイル名やファイル内容は業界の標準的なファイル名称なのでしょうか？なるべく社内の様々なプロジェクトで汎用的に使えるプロンプトエンジニアリングやコンテキストエンジニアリングにしたいと思っての質問になります。それとも、プロジェクトごとにこのあたりもカスタマイズしたほうがよいのでしょうか？

* * *

**ChatGPT:**

結論として、**`docs/current-status.md`や`docs/technical-decisions.md`は業界で固定された標準ファイル名ではありません**。私が前回示したのは、内容が分かりやすい実務的な命名例です。

一方で、GitHub Copilotなどのツールが正式に認識するファイルには、決められた名前や配置があります。したがって、社内で汎用化するなら、

> **ツール指定の名前は厳密に合わせ、その他は社内標準として統一し、内容だけプロジェクトごとにカスタマイズする**

という設計がよいです。

## 1. ファイル名には3段階あります

### A. ツールが名前を規定しているもの

これは名称を変更しない方がよいものです。

```
.github/
├── copilot-instructions.md
├── instructions/
│   └── fortran.instructions.md
├── prompts/
│   └── implement-feature.prompt.md
└── agents/
    └── fortran-modernizer.md
```

GitHub Copilotでは、リポジトリ全体の指示に`.github/copilot-instructions.md`、対象パス別の指示に`.github/instructions/**/*.instructions.md`、再利用プロンプトに`.prompt.md`形式が使われます。これらは単なる慣習ではなく、Copilotが認識するための正式な形式です。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+3![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+3![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+3](https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot?utm_source=chatgpt.com)

`README.md`や`CONTRIBUTING.md`もGitHubが特別に認識・表示する代表的なファイルです。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes?ref=miguelarios.com&utm_source=chatgpt.com)

### B. 業界で広く知られているが、完全には統一されていないもの

代表例がADR、Architecture Decision Recordです。

```
docs/
└── decisions/
    ├── README.md
    ├── 0001-use-fparser2.md
    ├── 0002-require-human-approval.md
    └── 0003-use-synthetic-fixtures.md
```

ADRは、重要な技術判断について「背景、選択肢、決定、理由、影響」を一件ずつ記録する方法です。ADRという概念や基本構造は広く共有されていますが、保存フォルダ名は次のようにプロジェクトごとに異なります。

```
docs/adr/
docs/adrs/
docs/decisions/
docs/architecture/decisions/
```

ファイル名も、次のような複数の形式があります。

```
0001-use-postgresql.md
ADR-0001-use-postgresql.md
0001-use-postgresql-for-persistence.md
```

つまり、**ADRという方法は準標準的ですが、ディレクトリ名とファイル名までは標準化されていません**。一般的なADRでは、少なくともタイトル、状態、背景、決定、結果・影響を記録します。[![](https://www.google.com/s2/favicons?domain=https://adr.github.io&sz=128)Architectural Decision Records+2![](https://www.google.com/s2/favicons?domain=https://adr.github.io&sz=128)Architectural Decision Records+2](https://adr.github.io/?utm_source=chatgpt.com)

### C. 各組織・プロジェクトが自由に決めるもの

以下はこちらに該当します。

```
docs/current-status.md
docs/project-context.md
docs/technical-decisions.md
docs/implementation-plan.md
docs/known-issues.md
docs/handoff.md
```

意味は分かりやすいですが、業界共通の予約名ではありません。たとえば、同じ内容でも次のような名前が使われます。

```
current-status.md
status.md
project-status.md
development-status.md
progress.md
state-of-project.md
```

したがって、社内の各プロジェクトでバラバラに命名すると、Copilotだけでなく人間も探しにくくなります。

## 2. プロジェクトごとに全部変えるのはおすすめしません

おすすめは、**「社内共通の骨格80％＋プロジェクト固有20％」**です。

### 社内共通にするもの

* ディレクトリ構成
* ファイル名
* 各文書の目的
* 必須見出し
* 更新ルール
* Copilotに参照させる方法
* 機密情報の区分
* ADRの書式

### プロジェクトごとに変えるもの

* 技術スタック
* 開発フェーズ
* テスト方法
* 用語
* 制約
* 対象コード
* プロジェクト固有の設計判断
* 個別のプロンプトやエージェント

たとえばFortranツールと`.nc`解析ツールで、ファイル名は同じにできます。

```
docs/project-context.md
docs/current-status.md
docs/decisions/
docs/implementation-plan.md
```

ただし、中身は当然異なります。

## 3. 社内汎用テンプレートとしての推奨構成

私は、最初からファイルを増やしすぎず、次の構成を社内標準の候補にするのがよいと思います。

```
project/
├── README.md
│
├── .github/
│   ├── copilot-instructions.md
│   ├── instructions/
│   │   └── *.instructions.md
│   ├── prompts/
│   │   └── *.prompt.md
│   └── agents/
│       └── *.md
│
└── docs/
    ├── project-context.md
    ├── current-status.md
    ├── implementation-plan.md
    ├── known-issues.md
    └── decisions/
        ├── README.md
        ├── 0001-xxxx.md
        └── 0002-xxxx.md
```

それぞれの役割は次のように固定します。

| ファイル | 社内標準としての役割 |
| --- | --- |
| `README.md` | 人間向けの入口、概要、実行方法 |
| `.github/copilot-instructions.md` | Copilotが常に守る短い指示 |
| `docs/project-context.md` | 目的、範囲、用語、制約、全体構成 |
| `docs/current-status.md` | 現在地点、実装済み、進行中、次の作業 |
| `docs/implementation-plan.md` | 今後の実装段階と完了条件 |
| `docs/known-issues.md` | 既知問題、制限、未検証事項 |
| `docs/decisions/` | 重要な意思決定をADRとして保存 |

## 4. `technical-decisions.md`はADRに変えた方がよい

小規模プロジェクトであれば、次の1ファイルでも構いません。

```
docs/technical-decisions.md
```

しかし、長期間の開発や社内横断利用を想定するなら、1つのファイルへ追記するより、判断を1件ずつADRとして分ける方が管理しやすくなります。

```
docs/decisions/
├── README.md
├── 0001-use-fparser2-for-parsing.md
├── 0002-require-approval-before-rewrite.md
├── 0003-test-with-synthetic-fixtures.md
└── 0004-separate-fortran-and-python-phases.md
```

ADRでは、採用済みの記録を後から上書きするのではなく、方針が変わったときに新しいADRを作り、以前の判断を置き換えたことを記録します。これにより「現在の方針」だけでなく「なぜ途中で変更したか」も残せます。Microsoftの現行ガイダンスでも、ADRを追加型の記録として扱い、変更時には以前の記録を置き換える新しいADRを作る方式が推奨されています。[![](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com&sz=128)Microsoft Learn+2![](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com&sz=128)Microsoft Learn+2](https://learn.microsoft.com/en-us/azure/well-architected/architect-role/architecture-decision-record?utm_source=chatgpt.com)

たとえば次の形です。

```Markdown
# 0002: 自動修正前に人間の承認を必須とする

- Status: Accepted
- Date: 2026-08-06

## Context

レガシーFortranコードでは、暗黙型やCOMMONの利用により、
機械的な修正が数値結果を変更する可能性がある。

## Decision

修正候補の検出とレポート生成までは自動化する。
ソースコードへの修正適用には人間の承認を必要とする。

## Alternatives considered

- すべて自動修正する
- 検出だけ行い修正機能を持たせない

## Consequences

- 安全性は向上する
- 完全自動化より作業時間は増える
- 承認結果を監査ログとして保存する必要がある
```

## 5. `current-status.md`は便利ですが、更新ルールが重要です

`current-status.md`という名前自体は標準ではありませんが、AI開発との相性は非常によいです。ただし、放置されると古い情報をCopilotに与える危険があります。

そのため、社内標準ではファイル名だけでなく、次の更新規則まで定めた方がよいです。

```Markdown
# Current Status

Last updated: 2026-08-06
Maintainer: Project team
Current phase: CALL argument analysis

## Goal of the current phase

## Completed

## In progress

## Blocked

## Next actions

## Verification status

## Important constraints

## Documents to read next
```

特に重要なのは次の項目です。

* `Last updated`
* `Current phase`
* `Next actions`
* `Verification status`
* `Important constraints`

これがあれば、Copilotが「実装済み」と「予定」を混同しにくくなります。

## 6. `copilot-instructions.md`に全情報を詰め込まない

社内共通化すると、すべてを`.github/copilot-instructions.md`へ入れたくなりますが、これは避けた方がよいです。

GitHubもカスタム指示について、短く自己完結した文として、リポジトリで作業するために必要な情報を記載するよう案内しています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/en/copilot/concepts/prompting/response-customization?utm_source=chatgpt.com)

役割は次のように分けます。

```
copilot-instructions.md
    常に守るルール
    ↓
project-context.md
    プロジェクトを理解するための情報
    ↓
current-status.md
    今どこまで進んでいるか
    ↓
implementation-plan.md
    今後どう進めるか
    ↓
decisions/
    なぜその方針になったか
```

`copilot-instructions.md`には、たとえば次の程度を記載します。

```Markdown
- Before modifying code, read `docs/project-context.md`.
- For the current development state, read `docs/current-status.md`.
- Follow accepted decisions under `docs/decisions/`.
- Do not describe planned functionality as implemented.
- Update `docs/current-status.md` after completing a development phase.
- Do not use confidential production data in tests.
```

## 7. 社内標準としての現実的な結論

次のように整理するのが最も汎用的です。

### 固定する

```
README.md
.github/copilot-instructions.md
.github/instructions/
.github/prompts/
.github/agents/
docs/project-context.md
docs/current-status.md
docs/implementation-plan.md
docs/known-issues.md
docs/decisions/
```

### プロジェクトごとに追加可能にする

```
docs/data-model.md
docs/api-contracts.md
docs/validation-plan.md
docs/migration-plan.md
docs/security-model.md
docs/numerical-verification.md
docs/experiment-plan.md
```

たとえばFortranプロジェクトなら、

```
docs/numerical-verification.md
docs/legacy-code-assumptions.md
```

`.nc`解析プロジェクトなら、

```
docs/file-format-assumptions.md
docs/gui-behavior.md
docs/measurement-validation.md
```

を追加できます。

したがって、**ファイル名まで完全にプロジェクト任せにするのではなく、共通のコア文書を社内標準化し、専門領域の文書だけを追加する方式**が適しています。

特に今回の用途では、`technical-decisions.md`だけは固定ファイルにせず、より一般的な`docs/decisions/`配下のADR方式へ置き換えるのがよいでしょう。これならソフトウェア、解析、実験、研究PoCなど、社内の異なる種類のプロジェクトでも共通運用できます。

* * *

**You:**

業界標準的なファイル名やフォルダ構成を教えてくださいましたが、参考にしたサイトや文献があればリンクを教えてください。

* * *

**ChatGPT:**

あります。ただし、先ほどの説明については一つ訂正があります。

**`docs/current-status.md`や`docs/project-context.md`まで含めて「業界標準的なファイル名」と表現したのは少し広すぎました。**  
正式にツールから規定されているもの、広く普及した慣行、私が社内運用向けに提案したものが混在しています。

## 1. GitHub・Copilotが正式に規定しているもの

### `.github/copilot-instructions.md`

GitHub Copilotのリポジトリ共通指示ファイルです。GitHub公式ドキュメントで、このパスとファイル名が明示されています。 [![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions-in-your-ide/add-repository-instructions-in-your-ide?utm_source=chatgpt.com)

* Adding repository custom instructions for GitHub Copilot
* About customizing GitHub Copilot responses

これは単なる命名例ではなく、Copilotに自動認識させるための正式な名前です。

### `.github/instructions/*.instructions.md`

ファイル種別やディレクトリ別に適用する指示ファイルです。VS Code版Copilotなどが対応していますが、対応範囲は環境によって異なります。GitHub公式の対応表を確認するのが重要です。 [![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/en/copilot/reference/custom-instructions-support?utm_source=chatgpt.com)

* Support for different types of custom instructions
* Use custom instructions in VS Code

### `.github/prompts/*.prompt.md`

VS Codeで再利用可能なプロンプトを保存する正式なワークスペース配置です。これはGitHubリポジトリ一般の標準というより、**VS CodeのGitHub Copilot機能が定める形式**です。 [![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/prompt-files?utm_source=chatgpt.com)

* Use prompt files in VS Code

公式ドキュメントでは、ワークスペース用プロンプトファイルを次の場所に保存します。

```
.github/prompts/
└── implement-feature.prompt.md
```

### `.github/agents/*.agent.md`

カスタムエージェントをリポジトリ内で共有するための配置です。GitHubとVS Codeの公式ドキュメントで説明されています。 [![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+2![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+2](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/create-custom-agents?utm_source=chatgpt.com)

* Custom agents in VS Code
* Creating custom agents for Copilot cloud agent
* Custom agents configuration

ただし、Copilotの実行環境によって、`.md`と`.agent.md`の認識条件や対応プロパティが異なることがあります。社内標準では、現在のVS Code・Copilot環境に合わせて`.agent.md`へ統一するのが分かりやすいと思います。

## 2. GitHubで正式に認識される一般的なファイル

### `README.md`

GitHubは、`.github`、リポジトリ直下、または`docs`ディレクトリに置かれたREADMEを認識して表示します。一般にはリポジトリ直下の`README.md`が最も分かりやすい配置です。 [![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes?utm_source=chatgpt.com)

* About the repository README file

### Community Health Files

GitHubでは、次のようなファイルを「Community Health Files」として認識します。

```
README.md
CONTRIBUTING.md
CODE_OF_CONDUCT.md
SECURITY.md
SUPPORT.md
LICENSE
.github/ISSUE_TEMPLATE/
.github/PULL_REQUEST_TEMPLATE.md
```

GitHubはこれらを`.github`、リポジトリ直下、`docs`などの対応場所から認識します。 [![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+2![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+2](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file?utm_source=chatgpt.com)

* Creating a default community health file
* About community profiles for public repositories
* About issue and pull request templates

これらはGitHub上で比較的強く標準化されている名称です。

## 3. ADRの参考文献

### ADR公式コミュニティサイト

ADRはArchitecture Decision Recordの略で、1件の重要な設計判断と、その背景・理由・影響を記録する方法です。複数のADRを集めたものをdecision logとして扱います。 [![](https://www.google.com/s2/favicons?domain=https://adr.github.io&sz=128)Architectural Decision Records+1](https://adr.github.io/?utm_source=chatgpt.com)

* Architectural Decision Records

### ADRテンプレート集

Nygard形式、MADR、Y-Statementなど、複数のADR形式が紹介されています。これを見ると、ADRという考え方は普及していても、ファイル名や見出し構成が一種類に統一されているわけではないことが分かります。 [![](https://www.google.com/s2/favicons?domain=https://adr.github.io&sz=128)Architectural Decision Records+1](https://adr.github.io/adr-templates/?utm_source=chatgpt.com)

* ADR Templates

典型的なNygard形式は次の項目を持ちます。

```
Title
Status
Context
Decision
Consequences
```

### MADR

MarkdownでADRを管理するための、より詳細で実用的なテンプレートです。MADRの公式例では、`docs/decisions/0000-...md`という配置が使われています。私が前回提案した`docs/decisions/`は、特にこの慣行を参考にしています。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/adr/madr?utm_source=chatgpt.com)

* About MADR
* MADR GitHub repository
* Use Markdown Architectural Decision Records

MADRの例は次のような形です。

```
docs/
└── decisions/
    ├── 0000-use-markdown-architectural-decision-records.md
    ├── 0001-use-postgresql.md
    └── 0002-require-human-approval.md
```

ただし、`docs/adr/`や`docs/architecture/decisions/`を使う組織もあり、保存先まで完全な標準ではありません。

### MicrosoftのADRガイド

Microsoft Azure Well-Architected FrameworkにもADRの運用ガイドがあります。重要な判断、却下した代替案、背景、根拠、影響を記録し、過去の承認済みADRを書き換えるのではなく、新しいADRで置き換える方法を説明しています。 [![](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com&sz=128)Microsoft Learn+2![](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com&sz=128)Microsoft Learn+2](https://learn.microsoft.com/en-us/azure/well-architected/architect-role/architecture-decision-record?utm_source=chatgpt.com)

* アーキテクチャ デシジョン レコードを維持する（日本語）
* Maintain an architecture decision record（英語）

社内へADR方式を説明する際には、このMicrosoftの資料が比較的紹介しやすいと思います。

## 4. アーキテクチャ文書全体の構成を考える資料

### arc42

arc42は、ソフトウェアアーキテクチャ文書の構成テンプレートです。目的、制約、コンテキスト、構成要素、実行時ビュー、配置、品質要件、リスク、技術的負債などを体系的に整理できます。 [![](https://www.google.com/s2/favicons?domain=https://arc42.org&sz=128)arc42+1](https://arc42.org/documentation/?utm_source=chatgpt.com)

* arc42公式サイト
* arc42 Template Overview
* arc42 Documentation
* arc42公式テンプレート

arc42は「ファイル名の標準」というより、**アーキテクチャ文書に何を書くかの標準的なテンプレート候補**です。

### Diátaxis

Diátaxisは技術文書を次の4種類に分ける考え方です。

* Tutorials
* How-to guides
* Reference
* Explanation

利用者向けドキュメント全体を社内標準化する場合に参考になります。 [![](https://www.google.com/s2/favicons?domain=https://diataxis.fr&sz=128)Diátaxis+1](https://diataxis.fr/?utm_source=chatgpt.com)

* Diátaxis公式サイト
* Diátaxis in five minutes

ただし、`current-status.md`や`implementation-plan.md`のような開発管理文書を直接規定するものではありません。

## 5. 私が提案したファイルの出典上の位置付け

| ファイル・フォルダ | 位置付け |
| --- | --- |
| `README.md` | GitHubが正式に認識 |
| `.github/copilot-instructions.md` | GitHub Copilotが正式に規定 |
| `.github/instructions/*.instructions.md` | Copilotが正式に規定 |
| `.github/prompts/*.prompt.md` | VS Code Copilotが正式に規定 |
| `.github/agents/*.agent.md` | Copilot／VS Codeが正式に規定 |
| `docs/decisions/0001-xxx.md` | MADRなどで使われる広く知られた慣行 |
| `docs/project-context.md` | 私が提案した社内標準候補 |
| `docs/current-status.md` | 私が提案した社内標準候補 |
| `docs/implementation-plan.md` | 私が提案した社内標準候補 |
| `docs/known-issues.md` | 一般的な名称だが、正式な業界標準ではない |
| `docs/technical-decisions.md` | 一般的な名称例。長期運用ではADRへの分割を推奨 |

したがって、次の構成は**一つのサイトに掲載された標準構成をそのまま転載したものではありません**。

```
docs/
├── project-context.md
├── current-status.md
├── implementation-plan.md
├── known-issues.md
└── decisions/
```

これは、

* GitHub／VS Codeが正式に定めるCopilot用ファイル
* GitHubの一般的なリポジトリ文書
* ADR／MADR
* AIに現在の開発状況を渡す必要性
* 社内の複数プロジェクトで統一運用する目的

を組み合わせた、**社内向けの推奨構成案**です。

最初に読む資料としては、GitHubのリポジトリカスタム指示、VS Codeのプロンプトファイル、ADR、MADR、MicrosoftのADRガイドの順が、今回の目的には特に参考になります。

* * *

**You:**

VScodeやgithub、anthoropicはプロンプトエンジニアリングやコンテキストエンジニアリングに関して、用意すべきファイルやフォルダ構成について提案してはいないの？

* * *

**ChatGPT:**

はい。**現在のVS Code、GitHub Copilot、Anthropic Claude Codeは、AIに与える指示・知識・手順をどのファイルへ分けるかについて、かなり具体的な構成を提案しています。**

ただし、各社が標準化しているのは主に次の領域です。

* AIが常時読む指示
* 特定ファイルだけに適用する指示
* 再利用するプロンプトやワークフロー
* 専門エージェント
* ツール実行・権限・フック
* 外部サービスとの接続

一方、以前提案した`docs/current-status.md`や`docs/implementation-plan.md`のような**プロジェクト管理文書の名称までは規定していません**。

## 1. VS Codeが提案している構成

VS Codeは「Agent Customization」として、段階的に次のものを導入することを公式に勧めています。

1. プロジェクト全体の指示
2. 言語・フォルダ別の指示
3. 再利用可能なスキル
4. 専門エージェント
5. MCPやHooksによる外部連携・自動化

公式ドキュメントでも、最初に`.github/copilot-instructions.md`を作り、その後、必要に応じて対象別指示、Skills、Custom Agents、MCP、Hooksを追加する流れが示されています。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/overview)

典型的には次の構成です。

```
project/
├── .github/
│   ├── copilot-instructions.md
│   ├── instructions/
│   │   ├── python.instructions.md
│   │   ├── fortran.instructions.md
│   │   └── tests.instructions.md
│   ├── prompts/
│   │   ├── implement-feature.prompt.md
│   │   └── review-code.prompt.md
│   ├── agents/
│   │   ├── planner.agent.md
│   │   └── reviewer.agent.md
│   └── skills/
│       └── regression-testing/
│           ├── SKILL.md
│           ├── scripts/
│           └── references/
└── ...
```

### `.github/copilot-instructions.md`

全チャットに自動適用する、プロジェクト全体の指示です。

VS Codeは、ここに次のような内容を置くことを例示しています。

* コーディング規約
* 採用技術や推奨ライブラリ
* アーキテクチャ方針
* セキュリティ要件
* エラー処理方針
* ドキュメント規約

公式にも、まずこの1ファイルから始めるよう推奨されています。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/agent-customization/custom-instructions)

### `.github/instructions/*.instructions.md`

言語、フレームワーク、ディレクトリなどに応じて条件付きで適用する指示です。

```
.github/instructions/
├── fortran.instructions.md
├── python.instructions.md
└── tests.instructions.md
```

たとえばFortranファイルにだけ「暗黙型を新規導入しない」という指示を適用し、Pythonファイルには別の規約を適用できます。VS Codeは、全体指示と対象別指示を分けることを正式に提案しています。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/custom-instructions)

### `.github/prompts/*.prompt.md`

繰り返し使う、比較的短いタスク用のプロンプトです。

```
.github/prompts/
├── analyze-common-blocks.prompt.md
├── generate-tests.prompt.md
└── update-documentation.prompt.md
```

`.prompt.md`には、利用するエージェント、モデル、ツール、説明などをYAML frontmatterで指定できます。また、別のワークスペースファイルをMarkdownリンクで参照でき、共通指示をプロンプトごとに重複させないよう推奨されています。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/prompt-files)

ただし、現在のVS Codeドキュメントでは、Agent Host上のエージェントはPrompt Filesを使わず、既存プロンプトをAgent Skillへ移す方向も示されています。したがって、今後長期利用する複雑なワークフローでは、`.prompt.md`よりSkillsを優先するのが安全です。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/prompt-files)

### `.github/agents/*.agent.md`

Plan担当、実装担当、レビュー担当など、役割と使用可能ツールを固定するものです。

```
.github/agents/
├── planner.agent.md
├── implementer.agent.md
└── security-reviewer.agent.md
```

VS Codeでは、ワークスペース用の既定配置として`.github/agents`、Claude形式との互換配置として`.claude/agents`も認識します。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/agent-customization/custom-agents)

### `.github/skills/<name>/SKILL.md`

繰り返し利用する専門知識や、複数段階のワークフローを保存します。

```
.github/skills/
└── numerical-regression/
    ├── SKILL.md
    ├── scripts/
    │   └── compare_results.py
    └── references/
        └── acceptance-criteria.md
```

Skillsは単なるプロンプトではなく、指示、スクリプト、テンプレート、参考資料を一つのフォルダにまとめる仕組みです。VS Codeは`.github/skills/`、`.claude/skills/`、`.agents/skills/`をプロジェクト用Skillの配置として認識します。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/agent-customization/agent-skills)

## 2. GitHub Copilotが提案している構成

GitHub Copilotの構成は、VS Codeのものとかなり重なります。これは、VS CodeのAI機能の中心がGitHub Copilotだからです。

GitHubが正式に規定している主要な構成は次のとおりです。

```
project/
└── .github/
    ├── copilot-instructions.md
    ├── instructions/
    │   └── *.instructions.md
    ├── prompts/
    │   └── *.prompt.md
    ├── agents/
    │   └── *.agent.md
    └── skills/
        └── <skill-name>/
            └── SKILL.md
```

### GitHubが示している使い分け

| 仕組み | 用途 |
| --- | --- |
| `copilot-instructions.md` | リポジトリ全体で常に守る指示 |
| `*.instructions.md` | パスや言語別に適用する指示 |
| `*.prompt.md` | 明示的に呼び出す再利用プロンプト |
| `*.agent.md` | 特定の役割・ツールを持つエージェント |
| `SKILL.md` | 必要なときだけ読み込む知識・手順・スクリプト |

GitHub公式は、`.github/copilot-instructions.md`を「プロジェクトをどう理解し、ビルドし、テストし、変更を検証するか」というコンテキストをCopilotに与える場所と説明しています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot)

パス固有指示については、`.github/instructions/**/*.instructions.md`が正式な配置です。利用可能なCopilot機能やIDEによって対応範囲が異なるため、GitHubはサポート表も公開しています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/en/copilot/reference/custom-instructions-support?utm_source=chatgpt.com)

Custom Agentは`.github/agents/*.agent.md`、Agent Skillsは`.github/skills/<skill>/SKILL.md`が代表的な構成です。Skillsについては、GitHubも`.github/skills`だけでなく`.claude/skills`と`.agents/skills`を認識しています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/create-custom-agents?utm_source=chatgpt.com)

## 3. Anthropicが提案している構成

Claude Codeは、3社の中でも特に明確に**`.claude`ディレクトリ全体の役割分担**を示しています。

公式ドキュメントには「Explore the `.claude` directory」という専用ページがあり、目的ごとにどのファイルを編集するかが表として整理されています。[![](https://www.google.com/s2/favicons?domain=https://code.claude.com&sz=128)Claude+1](https://code.claude.com/docs/en/claude-directory)

代表的な構成は次のようになります。

```
project/
├── CLAUDE.md
├── CLAUDE.local.md
├── .mcp.json
│
└── .claude/
    ├── CLAUDE.md
    ├── settings.json
    ├── settings.local.json
    ├── rules/
    │   ├── code-style.md
    │   ├── testing.md
    │   └── security.md
    ├── skills/
    │   ├── regression-testing/
    │   │   ├── SKILL.md
    │   │   ├── scripts/
    │   │   └── references/
    │   └── release/
    │       └── SKILL.md
    ├── agents/
    │   ├── planner.md
    │   └── reviewer.md
    ├── workflows/
    │   └── ...
    └── output-styles/
        └── ...
```

### `CLAUDE.md`

Claude Codeが毎回読む、プロジェクト共通のコンテキストです。

Anthropicはここに、次のような情報を置くよう提案しています。

* ビルド・テストコマンド
* コーディング規約
* プロジェクト構造
* アーキテクチャ
* 命名規則
* 共通ワークフロー
* 常に守る禁止事項

プロジェクト用はルートの`CLAUDE.md`または`.claude/CLAUDE.md`に置けます。個人用のプロジェクト設定は`CLAUDE.local.md`へ置き、Git管理から除外する構成です。[![](https://www.google.com/s2/favicons?domain=https://code.claude.com&sz=128)Claude+1](https://code.claude.com/docs/en/memory?utm_source=chatgpt.com)

### `.claude/rules/`

大規模プロジェクトで`CLAUDE.md`が長くなった場合に、トピック別・パス別へ分割する公式の仕組みです。

```
.claude/rules/
├── fortran.md
├── python.md
├── testing.md
├── security.md
└── gui/
    └── qt.md
```

Anthropicは、全セッションで必要なものを`CLAUDE.md`、言語・ディレクトリ別のものを`.claude/rules/`、特定タスクでだけ必要なものをSkillsへ置くよう整理しています。[![](https://www.google.com/s2/favicons?domain=https://code.claude.com&sz=128)Claude+1](https://code.claude.com/docs/en/memory)

また、目安として`CLAUDE.md`を200行未満に保ち、長くなった情報をRulesやSkillsへ分けることも推奨しています。[![](https://www.google.com/s2/favicons?domain=https://code.claude.com&sz=128)Claude](https://code.claude.com/docs/en/features-overview)

### `.claude/skills/<name>/SKILL.md`

特定のドメイン知識や、再利用する作業手順です。

```
.claude/skills/
└── fix-common-mismatch/
    ├── SKILL.md
    ├── scripts/
    │   └── validate_common.py
    └── references/
        └── common-rules.md
```

Claude Codeでは、`CLAUDE.md`の内容が手順書のように長くなってきた場合、その部分をSkillに移すことが明確に勧められています。Skillの本文は必要になったときだけ読み込まれるため、常時コンテキストを消費しません。[![](https://www.google.com/s2/favicons?domain=https://docs.anthropic.com&sz=128)Claude Platform Docs+1](https://docs.anthropic.com/en/docs/claude-code/skills)

### `.claude/agents/`

調査、計画、レビューなどを独立したコンテキストで実行するサブエージェントです。

```
.claude/agents/
├── codebase-explorer.md
├── implementation-planner.md
└── numerical-reviewer.md
```

サブエージェントが大量のコードやログを読み、本体の会話へ要約だけを返すことで、メインコンテキストを汚さない設計です。[![](https://www.google.com/s2/favicons?domain=https://docs.anthropic.com&sz=128)Claude Platform Docs+1](https://docs.anthropic.com/en/docs/claude-code/sub-agents)

### `.claude/settings.json`

プロンプトそのものではなく、次を制御します。

* 読み書きを許可するファイル
* 実行可能なコマンド
* Hooks
* 環境変数
* プラグイン
* セキュリティ設定

チーム共有設定は`.claude/settings.json`、個人・端末固有設定は`.claude/settings.local.json`です。[![](https://www.google.com/s2/favicons?domain=https://code.claude.com&sz=128)Claude](https://code.claude.com/docs/en/configuration)

## 4. 3者に共通する考え方

実質的には、各社とも次の5層へ収束しています。

```
1. 常時適用する短い指示
2. 対象別・パス別の指示
3. 必要時だけ読む知識やワークフロー
4. 専門エージェント
5. 決定論的な設定・ツール・Hooks
```

対応関係は次のようになります。

| 概念 | VS Code／Copilot | Claude Code |
| --- | --- | --- |
| 常時指示 | `.github/copilot-instructions.md`、`AGENTS.md` | `CLAUDE.md` |
| パス別指示 | `.github/instructions/*.instructions.md` | `.claude/rules/*.md` |
| 再利用タスク | `.github/prompts/*.prompt.md` | Skills |
| 再利用知識・複雑な手順 | `.github/skills/*/SKILL.md` | `.claude/skills/*/SKILL.md` |
| 専門エージェント | `.github/agents/*.agent.md` | `.claude/agents/*.md` |
| 設定・Hooks | VS Code設定、Hooks | `.claude/settings.json` |
| MCP | `.vscode/mcp.json`など | `.mcp.json` |

## 5. 最も標準に近いのはAgent Skills

現在、複数製品間で最も共通化が進んでいるのは、次の構成です。

```
<skills-directory>/
└── <skill-name>/
    ├── SKILL.md
    ├── scripts/
    ├── references/
    └── assets/
```

Agent Skillsはオープン標準であり、VS Code、GitHub Copilot、Claude Codeのいずれも対応しています。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+2![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+2](https://code.visualstudio.com/docs/agent-customization/agent-skills)

したがって、社内の複数プロジェクト・複数AIで共有したい知識や手順については、Skills形式が有力です。

たとえば、

```
.agents/
└── skills/
    ├── project-status-update/
    │   └── SKILL.md
    ├── numerical-regression/
    │   ├── SKILL.md
    │   └── scripts/
    └── confidential-data-filter/
        ├── SKILL.md
        └── references/
```

のようなベンダー中立的な配置も考えられます。VS CodeとGitHub Copilotは`.agents/skills/`を正式に認識します。Claude Codeでは`.claude/skills/`が標準配置なので、社内で複数ツールを混用する場合は、どちらかを正本にして同期またはリンクする運用が必要です。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/agent-customization/agent-skills)

## 6. `current-status.md`などはどう位置付けるべきか

重要なのは、各社の公式構成にも次のファイルは規定されていないことです。

```
docs/current-status.md
docs/project-context.md
docs/implementation-plan.md
docs/known-issues.md
docs/decisions/
```

これらは、CopilotやClaude Codeが自動認識する専用形式ではなく、**AIと人間が共有するプロジェクト文書**です。

したがって、次のように組み合わせるのが自然です。

```
project/
├── README.md
│
├── docs/
│   ├── project-context.md
│   ├── current-status.md
│   ├── implementation-plan.md
│   ├── known-issues.md
│   └── decisions/
│
├── .github/
│   ├── copilot-instructions.md
│   ├── instructions/
│   ├── prompts/
│   ├── agents/
│   └── skills/
│
└── .claude/
    ├── CLAUDE.md
    ├── rules/
    ├── agents/
    ├── skills/
    └── settings.json
```

役割は次のように分けます。

* `docs/`：プロジェクトの事実を記録する正本
* `.github/`：Copilotがその事実をどう参照し、どう作業するか
* `.claude/`：Claude Codeがその事実をどう参照し、どう作業するか
* Skills：複数プロジェクトや複数AIで再利用する手順
* Agents：役割と権限の分離
* Hooks：必ず実行させる検証や更新

つまり、以前の提案は各社の公式構成と競合するものではありません。

> **各社の公式ファイルはAIへの入口、`docs/`はプロジェクト情報の正本**

という関係です。

特に社内横断利用では、プロジェクト固有情報を`.github/copilot-instructions.md`と`CLAUDE.md`の両方へ重複記載するより、共通の`docs/`文書を正本として、各AI用ファイルから必要な文書を参照させる構成が保守しやすいです。

* * *

**You:**

なるほど。そして、.github/copilot-instructions.mdにdocs/内のファイルを適宜参照するように指示しておくということですね。そして、このdocs/内のファイルにあたるものは業界標準は存在しないという認識で良いでしょうか？論文等で提案もされている有名な方法もない？

* * *

**ChatGPT:**

はい、その理解でほぼ正しいです。

`.github/copilot-instructions.md`には、すべての情報を直接書き込むのではなく、

* どの文書が正本か
* どの状況でどの文書を読むか
* 実装後にどの文書を更新するか

を短く指示しておき、詳細は`docs/`以下へ置く設計が適しています。

ただし、**`docs/current-status.md`、`docs/project-context.md`、`docs/implementation-plan.md`というファイル名・組み合わせについて、業界共通の標準はありません。**

## 「標準がない」の正確な意味

標準化されていないのは、次のような具体的な構成です。

```
docs/
├── project-context.md
├── current-status.md
├── implementation-plan.md
├── known-issues.md
└── technical-decisions.md
```

「プロジェクトの目的、現在状況、計画、判断理由を記録する」という考え方自体は古くから存在します。しかし、

* どのファイルに分けるか
* ファイル名を何にするか
* `docs/`のどこに置くか
* AIにどの順番で読ませるか
* いつ更新するか

まで統一した国際標準やデファクトスタンダードは、現時点ではありません。

ソフトウェアアーキテクチャ知識管理の研究では、要求、設計判断、その根拠、経験などを保持する必要性が長年論じられていますが、文献レビューでも多数の異なる管理手法が扱われており、単一のファイル構成に収束しているわけではありません。[![](https://www.google.com/s2/favicons?domain=https://www.sciencedirect.com&sz=128)ScienceDirect+1](https://www.sciencedirect.com/science/article/pii/S0950584916301707?utm_source=chatgpt.com)

## ただし、部分ごとには有名な方法があります

「共有コンテキスト全体」の標準はありませんが、その一部を扱う有名な方式はあります。

### 1. ISO/IEC/IEEE 42010

システムやソフトウェアの**アーキテクチャ記述**についての国際標準です。

ステークホルダー、関心事、ビューポイント、ビュー、モデルなど、アーキテクチャ記述に必要な概念と構造を定めています。ただし、`docs/architecture.md`のようなファイル名やMarkdownのフォルダ構成までは規定しません。現在の版はISO/IEC/IEEE 42010:2022です。[![](https://www.google.com/s2/favicons?domain=https://www.iso.org&sz=128)ISO](https://www.iso.org/standard/74393.html?utm_source=chatgpt.com)

したがって、

```
docs/architecture/
```

の中身を設計する際の理論的な土台にはなりますが、プロジェクト全体の状況管理方式ではありません。

### 2. arc42

arc42は、アーキテクチャ文書を次のような章に分ける有名なテンプレートです。

1. 目的とゴール
2. 制約
3. コンテキストとスコープ
4. ソリューション戦略
5. ビルディングブロック
6. 実行時ビュー
7. 配置ビュー
8. 横断的概念
9. アーキテクチャ上の判断
10. 品質要求
11. リスクと技術的負債
12. 用語集

arc42はさまざまな技術・業種・開発手法に適用できるアーキテクチャ文書テンプレートとして提供されています。[![](https://www.google.com/s2/favicons?domain=https://arc42.org&sz=128)arc42+1](https://arc42.org/?utm_source=chatgpt.com)

たとえば社内標準として、次のように採用できます。

```
docs/
└── architecture/
    ├── 01-goals.md
    ├── 02-constraints.md
    ├── 03-context.md
    ├── 04-solution-strategy.md
    ├── 05-building-blocks.md
    ├── 09-decisions.md
    └── 11-risks.md
```

ただし、この分割方法やファイル名は実装側で決めるもので、arc42自体がこの名前を強制しているわけではありません。

### 3. ADR／MADR

技術的判断の記録については、ADRが最も有名な方法の一つです。

1件の判断ごとに、

* Context
* Decision
* Alternatives
* Consequences
* Status

などを記録します。Nygard形式やMADRなど複数のテンプレートがあります。[![](https://www.google.com/s2/favicons?domain=https://adr.github.io&sz=128)Architectural Decision Records+2![](https://www.google.com/s2/favicons?domain=https://adr.github.io&sz=128)Architectural Decision Records+2](https://adr.github.io/?utm_source=chatgpt.com)

よくある構成は次のとおりです。

```
docs/
└── decisions/
    ├── 0001-use-fparser2.md
    ├── 0002-require-human-approval.md
    └── 0003-use-synthetic-test-data.md
```

この`docs/decisions/0001-...md`形式は広く知られた慣行ですが、国際標準の固定パスではありません。

それでも、`technical-decisions.md`という独自の長大なファイルを作るより、ADR方式を採用する方が、社内で説明しやすく汎用性も高いでしょう。

## 4. Software Architecture Knowledge Management

論文上では、今回の考え方に最も近い既存分野は、**Software Architecture Knowledge Management：ソフトウェアアーキテクチャ知識管理**です。

この分野では、アーキテクチャ知識を単なる完成図ではなく、

* 要求
* 制約
* 設計判断
* 判断理由
* 却下した選択肢
* 経験
* パターン
* リスク
* システム進化の履歴

として管理します。

研究では、文書ベース、Wiki、オントロジー、専用ツール、意思決定中心の方式などが比較されています。しかし、ここでも「この5つのMarkdownをこの名前で置く」という統一方式は確立されていません。ファイルベースの文書は業界で一般的に使われていますが、情報の検索・構造化方法には複数のアプローチがあります。[![](https://www.google.com/s2/favicons?domain=https://www.sciencedirect.com&sz=128)ScienceDirect+2![](https://www.google.com/s2/favicons?domain=https://research.vu.nl&sz=128)Vrije Universiteit Amsterdam+2](https://www.sciencedirect.com/science/article/pii/S0167642315003172?utm_source=chatgpt.com)

したがって、

> `docs/`をプロジェクト知識の正本にする

という考え方には研究上の背景がありますが、

> `project-context.md`と`current-status.md`を作る

という具体的な名前は、標準から直接導かれたものではありません。

## 5. AI開発ではGitHub Spec Kitが近い

AIを使った開発については、GitHubのSpec Kitがかなり近い構造を提案しています。

Spec Kitは、仕様を中心に置くSpec-Driven Developmentとして、概ね次の成果物を使います。

```
constitution
spec
plan
tasks
checklist
```

* `constitution`：プロジェクトで守る原則
* `spec`：何を作るか
* `plan`：どう実装するか
* `tasks`：具体的な作業分解
* `checklist`：仕様の品質確認

GitHubはSpec Kitを「仕様をAI支援開発の中心に置く」ためのツールキットとして位置付けています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+2![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+2](https://github.github.com/spec-kit/?utm_source=chatgpt.com)

これは、以前議論した、

```
feature-brief.md
implementation-plan.md
tasks.md
```

にかなり近い考え方です。

ただし、Spec Kitは主に**機能単位の仕様から実装まで**を管理する方式です。次の情報をすべて扱うものではありません。

* プロジェクト全体の現在状況
* 過去に失敗した試行
* 現在の検証レベル
* 社内コードをまだ適用できないという制約
* 複数機能を横断した開発履歴
* チャット間の引き継ぎ

したがって、Spec Kitだけで`current-status.md`相当を完全に代替できるわけではありません。

## 6. AI用の「プロジェクト記憶」は研究が始まった段階

AIコーディングエージェント向けには、2026年になって「プロジェクトの知識を永続化する方法」を直接扱う研究が増えています。

たとえば、Codified Contextという提案では、

1. 常時読み込む短いconstitution
2. 専門エージェント
3. 必要時に読む仕様文書群

という3層の構成が提案されています。これは、私たちが議論していた、

```
copilot-instructions.md
agents/
docs/
```

の分離にかなり近い考え方です。ただし、単一のシステム開発を対象とした2026年の事例研究であり、確立した業界標準ではありません。[![](https://www.google.com/s2/favicons?domain=https://arxiv.org&sz=128)arXiv](https://arxiv.org/abs/2602.20478?utm_source=chatgpt.com)

PROJECTMEMという別の2026年の提案では、問題、試行、修正、判断、メモを追記型ログへ保存し、AI向けの短い要約を自動生成する方式が提示されています。これも興味深いですが、現時点では新しい研究・実装提案です。[![](https://www.google.com/s2/favicons?domain=https://arxiv.org&sz=128)arXiv](https://arxiv.org/abs/2606.12329?utm_source=chatgpt.com)

つまり、

> AIが会話をまたいでプロジェクト状態を共有するための文書構造

は、まさに現在研究と実務が進んでいる領域であり、まだ一つの有名方式に収束していません。

## 7. `AGENTS.md`や`CLAUDE.md`は標準に近づいている

AIへの**指示ファイル**については、プロジェクト知識文書より標準化が進んでいます。

`AGENTS.md`は、「人間向けのREADMEに対する、コーディングエージェント向けのREADME」として提案されたオープン形式です。GitHub Copilotも現在、`AGENTS.md`をリポジトリ指示としてサポートしています。[![](https://www.google.com/s2/favicons?domain=https://agents.md&sz=128)Agents+1](https://agents.md/?utm_source=chatgpt.com)

Anthropicでは`CLAUDE.md`が、プロジェクトごとの永続指示ファイルとして正式に用意されています。[![](https://www.google.com/s2/favicons?domain=https://code.claude.com&sz=128)Claude+1](https://code.claude.com/docs/en/memory?utm_source=chatgpt.com)

ただし、これらは主に、

* ビルド方法
* テスト方法
* コーディング規約
* 禁止事項
* 作業手順
* 参照すべき文書

を書くものです。

現在の進捗や仕様の全履歴を大量に詰め込む場所ではありません。

## 8. 現時点での整理

| 対象 | 有名な方法・標準 | ファイル名まで標準化 |
| --- | --- | --- |
| AIへの常時指示 | `AGENTS.md`、`CLAUDE.md`、`copilot-instructions.md` | 比較的されている |
| アーキテクチャ記述 | ISO/IEC/IEEE 42010 | されていない |
| アーキテクチャ文書構成 | arc42 | 章構成はあるがファイル名は任意 |
| 技術判断 | ADR／MADR | 慣行はあるが固定パスではない |
| 機能仕様から実装 | GitHub Spec Kit | Spec Kit内では規定 |
| プロジェクトの現在状況 | 統一方式なし | されていない |
| AI向けプロジェクト記憶 | 新しい研究・実装が複数存在 | まだ未成熟 |
| チャット間引き継ぎ | 統一方式なし | されていない |

## 社内標準にするなら

完全な独自方式を作るよりも、既存の有名な考え方を組み合わせるのがよいです。

```
project/
├── README.md
├── AGENTS.md
│
├── .github/
│   └── copilot-instructions.md
│
└── docs/
    ├── architecture/      # arc42を参考
    ├── decisions/         # ADR／MADR
    ├── specs/             # Spec Kit的な機能仕様
    ├── plans/             # 実装計画
    ├── status/            # 社内独自の現在状況
    └── handoffs/          # 社内独自の引き継ぎ
```

このうち、

* `architecture/`はarc42
* `decisions/`はADR
* `specs/`と`plans/`はSpec Kit
* `AGENTS.md`はAIエージェント指示

という既存方式に根拠があります。

一方、

* `status/`
* `handoffs/`

は、AI開発で実務上必要になっているものの、まだ有名な標準が存在しない部分です。ここを社内で独自標準化する、という位置付けが最も正確です。

したがって、**以前提案した`current-status.md`などを「業界標準」として採用するのではなく、既存方式でカバーされていない領域に対する社内標準として採用する**のが妥当です。

* * *

**You:**

spec kitにプロジェクト全体の現在状況を記録・共有させるようにカスタマイズすることは可能なのでしょうか？

* * *

**ChatGPT:**

はい、可能です。しかも現在のSpec Kitには、**プロジェクト全体の状況管理を追加するための正式な拡張手段**がかなり整っています。

さらに、完全な「プロジェクト状態ファイル」ではありませんが、GitHub公式ドキュメントには、複数の機能仕様を束ねる上位文書として、トップレベルの`ROADMAP.md`を置き、各機能に`planned`、`in-progress`、`done`の状態を持たせる方法も紹介されています。したがって、今回の考え方はSpec Kitの思想から外れたものではありません。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/concepts/spec-of-specs.html)

## 標準のSpec Kitだけでは不足する部分

Spec Kitの標準フローは基本的に、機能単位です。

```
specs/
└── 001-feature-name/
    ├── spec.md
    ├── plan.md
    ├── tasks.md
    ├── research.md
    ├── data-model.md
    └── contracts/
```

`/speckit.specify`、`/speckit.plan`、`/speckit.tasks`、`/speckit.implement`、`/speckit.converge`を、現在選択されている一つの機能に対して実行します。Spec Kitは`.specify/feature.json`によってアクティブな機能を追跡しますが、これはプロジェクト全体の進捗、過去の実装、ブロッカー、次の作業をまとめたものではありません。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/quickstart.html)

したがって、標準状態では次が分散しています。

* 機能の要求：各`spec.md`
* 実装方針：各`plan.md`
* 作業状況：各`tasks.md`
* 現在選択中の機能：`.specify/feature.json`
* プロジェクト原則：`.specify/memory/constitution.md`

これらを横断した**プロジェクト全体のスナップショット**は、別途追加する必要があります。

## 最も自然な構成

次のように、上位状態と機能単位の成果物を分けるのがよいと思います。

```
project/
├── ROADMAP.md
│
├── docs/
│   └── project-status.md
│
├── .specify/
│   ├── feature.json
│   ├── memory/
│   │   └── constitution.md
│   └── templates/
│       └── overrides/
│
├── specs/
│   ├── 001-common-analysis/
│   │   ├── spec.md
│   │   ├── plan.md
│   │   └── tasks.md
│   ├── 002-call-analysis/
│   │   ├── spec.md
│   │   ├── plan.md
│   │   └── tasks.md
│   └── 003-python-migration/
│       ├── spec.md
│       ├── plan.md
│       └── tasks.md
│
└── .github/
    └── copilot-instructions.md
```

役割は次のように分けます。

| 文書 | 役割 |
| --- | --- |
| `ROADMAP.md` | プロジェクト全体の機能一覧、依存関係、進行状態 |
| `docs/project-status.md` | 現在地点、ブロッカー、検証状態、次の行動 |
| `constitution.md` | 長期的に守る開発原則 |
| 各`spec.md` | 機能ごとの要求 |
| 各`plan.md` | 機能ごとの技術計画 |
| 各`tasks.md` | 機能ごとの実装タスクと完了状態 |

Spec Kit公式の「Spec of Specs」でも、大規模な機能を複数のサブ仕様へ分け、上位の`ROADMAP.md`で依存関係と状態を管理する方法が示されています。大きな横断プロジェクトではトップレベルの`ROADMAP.md`を使うことも明記されています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/concepts/spec-of-specs.html)

## `ROADMAP.md`と`project-status.md`は分けた方がよい

両者は似ていますが、更新頻度と目的が異なります。

### `ROADMAP.md`

比較的安定した計画です。

```Markdown
# Project Roadmap

| ID | Feature | Depends on | Status | Spec |
|---|---|---|---|---|
| F01 | COMMON analysis | — | done | `specs/001-common-analysis/` |
| F02 | CALL analysis | F01 | in-progress | `specs/002-call-analysis/` |
| F03 | Typo correction | F01, F02 | planned | — |
| F04 | gfortran migration | F01–F03 | planned | — |
| F05 | Python migration | F04 | planned | — |
```

### `docs/project-status.md`

現在の作業再開に必要な短いスナップショットです。

```Markdown
# Project Status

Last updated: 2026-08-06
Overall status: In progress
Current phase: CALL argument analysis
Active feature: `specs/002-call-analysis/`

## Completed

- COMMON block collection
- COMMON layout mismatch detection
- Synthetic fixture tests
- Approval candidate report generation

## In progress

- CALL-site and callee argument association
- Implicit type inference
- Suppression of false-positive kind mismatches

## Blocked

- Validation against production Fortran code
  - Production code is currently unavailable
  - Applicability has not been validated

## Verification status

- Unit tests: Passed
- Synthetic integration tests: Passed
- gfortran compile tests: Partial
- Production-code validation: Not performed
- Numerical regression against Absoft: Not performed

## Next action

Implement implicit-type-aware CALL argument comparison and add regression fixtures.

## Important constraints

- Do not claim production applicability
- Use synthetic fixtures only
- Require human approval before source rewriting
```

`ROADMAP.md`だけでも進行状態は表現できますが、実装上の注意事項や未検証事項、直近の次の作業まで書くには`project-status.md`が適しています。

## Spec Kitへの組み込み方は4段階あります

### 1. 最小構成：通常のMarkdownとして追加する

まずはこれで十分です。

```
ROADMAP.md
docs/project-status.md
```

`.github/copilot-instructions.md`に次のような指示を加えます。

```Markdown
## Project context

- Read `ROADMAP.md` to understand project-wide feature dependencies and status.
- Read `docs/project-status.md` before planning or implementing work.
- Treat feature artifacts under `specs/` as the source of truth for individual features.
- Do not describe planned or unverified functionality as implemented.
- After completing or materially changing a development phase, update
  `docs/project-status.md` and the corresponding `ROADMAP.md` entry.
```

これは最も簡単ですが、AIが更新を忘れる可能性はあります。

### 2. プロジェクトローカルのオーバーライドを使う

Spec Kitは、次の場所で標準テンプレートをプロジェクト単位に上書きできます。

```
.specify/templates/overrides/
```

公式には、テンプレートの解決順序が次のようになっています。

1. プロジェクトローカルのオーバーライド
2. Preset
3. Extension
4. Spec Kit本体

したがって、本体ファイルを直接改造せずに、標準コマンドや成果物へ社内独自項目を追加できます。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/github/spec-kit)

たとえば、`plan-template.md`をオーバーライドして、各機能の計画へ次を追加できます。

```Markdown
## Project-level impact

- Affected roadmap entries:
- Project status implications:
- Dependencies on other specs:
- Status document updates required:
```

ただし、テンプレートを変えるだけでは`project-status.md`の更新を完全には自動化できません。

### 3. Presetで社内標準化する

複数プロジェクトへ同じ構成を展開するなら、**Presetが適しています**。

Presetは、Spec Kitの既存テンプレートやコマンドを変更し、組織の開発手法や標準を適用するための正式な仕組みです。複数のPresetを優先順位付きで重ねることもできます。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/github/spec-kit)

たとえば社内Presetとして、

```
company-project-context/
├── preset.yml
├── templates/
│   ├── spec-template.md
│   ├── plan-template.md
│   ├── tasks-template.md
│   └── project-status-template.md
└── commands/
    ├── plan.md
    ├── tasks.md
    └── implement.md
```

のようなものを作り、次を標準化できます。

* すべてのプロジェクトに`ROADMAP.md`を置く
* `project-status.md`の見出しを統一する
* 計画時に上位ロードマップを参照する
* 実装後に状態文書を更新する
* 未検証事項を明示する
* 機密情報を状態ファイルへ記載しない

Presetは「既存のSpec Kitの振る舞いを社内流に変更する」場合に向いています。

### 4. Extensionで`/speckit.status`を追加する

プロジェクト状態という**新しい機能**をSpec Kitへ追加するなら、Extensionが最も本格的です。

たとえば次のようなコマンドを追加します。

```
/speckit.status
```

このコマンドが、

1. `.specify/feature.json`を確認
2. `ROADMAP.md`を読む
3. 全`specs/*/spec.md`を確認
4. 全`specs/*/plan.md`を確認
5. 全`specs/*/tasks.md`のチェック状態を集計
6. テスト・Git・検証結果を確認
7. `docs/project-status.md`を生成または更新
8. 次に実行すべき作業を提示

という処理を担当します。

Spec KitのExtensionは、新しいコマンド、テンプレート、品質ゲート、外部連携、独自の開発段階を追加するための正式な拡張手段です。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/reference/extensions.html)

実際、Spec Kitの公式コミュニティカタログには、すでに以下の拡張があります。

* **Project Status**：アクティブ機能、成果物、タスク完了率、ワークフロー段階、Extension状態を表示
* **Status Report**：プロジェクト状態、機能進捗、次の行動を報告
* **Spec Roadmap**：永続的なロードマップを管理
* **Spec Orchestrator**：複数仕様を横断して状態や競合を管理
* **SpecKit Companion**：進捗、再開、ライフサイクル状態を管理

つまり、プロジェクト全体の状況管理をSpec Kitへ足すという発想は、すでにコミュニティでも実装され始めています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/community/extensions.html)

## 自動更新にはWorkflowも利用できる

Spec KitのWorkflowは、複数のコマンド、プロンプト、シェル処理、人間の確認を連結できます。また、途中停止と再開にも対応しています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/reference/workflows.html)

たとえば概念的には、

```
specify
→ clarify
→ plan
→ status update
→ tasks
→ status update
→ implement
→ test
→ converge
→ status update
→ human review
```

という社内ワークフローを作れます。

特に、

```
/speckit.implement
→ テスト
→ /speckit.converge
→ /speckit.status
```

を一連の処理にすれば、状態ファイルの更新忘れを減らせます。

社内の全プロジェクトへPreset、Extension、Workflowをまとめて導入する場合は、Spec KitのBundleとして配布することもできます。Bundleは、複数のPreset、Extension、Workflowなどをバージョン固定して一括導入する仕組みです。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/reference/overview.html)

## `constitution.md`に状態を書かない方がよい

プロジェクト全体のファイルなので、`.specify/memory/constitution.md`へ進捗も記録したくなるかもしれませんが、これは避けた方がよいです。

`constitution.md`は、

* 品質原則
* 開発原則
* セキュリティ方針
* テスト方針
* 人間承認の要否
* プロジェクトで常に守る制約

のような、比較的安定したガバナンスを置く場所です。公式の`/speckit.constitution`も、constitutionそのものの更新だけにスコープを限定しています。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/github/spec-kit/blob/main/templates/commands/constitution.md?plain=1)

したがって、

```
constitution.md = 変わりにくい原則
project-status.md = 頻繁に変わる現在状態
ROADMAP.md = 中期的な全体計画
```

と分けるべきです。

## おすすめする導入順

あなたの用途では、いきなり大規模なExtensionを作るより、次の順がよいと思います。

### 第1段階

```
ROADMAP.md
docs/project-status.md
```

を作り、`.github/copilot-instructions.md`から参照させます。

### 第2段階

プロジェクト内に`/speckit.status`相当のCopilot用プロンプトを追加し、状態更新を半自動化します。

### 第3段階

複数プロジェクトで運用して、必要な項目が固まったら、社内用Presetとしてテンプレートと参照規則を配布します。

### 第4段階

タスク集計や検証状態の自動取得が必要になったら、`/speckit.status` ExtensionとWorkflowを作ります。

この順序なら、最初から複雑な仕組みを作って実運用に合わなくなることを避けられます。

特に社内汎用化を考えるなら、最終形は次のようになります。

```
社内Spec Kit Bundle
├── company-context-preset
│   ├── 共通文書テンプレート
│   └── コマンドの参照規則
├── company-status-extension
│   └── /speckit.status
├── company-development-workflow
│   └── plan → tasks → implement → verify → status
└── company-security-preset
    └── 機密情報・外部AI利用規則
```

したがって、**Spec Kitへプロジェクト全体の現在状況を記録・共有する機能を追加することは十分可能で、現在のSpec KitではPreset、Extension、Workflow、Bundleという正式な拡張経路まで用意されています**。まずは公式が紹介している`ROADMAP.md`方式を土台にして、`docs/project-status.md`を追加するのが最も無理のない出発点です。

* * *

**You:**

spec kitを使っていて混乱する点があります。それは、途中で仕様を変えたくなったり、実装して見て出来上がったものが想定と異なっていたり事前の仕様がずれていたことに気付いたりした場合に、どうすれば良いかがわからない点です。これらの場合、speckit/specify→speckit/implementの流れを一からやり直さないといけないのでしょうか？あと、speckitが生成したファイルの一部を手動で直すことは許されるのでしょうか？例えば、specifyで生成された仕様書の一部を変更するなどです。speck kitがどのような仕組みでどのようなファイルを生成しているか十分に理解できないため、speckit/specify→speckit/implementの流れに強く縛られているように感じ、窮屈に感じます。

* * *

**ChatGPT:**

その窮屈さは自然です。Spec Kitのコマンド列は**初回作成の代表的な順序**であって、仕様変更のたびに必ず`specify → plan → tasks → implement`を最初から全部やり直す規則ではありません。

現在の公式ドキュメントでも、Spec Kitは要件変更後の`spec.md`、`plan.md`、`tasks.md`の維持方法を一つに強制しておらず、チームが運用方式を選ぶものと明記されています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+1](https://github.github.com/spec-kit/concepts/sdd.html)

## 結論

* `spec.md`、`plan.md`、`tasks.md`を**人間が直接編集して構いません**
* 変更のたびに全工程をやり直す必要はありません
* **変更した階層から下流だけを更新する**と考えると分かりやすいです
* 更新後は`/speckit.analyze`で整合性を確認します
* 実装は変更対象のタスクやフェーズだけに限定できます

公式の既存プロジェクト向けガイドでも、`spec.md`を`/speckit.clarify`または明示的な手動編集で更新し、`plan.md`と`tasks.md`も再生成または手動修正する方法が示されています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/guides/evolving-specs.html)

## 変更した場所から下流だけ直す

Spec Kitのファイルは、次の階層として見ると理解しやすくなります。

```
意図・要求
  spec.md
     ↓
技術的な実現方法
  plan.md
     ↓
具体的な作業
  tasks.md
     ↓
実装
  ソースコード・テスト
```

変更内容によって、戻る場所が異なります。

| 気付いた内容 | 最初に直す場所 | その後 |
| --- | --- | --- |
| 誤字、説明の改善 | 該当ファイル | 必要なら`analyze` |
| ユーザーに見える挙動の変更 | `spec.md` | 必要に応じて`plan.md`、`tasks.md` |
| ライブラリ、構成、アルゴリズムの変更 | `plan.md` | `tasks.md` |
| 作業順序やファイル分割の変更 | `tasks.md` | 実装 |
| 実装バグで仕様は正しい | コード・テスト | 必要ならタスク追記 |
| 大きな追加機能 | 新しいfeature spec | 通常フロー |

つまり、たとえば実装してみて「画面の操作方法が想定と違った」と分かったなら、`spec.md`まで戻ります。

一方、「要求は正しいが、SQLiteではなく別方式に変えたい」というだけなら、通常は`spec.md`を変えず、`plan.md`から修正します。

## 小さな仕様変更の場合

たとえば、実装後に次の変更をしたくなったとします。

> エラー時に処理を中断するだけでなく、原因をGUIに表示したい。

これはユーザーに見える挙動の変更なので、まず現在の`spec.md`を直接修正します。

```Markdown
## Functional Requirements

- FR-012: When analysis fails, the application shall display
  an actionable error message containing the failure category
  and the affected input file.
```

その後、AIに次のように指示します。

```
現在のfeatureのspec.mdにFR-012を追加しました。

この変更に関係する範囲だけを確認し、
plan.mdとtasks.mdを更新してください。

既存の完了済みタスクや、今回と無関係な設計判断は保持してください。
新規・変更タスクを明示した後、/speckit.analyze を実行してください。
```

そして、変更タスクだけを実装します。

```
/speckit.implement
今回追加・変更されたエラー表示関連タスクだけを実装してください。
その他の完了済みタスクは再実装しないでください。
```

`/speckit.implement`には自由形式の範囲指定を付けられ、特定フェーズやタスクだけを実装できます。完了タスクは`tasks.md`で`[X]`として管理され、後続の実行は残りから再開できます。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+1](https://github.github.com/spec-kit/reference/agentic-sdd.html)

## 実装方針だけ変わった場合

たとえば、

> 要求仕様は同じだが、解析結果の保存形式を変更したい。

この場合、`spec.md`に保存形式が利用者要求として規定されていなければ、`plan.md`から修正して構いません。

```
現在のspec.mdに記載された利用者要求は変更しません。

plan.mdについて、解析結果の内部保存方式をAからBへ変更してください。
変更理由と影響範囲を記録し、関連するtasks.mdだけを更新してください。
完了済みで影響を受けないタスクは保持してください。
```

その後、

```
/speckit.analyze
```

で`spec.md`、`plan.md`、`tasks.md`間の矛盾を確認します。`analyze`は読み取り専用で、ファイルを勝手に編集しません。問題があれば、要件問題は`spec.md`、設計問題は`plan.md`、作業分解問題は`tasks.md`へ戻って直すのが公式の考え方です。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/reference/agentic-sdd.html)

## 「出来上がったものが想定と違う」場合

ここでは、まず次のどちらかを判断します。

### 仕様どおりに実装されたが、仕様そのものが間違っていた

これは仕様変更です。

```
spec.md
→ plan.md
→ tasks.md
→ 該当実装
```

の順に整合させます。

### 仕様は正しいが、実装が仕様と違っていた

これは実装不具合です。

```
コード・テスト
→ 必要ならtasks.mdへ修正タスク追加
```

で済みます。`spec.md`を変える必要はありません。

最後に`/speckit.converge`を使うと、コードが現在のspec、plan、tasksを満たしているか確認し、不足があれば`tasks.md`へ追加タスクを追記します。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/reference/agentic-sdd.html)

## 手動編集は正式に許されている

現在の公式ガイドでは、次の両方が明記されています。

```
spec.md
  → /speckit.clarifyで更新
  → または明示的に直接編集

plan.md
  → /speckit.planで再生成
  → または直接修正

tasks.md
  → /speckit.tasksで再生成
  → または直接修正
```

[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/guides/evolving-specs.html)

したがって、Spec Kitが生成したファイルは「人間が触ってはいけない中間ファイル」ではありません。むしろ、**人間とAIが共同で維持するプロジェクト成果物**です。

ただし、コマンドを再実行すると、手動で加えた内容が置換される可能性があります。そのため公式ガイドは、変更前に作業ツリーをクリーンにするか専用ブランチを作り、生成差分をレビューすること、再生成前に重要な技術判断を明示的に引き継ぐことを推奨しています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/guides/evolving-specs.html)

## 触ってよいファイルと注意が必要なファイル

### 普通に編集してよいプロジェクト成果物

```
specs/<feature>/spec.md
specs/<feature>/plan.md
specs/<feature>/tasks.md
specs/<feature>/research.md
specs/<feature>/data-model.md
specs/<feature>/contracts/
```

これらは、プロジェクト固有の内容です。

### 直接改造に注意するSpec Kitの仕組み側

```
.specify/templates/
.specify/scripts/
.github/prompts/speckit.*.prompt.md
.github/agents/
```

こちらはSpec Kitのコマンドやテンプレート自体です。社内標準として変更する場合は、直接上書きするより、プロジェクトローカルのoverride、Preset、Extensionを使う方が安全です。Spec Kitの更新時には共有テンプレートやスクリプトが更新対象になるためです。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/github/spec-kit/blob/main/README.md?utm_source=chatgpt.com)

## Spec Kit公式の3つの運用方式

現在の公式ドキュメントでは、仕様変更後の運用を3つに分類しています。

### Living Spec

`spec.md`を契約・正本として扱います。

```
spec.mdを変更
→ plan.mdを再生成または修正
→ tasks.mdを再生成または修正
→ analyze
→ 変更部分をimplement
```

整合性を重視する方式です。ただし、planやtasksを再生成すると、そこだけに残していた重要な判断を失う危険があります。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+1](https://github.github.com/spec-kit/concepts/spec-persistence.html?utm_source=chatgpt.com)

### Flow-Back Spec

気付いた場所から修正します。

```
コードで気付いた
→ まずコードまたはtasks.mdへ反映
→ 必要ならplan.mdとspec.mdへ戻す
→ analyzeで整合確認
```

試行錯誤の多い研究開発やPoCに向いています。ただし、最終的には各ファイルを整合させないと、どれが正しいか分からなくなります。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+1](https://github.github.com/spec-kit/concepts/spec-persistence.html?utm_source=chatgpt.com)

### Flow-Forward Spec

完成したfeatureディレクトリを履歴として固定し、大きな変更は新しいfeatureとして作ります。

```
specs/001-original-feature/
specs/002-change-error-handling/
```

監査性は高いですが、小さな変更でも新規specを作ると文書が増えます。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+1](https://github.github.com/spec-kit/concepts/spec-persistence.html?utm_source=chatgpt.com)

## あなたの用途にはハイブリッドがよい

Fortranリファクタリングツールのように、

* 実装して初めて分かることが多い
* 実コードをまだ適用できない
* 合成fixtureで仮説検証している
* 誤検出から仕様を見直す可能性が高い

というプロジェクトでは、最初から完成仕様を固定するのは難しいです。

そのため、次の運用が現実的です。

### 開発中のfeature

**Flow-BackまたはLiving Spec**として扱う。

```
発見
→ 最も近いファイルを修正
→ 上流・下流を整合
→ analyze
→ 変更タスクだけ実装
```

### マイルストーン完了後

featureを一度凍結する。

```
specs/012-call-analysis/
```

その後、意味の大きく異なる変更が必要になったら新しいfeatureを作ります。

```
specs/019-call-analysis-implicit-kind-refinement/
```

つまり、

> 開発途中では柔軟に直接修正し、区切りを越える大きな変更だけ新しいSpec Kitサイクルにする

という使い方です。

## 運用ルールをconstitutionに追加するとよい

Spec Kit自体は3方式のどれを使うか強制しません。公式も、チームで選んだ方式をconstitutionやオンボーディング文書へ記録するよう勧めています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/concepts/spec-persistence.html?utm_source=chatgpt.com)

あなたのプロジェクトなら、次のような原則が適しています。

```Markdown
## Specification Evolution

- Spec Kit artifacts are editable project documents, not immutable generated files.
- During active development, use a living-spec or flow-back workflow.
- When intended behavior changes, update `spec.md`.
- When only the implementation strategy changes, update `plan.md`.
- When only work decomposition changes, update `tasks.md`.
- After changing any artifact, reconcile affected downstream artifacts and run
  `/speckit.analyze`.
- Do not regenerate unaffected artifacts unnecessarily.
- Implement only newly added or changed tasks.
- After a feature milestone is accepted, treat its directory as historical.
- Create a new feature spec only for substantial follow-up behavior changes.
```

これを採用すると、Spec Kitは「必ず最初から順番に実行するウィザード」ではなく、

> **仕様・計画・タスク・実装を整合させるための道具箱**

として扱えるようになります。

* * *

**You:**

Living Specは
speckを変更
→ plan.mdを再生成または修正
→ tasks.mdを再生成または修正
→ analyze
→ 変更部分をimplement
、Flow-Back Specは
コードで気付いた
→ まずコードまたはtasks.mdへ反映
→ 必要ならplan.mdとspec.mdへ戻す
→ analyzeで整合確認
などとありますが、このような修正を加える場合は、agentのモードとしてどれを選択すれば良いのかわかりません。copilot標準のagentモードでよいの？それともspeckitのエージェントでないと整合性が担保されない？

* * *

**ChatGPT:**

**通常のCopilot標準Agentモードで修正して構いません。**  
ただし、標準Agentモードだけで自由に編集して終わるのではなく、最後にSpec Kitの`/speckit.analyze`を実行して整合性を確認する、という使い分けが重要です。

Spec Kitの「Living Spec」「Flow-Back Spec」は、Copilotのモード設定ではなく、チームが採用する文書更新ルールです。Spec Kit公式も、どの方式を使うかはCLI設定ではなくチームの運用規約だと説明しています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/concepts/spec-persistence.html)

## まず区別すべき3つ

名称が似ているため混乱しやすいのですが、次は別物です。

### 1. Copilotの標準Agentモード

VS Code上で、ファイルの読み取り、編集、ターミナル実行などを行う汎用エージェントです。

```
Agent
```

`spec.md`、`plan.md`、`tasks.md`やコードを直接修正するときに使えます。

### 2. Copilotの標準Planエージェント

実装前に調査・計画を作るための汎用エージェントです。

```
Plan
```

これは**Spec Kitの`/speckit.plan`とは別物**です。

VS CodeのPlanエージェントが扱う計画用`plan.md`は、VS Code側がセッションの計画管理に使うメモリファイルです。一方、`/speckit.plan`は、選択中の機能について次のようなプロジェクト成果物を生成します。

```
specs/001-feature/plan.md
```

したがって、

> CopilotのPlanエージェントを選ぶ  
> ＝Spec Kitの`plan.md`を正しく更新する

ではありません。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/agents/planning?utm_source=chatgpt.com)

### 3. Spec Kitのコマンド・スキル・カスタムエージェント

次のようなものです。

```
/speckit.specify
/speckit.plan
/speckit.tasks
/speckit.analyze
/speckit.implement
/speckit.converge
```

これらは、Spec Kitが用意した手順をCopilotに実行させる仕組みです。Spec Kitは`specify init`時に、対象のAIツールに合わせて、コマンド、スキル、カスタムエージェントなどを配置します。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/github/spec-kit/blob/main/docs/reference/integrations.md)

## 「Spec Kitエージェント」を手動で選ぶ必要はあるか

通常は、**毎回AgentドロップダウンからSpec Kit専用エージェントを探して選ぶ必要はありません**。

Spec KitのCopilot統合方法には、バージョンや初期化方法によって大きく2種類あります。

### カスタムエージェント＋プロンプト方式

次の構成がある場合です。

```
.github/
├── agents/
│   ├── speckit.specify.agent.md
│   ├── speckit.plan.agent.md
│   ├── speckit.analyze.agent.md
│   └── speckit.implement.agent.md
└── prompts/
    ├── speckit.specify.prompt.md
    ├── speckit.plan.prompt.md
    ├── speckit.analyze.prompt.md
    └── speckit.implement.prompt.md
```

`.prompt.md`側には、概念的に次の指定が入っています。

```YAML
---
agent: speckit.analyze
---
```

そのため、チャットで、

```
/speckit.analyze
```

を実行すると、対応する`Speckit Analyze`エージェントへ自動的にルーティングされます。VS Codeのプロンプトファイルは、frontmatterの`agent`欄で、実行に使用する標準またはカスタムエージェントを指定できます。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/agent-customization/prompt-files?utm_source=chatgpt.com)

### Skills方式

次の構成の場合です。

```
.github/skills/
├── speckit-specify/
│   └── SKILL.md
├── speckit-plan/
│   └── SKILL.md
├── speckit-analyze/
│   └── SKILL.md
└── speckit-implement/
    └── SKILL.md
```

この場合は、Agentドロップダウンに各Spec Kitエージェントが表示されないことがあります。

その場合は、**標準Agentを選択した状態でSpec Kitのスキルを呼び出せば十分**です。

```
/speckit-analyze
```

または、VS Code上で候補表示される対応するSpec Kitコマンドを選びます。現在のSpec KitはCopilot向けにSkills形式もサポートしています。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/github/spec-kit/blob/main/docs/reference/integrations.md)

## 整合性を担保するのは「モード」ではない

ここが一番重要です。

**Spec Kit専用エージェントを選択しただけで、整合性が自動的に保証されるわけではありません。**

整合性は、次の手順によって高めます。

```
1. 適切な成果物を修正する
2. 他の成果物へ影響を反映する
3. /speckit.analyze を実行する
4. 指摘された矛盾を修正する
5. 変更対象だけを実装する
6. /speckit.converge で実装漏れを確認する
```

`/speckit.analyze`は、`spec.md`、`plan.md`、`tasks.md`の間にある矛盾、欠落、カバレッジ不足を読み取り専用で調べます。`/speckit.converge`は、実装後のコードを成果物と照合し、残作業があれば`tasks.md`へ追加します。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+1](https://github.github.com/spec-kit/guides/evolving-specs.html)

それでも、意味的な正しさまで完全に機械保証されるわけではありません。最終的には、人間が差分と仕様の意図を確認する必要があります。

## 作業別の推奨モード

| 作業 | 選択するもの | 実行方法 |
| --- | --- | --- |
| 仕様変更について相談する | AskまたはPlan | ファイル変更なしで検討 |
| `spec.md`の一部を直接直す | 標準Agent | 対象範囲を限定して編集 |
| `plan.md`の一部を直接直す | 標準Agent | 対象範囲を限定して編集 |
| `tasks.md`を追加・修正する | 標準Agent | 完了済みタスクを保持 |
| `plan.md`を大きく再生成する | Spec Kit | `/speckit.plan` |
| `tasks.md`を大きく再生成する | Spec Kit | `/speckit.tasks` |
| 成果物間の整合性を調べる | Spec Kit | `/speckit.analyze` |
| タスクに従って実装する | Spec Kit推奨 | `/speckit.implement` |
| 実装後の不足を調べる | Spec Kit | `/speckit.converge` |
| 小さなコードバグだけ直す | 標準Agent | 修正後に関連テスト実行 |

「Spec Kit」と書いた箇所は、プロジェクトがカスタムエージェント方式なら対応するSpec Kitエージェント／スラッシュコマンド、Skills方式なら標準Agent上で対応Skillを呼び出す、という意味です。

## Living Specの場合

Living Specでは、次の使い方が扱いやすいです。

### 仕様の小さな変更

標準Agentで`spec.md`を直接編集します。

```
Agentモードで、現在のspec.mdのFR-012だけを修正してください。
ほかの要件、要件ID、ユーザーストーリーは変更しないでください。
```

次に、変更の影響範囲を確認します。

```
/speckit.analyze
```

矛盾が少なければ、標準Agentで`plan.md`と`tasks.md`の該当部分だけ直します。

```
Agentモードで、FR-012の変更によって不整合になった箇所だけを
plan.mdとtasks.mdで修正してください。

完了済みタスク、無関係な設計判断、既存のタスクIDは保持してください。
```

再度、

```
/speckit.analyze
```

を実行し、問題がなければ、

```
/speckit.implement
FR-012の変更に対応する未完了タスクだけを実装してください。
```

とします。

### 仕様の大きな変更

`spec.md`を修正した後、`plan.md`や`tasks.md`の大半が無効になるなら、直接編集ではなく、

```
/speckit.plan
```

```
/speckit.tasks
```

で再生成する方がよいです。

ただし、以前の`plan.md`に重要な判断理由が残っている場合は、再生成前にADRなどへ移すか、「この判断は保持する」と明示してください。Spec Kit公式も、派生成果物を置き換える前に、重要な実装上の判断を明示的に引き継ぐよう注意しています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/guides/evolving-specs.html)

## Flow-Back Specの場合

実装中に気付いた内容を、まずコードや`tasks.md`へ反映するときは、標準Agentで構いません。

```
Agentモードで、今回判明したCALL引数判定の問題を修正してください。

まずコードとテストを修正し、その変更が
- intended behavior
- implementation strategy
- task breakdown
- code only

のどれに当たるか判定してください。
```

その後、同じAgentへ次のように依頼します。

```
今回の変更により現在のspec.md、plan.md、tasks.mdと
矛盾する箇所を確認し、必要最小限の範囲で上流成果物へ反映してください。

仕様上の挙動を変えていない場合はspec.mdを変更しないでください。
```

最後に、

```
/speckit.analyze
```

を実行します。

Spec Kit公式のFlow-Backでも、最初の編集は`spec.md`、`plan.md`、`tasks.md`、コードのどこから始めてもよいものの、その後に他の成果物を整合させ、`/speckit.analyze`を実行する流れになっています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+1](https://github.github.com/spec-kit/guides/evolving-specs.html)

## おすすめの実務ルール

あなたのように、実装して初めて分かることが多い研究開発・解析ツールでは、次のルールが扱いやすいです。

```
調査・相談だけ
    → AskまたはPlan

spec / plan / tasksを部分修正
    → 標準Agent

成果物を全面的に再生成
    → 対応する /speckit.* コマンド

コード実装
    → /speckit.implement
       または小修正なら標準Agent

成果物の変更後
    → 必ず /speckit.analyze

実装後
    → テスト + /speckit.converge
```

特に、**Copilot標準Planエージェントと`/speckit.plan`を混同しないこと**、そして**標準Agentで編集した後も`/speckit.analyze`を実行すればSpec Kitの運用に戻れること**を押さえておけば、かなり窮屈さが減ります。

Spec Kitは専用エージェントだけで完結させる閉じたシステムではなく、標準Agentによる手動修正とSpec Kitコマンドによる検証を組み合わせて使えます。

* * *

**You:**

speckitに対して、プロジェクト全体の進捗状況管理、社内AIチャットとの連携を関連づけることは可能でしょうか？

* * *

**ChatGPT:**

可能です。しかも、**Spec Kitの機能単位の仕様管理を、プロジェクト全体の状態管理と社内AI相談まで拡張する**のは、かなり自然な設計です。

ただし、Spec Kitそのものに「社内AIチャット連携」が標準搭載されているというより、次の外側の層を追加するイメージです。

```
Spec Kit
  └─ 機能ごとの仕様・計画・タスク管理

社内拡張
  ├─ プロジェクト全体の進捗集約
  ├─ 社内AIへ渡す情報の抽出・匿名化
  ├─ 社内AI回答の保存・レビュー
  └─ 採用した提案をSpec Kit成果物へ反映
```

## 推奨する全体構成

```
project/
├── .specify/
│   └── memory/
│       └── constitution.md
│
├── specs/
│   ├── 001-common-analysis/
│   │   ├── spec.md
│   │   ├── plan.md
│   │   └── tasks.md
│   └── 002-call-analysis/
│       ├── spec.md
│       ├── plan.md
│       └── tasks.md
│
├── docs/
│   ├── project-status.md
│   ├── roadmap.md
│   ├── known-issues.md
│   ├── decisions/
│   │   └── 0001-xxxx.md
│   └── ai-exchange/
│       ├── README.md
│       ├── outbox/
│       ├── inbox/
│       └── reviews/
│
├── scripts/
│   ├── collect_project_status.py
│   ├── build_ai_context.py
│   └── validate_ai_context.py
│
└── .github/
    ├── copilot-instructions.md
    ├── prompts/
    │   ├── project-status.prompt.md
    │   ├── export-ai-context.prompt.md
    │   └── review-ai-response.prompt.md
    └── skills/
        ├── project-status-management/
        └── internal-ai-handoff/
```

## 役割を明確に分ける

重要なのは、すべての文書を同じ扱いにしないことです。

| 場所 | 役割 |
| --- | --- |
| `constitution.md` | 長期的に守る原則 |
| `specs/*/spec.md` | 機能として何を実現するか |
| `specs/*/plan.md` | どう実現するか |
| `specs/*/tasks.md` | 具体的に何を実施するか |
| `project-status.md` | プロジェクト全体の現在地点 |
| `roadmap.md` | 機能間の依存関係と中期計画 |
| `decisions/` | 採用した判断と理由 |
| `ai-exchange/outbox/` | 社内AIへ渡してよい相談資料 |
| `ai-exchange/inbox/` | 社内AIから返された原文 |
| `ai-exchange/reviews/` | 回答を人間が評価した結果 |

特に、**社内AIの回答をそのまま仕様の正本にしない**ことが重要です。

```
社内AIの回答
  ↓
レビュー対象の助言
  ↓ 人間が採否判断
採用された判断
  ↓
spec / plan / tasks / ADRへ反映
```

## プロジェクト全体の進捗管理

`docs/project-status.md`は、各Spec Kit成果物を横断してまとめます。

```Markdown
# Project Status

Last updated: 2026-08-06
Based on commit: abc1234
Overall phase: CALL analysis
Active spec: specs/002-call-analysis/

## Feature status

| Feature | Status | Task progress | Verification |
|---|---|---:|---|
| COMMON analysis | Completed | 24/24 | Synthetic tests passed |
| CALL analysis | In progress | 12/20 | Partial |
| Typo correction | Planned | 0/15 | Not started |
| gfortran migration | Planned | — | Not started |

## Current goal

暗黙型を考慮したCALL引数整合性判定を実装する。

## Completed

- COMMONブロック不整合検出
- 要承認候補レポート
- 合成fixtureによる回帰試験

## In progress

- CALL側とcallee側の対応付け
- 暗黙型変数の型推定

## Blockers

- 社内実コードは利用不可
- Absoftとの数値比較は未実施

## Verification status

- Unit tests: Passed
- Synthetic integration tests: Passed
- Production-code validation: Not performed
- Numerical regression: Not performed

## Next actions

1. CALL引数型推定を実装
2. 誤検出fixtureを追加
3. `/speckit.analyze`相当の整合性確認を実施
```

ただし、単純な「完了タスク数÷全タスク数」を進捗率にするのは注意が必要です。タスクの大きさが異なるため、進捗率よりも、

* 機能状態
* 検証状態
* 未解決リスク
* 次の判断
* 実コードでの検証有無

を重視した方がよいです。

## 社内AIへ渡すコンテキスト

社内AIへは、プロジェクト全体を丸ごと渡すのではなく、相談ごとのパッケージを生成します。

```
docs/ai-exchange/outbox/
└── 2026-08-06-call-analysis-consultation/
    ├── brief.md
    ├── sanitized-status.md
    ├── technical-question.md
    └── manifest.json
```

`brief.md`には、例えば次を含めます。

```Markdown
# Consultation Brief

## Objective

CALL引数の型整合性判定において、暗黙型をどこまで考慮すべきか検討する。

## Current approach

- program unitごとに構文解析
- call-siteとcalleeの仮引数を位置対応
- 明示宣言を優先
- 暗黙型規則から補完

## Known problem

暗黙型変数についてkindが未確定となり、
型不一致候補が過剰に出力される。

## Constraints

- 実コード、製品名、固有変数名は共有しない
- 合成例のみ使用する
- 自動修正前に人間承認を要求する

## Questions

1. 型とkindを別々の確度で管理すべきか
2. 未確定kindを不一致ではなく要確認に分類すべきか
3. 推奨するデータモデルは何か
```

## 機密情報の除外はAI任せにしない

ここは特に重要です。

```
プロジェクト文書
  ↓
決定論的な抽出・許可リスト
  ↓
人間による確認
  ↓
社内AIへ投入
```

という順序にします。

避けた方がよいのは、

```
プロジェクト全体をLLMへ渡す
→ LLMに機密情報を消させる
→ その出力を別AIへ送る
```

という方法です。漏れを検証しにくいためです。

`build_ai_context.py`では、最初から共有可能な項目だけを抽出する方が安全です。

例えば次だけを許可します。

* 抽象化された目的
* 一般化された技術課題
* 合成コード
* 検証済み／未検証の区分
* 製品名を含まない制約
* AIへ質問したい事項

ソースコード、ファイルパス、社員名、製品名、実験値などは、原則として自動的に含めない設計にします。

## 社内AIの回答をSpec Kitへ戻す流れ

社内AIから次の回答を得たとします。

```
「unknown kind」と「type mismatch」は別分類にすべき
```

これを直接`spec.md`へ書き込ませるのではなく、まずレビュー記録を作ります。

```
docs/ai-exchange/reviews/
└── 2026-08-06-call-analysis-review.md
```

```Markdown
# AI Response Review

## Proposal

unknown kindをtype mismatchと分離する。

## Decision

Accepted with modification.

## Rationale

実際の型不一致と解析情報不足を区別でき、
要承認候補の過剰出力を抑制できるため。

## Affected artifacts

- `specs/002-call-analysis/spec.md`
- `specs/002-call-analysis/plan.md`
- `specs/002-call-analysis/tasks.md`
- `docs/decisions/0007-classify-unknown-kind.md`

## Required changes

- 新しい診断分類`unknown-kind`を追加
- 型不一致判定から除外
- 要確認レポートには残す
```

その後、変更の意味に応じて反映先を選びます。

| 社内AIの提案 | 反映先 |
| --- | --- |
| 利用者から見える要求の変更 | `spec.md` |
| 内部アルゴリズムや構成の変更 | `plan.md` |
| 作業追加・順序変更 | `tasks.md` |
| 長期的な設計判断 | ADR |
| 単なる参考意見 | `ai-exchange/reviews/`だけ |

反映後に、成果物間の整合性確認を実行し、変更部分だけ実装します。

```
社内AI相談
→ review
→ 人間の採否判断
→ spec / plan / tasks / ADR更新
→ analyze
→ 変更タスクだけimplement
→ project-status更新
```

## Copilotとの役割分担

標準Copilot Agentでも実行できますが、役割を次のように分けると安定します。

### 決定論的スクリプトに任せるもの

* `tasks.md`のチェック状態集計
* テスト結果の取得
* GitコミットIDの記録
* 許可された項目だけの抽出
* 禁止語・パス・拡張子の検査
* コンテキストパッケージの生成

### Copilot Agentに任せるもの

* 状況の要約
* 複数spec間の関係整理
* 社内AIへの質問文作成
* 社内AI回答の論点整理
* 影響を受ける成果物の候補提示
* `spec.md`、`plan.md`、`tasks.md`の変更案

### 人間が行うもの

* 社内AIへ送る前の確認
* AI回答の採否判断
* 仕様変更の承認
* 機密性の最終確認
* 実コード適用可否の判断

専用のSpec Kitエージェントでなければできないわけではありません。整合性を高めるのは、専用エージェントの名前よりも、

* 参照する正本の明確化
* 決定論的な検査
* 差分レビュー
* 人間承認
* 最後の整合性確認

です。

## 導入は3段階がよい

### 第1段階：手動運用

まず次だけ追加します。

```
docs/project-status.md
docs/roadmap.md
docs/ai-exchange/
```

Copilotに要約と相談資料を作らせ、人間が社内AIへコピーし、回答を戻します。

### 第2段階：半自動化

次のスクリプトとプロンプトを追加します。

```
scripts/collect_project_status.py
scripts/build_ai_context.py
.github/prompts/project-status.prompt.md
.github/prompts/export-ai-context.prompt.md
```

機密情報確認と送受信はまだ人間が行います。

### 第3段階：APIまたはMCP連携

社内AIが正式なAPIやMCPを提供し、社内セキュリティ上も許可されている場合だけ、

```
状態収集
→ コンテキスト生成
→ 承認
→ 社内AI送信
→ 回答保存
→ レビュー案生成
```

まで自動化します。

最初から完全自動連携にすると、どの情報を誰が送ったか、AI回答がどの仕様変更を引き起こしたかが追跡しにくくなります。まずはファイルを介した半自動運用が適しています。

## `.github/copilot-instructions.md`には何を書くか

概念的には次のような指示を追加します。

```Markdown
## Project-wide context

- Read `docs/project-status.md` before planning or implementing work.
- Use `docs/roadmap.md` to identify dependencies between feature specs.
- Treat artifacts under `specs/` as the source of truth for feature requirements,
  plans, and tasks.
- Treat responses under `docs/ai-exchange/inbox/` as unreviewed advice.
- Do not modify specifications solely on the basis of an AI response.
- Apply only proposals accepted in `docs/ai-exchange/reviews/`.
- After changing a specification, reconcile the corresponding plan and tasks.
- Update `docs/project-status.md` after a milestone or material status change.
- Never include production source code or confidential identifiers in an
  AI consultation package.
```

この構成にすると、Spec Kitは、

> 機能単位で仕様から実装までを管理する仕組み

から、

> プロジェクト全体の状態を共有し、社内AIの助言を統制された形で取り込みながら仕様を進化させる仕組み

へ拡張できます。

なお、Spec Kitの最新バージョンにおける正式な拡張API名や組み込みコマンドの有無は、今回Web検索機能の不調で確認できませんでした。そのため、上記は特定バージョンに依存しない、Copilot・Markdown・スクリプトで実現できる設計として示しています。

* * *

**You:**

docs/decisionsってどんどん増えていきませんか？specsとの違いをしっかり理解していないのですが、specsを保存・更新するだけでは不十分なの？

* * *

**ChatGPT:**

はい、`docs/decisions/`は運用の仕方によってはどんどん増えます。  
ただし、それは**すべての仕様変更や実装判断をADRとして記録した場合**です。本来、`docs/decisions/`に残すのは、少数の重要かつ長期的な判断だけです。

そして、あなたのプロジェクトでは、まずは**Spec Kitの`spec.md`、`plan.md`、`tasks.md`を正しく保存・更新するだけでも、多くの場合は十分**です。`docs/decisions/`は必須ではありません。

## specsとdecisionsの根本的な違い

端的に分けると、次の違いです。

> `specs/`は「現在、何をどのように作るか」  
> `decisions/`は「なぜその選択をしたのか」

Spec Kitの標準成果物は、次の役割を持ちます。

| 成果物 | 主に答える質問 |
| --- | --- |
| `spec.md` | 何を実現するのか |
| `plan.md` | どのような技術構成で実現するのか |
| `tasks.md` | 具体的に何を作業するのか |
| コード・テスト | 実際にどう実現されているか |
| ADR | なぜ別案ではなく、この判断を選んだのか |

Spec Kitは、`spec.md → plan.md → tasks.md → implement`という成果物の連鎖を中心にしており、`/speckit.analyze`もこの3文書の矛盾や欠落を検査します。ADRはSpec Kitのコアフローに必須の成果物ではありません。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+2![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+2](https://github.github.com/spec-kit/index.html?utm_source=chatgpt.com)

## 具体例

たとえばFortranリファクタリングツールで、次の仕様があるとします。

### `spec.md`

```Markdown
## Functional Requirement

CALL引数の解析情報が不足している場合は、
型不一致とは断定せず「要確認」として報告する。
```

これは、ツールが**何をするべきか**です。

### `plan.md`

```Markdown
## Type resolution strategy

引数の解析結果は次の3状態で管理する。

- resolved
- inconsistent
- unknown

`unknown`は型不一致診断から除外し、
要確認レポートに出力する。
```

これは、その仕様を**どのように実現するか**です。

### `tasks.md`

```Markdown
- [ ] 型解決結果を表す列挙型を追加する
- [ ] unknown判定を実装する
- [ ] 要確認レポートへ出力する
- [ ] 回帰fixtureを追加する
```

これは、**何を作業するか**です。

この程度なら、`spec.md`、`plan.md`、`tasks.md`だけで十分です。別途ADRを作る必要はありません。

## ADRが必要になる場合

一方、次のような判断をしたとします。

> Fortran解析には正規表現中心の独自解析ではなく、fparser2の構文木を基盤として使う。

この判断には、複数の有力な選択肢があります。

* 正規表現
* fparser2
* コンパイラの解析結果
* 商用解析ツール
* 独自パーサー

さらに、この選択はCOMMON解析、CALL解析、暗黙型解析、Python移行など、複数機能へ長期間影響します。将来、別の開発者が、

> なぜ正規表現だけで実装しなかったのか  
> なぜ別のパーサーへ移行しないのか

と疑問を持つ可能性があります。

このような場合にだけADRを作ります。

```Markdown
# ADR-0001: Fortran構文解析の基盤にfparser2を採用する

## Status

Accepted

## Context

正規表現ベースのCOMMON解析では、
継続行、複雑な宣言、program unit境界の扱いに限界がある。

## Considered options

- 正規表現中心の解析
- fparser2
- 独自パーサー
- 商用解析ツール

## Decision

fparser2のASTを主要な解析基盤として使用する。
解析不能箇所では限定的なフォールバックを許可する。

## Consequences

- Fortran構文を構造的に扱える
- fparser2の対応範囲に依存する
- パーサー失敗時の診断機能が必要になる
```

ADRは一般に、タイトル、状態、背景、決定、結果・影響などを記録し、重要な選択とその理由を保持するためのものです。すべての実装詳細を記録するものではありません。[![](https://www.google.com/s2/favicons?domain=https://adr.github.io&sz=128)Architectural Decision Records+2![](https://www.google.com/s2/favicons?domain=https://adr.github.io&sz=128)Architectural Decision Records+2](https://adr.github.io/?utm_source=chatgpt.com)

## specsだけでは不足し得る理由

`spec.md`や`plan.md`は、通常、現在の正しい状態へ更新されていきます。

たとえば最初は、

```Markdown
正規表現でFortranコードを解析する。
```

と書かれていたものを、後で、

```Markdown
fparser2を使ってFortranコードを解析する。
```

へ書き換えたとします。

現在の計画は分かりますが、次は失われます。

* 正規表現方式を検討した事実
* 正規表現方式で何が問題だったのか
* fparser2を選んだ判断基準
* fparser2の欠点を承知して採用したのか
* 将来見直す条件

Gitの履歴を調べれば一部は分かりますが、複数コミットやチャットに判断理由が分散しがちです。ADRは、その「なぜ」を1か所に残します。

つまり、

```
specs = 現在の正本
decisions = 重要な判断の履歴
```

という関係です。

## ただし、plan.mdにも判断理由は書ける

ここが重要です。

**すべての判断理由をADRへ分離する必要はありません。**  
一つのfeature内だけで完結する判断なら、`plan.md`や`research.md`に書けば十分です。

Spec Kitでは、機能単位の設計内容は`plan.md`などへ記録されます。そのため、次のように分けられます。

### `plan.md`で十分な判断

* この画面だけで使用するGUIコンポーネント
* 一つの解析処理内で使うデータ構造
* 一時ファイルの名前
* そのfeature固有のアルゴリズム
* テストfixtureの具体的な分割方法

### ADRを検討する判断

* 複数featureに影響する
* 後から変更するのが高コスト
* データ形式や公開APIを固定する
* セキュリティ・安全性に関係する
* 複数の有力な選択肢を比較した
* コードを見ただけでは採用理由が分からない
* 将来、同じ議論が繰り返されそう
* 組織として説明責任がある

MicrosoftのADRガイドも、アーキテクチャ上重要な要求に対する主要な判断、却下した代替案、判断理由を残すものとしてADRを位置付けています。判断が変わった場合は旧ADRを書き換えて履歴を消すのではなく、新しいADRで置き換え関係を記録します。[![](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com&sz=128)Microsoft Learn+1](https://learn.microsoft.com/et-ee/azure/well-architected/architect-role/architecture-decision-record?utm_source=chatgpt.com)

## ADRを作らないもの

次のようなものまでADRにすると、確実に増えすぎます。

* 誤字修正
* バグ修正
* 小規模なリファクタリング
* タスクの順序変更
* 変数名の変更
* テストケースの追加
* 画面表示文言の変更
* 一つのfeature内だけの細かい実装方法
* 現在の進捗
* 未完了タスク
* 一時的な検討メモ
* AIが提示しただけで採用していない提案

これらはそれぞれ、コード、テスト、`tasks.md`、`plan.md`、`project-status.md`などに記録します。

## あなたのプロジェクトでの分類例

| 内容 | 主な記録先 |
| --- | --- |
| CALL引数の型不一致を検出する | `spec.md` |
| unknown kindを要確認に分類する | `spec.md`と`plan.md` |
| unknown kind判定用のクラスを作る | `plan.md`と`tasks.md` |
| 誤検出fixtureを追加する | `tasks.md` |
| 現在CALL解析を実装中 | `project-status.md` |
| 実コードでは未検証 | `project-status.md`と該当spec |
| fparser2を解析基盤に採用する | ADR候補 |
| 自動書換え前に人間承認を必須にする | `constitution.md`、必要ならADR |
| 合成fixtureのみで当面検証する | `constitution.md`または検証方針 |
| Python移行はgfortran移植後に行う | roadmap、重要ならADR |
| 社内AIの提案を正本へ直接反映しない | `constitution.md`またはAI連携ルール |

「常に守る原則」は、ADRより`.specify/memory/constitution.md`に書く方が適することもあります。ADRは原則そのものではなく、原則や制約を踏まえて行った特定の重要判断を記録するものです。

## `docs/decisions/`を増やしすぎない判定ルール

次の質問のうち、**2つ以上が明確に当てはまる場合だけADRを作る**という社内ルールが実用的です。

1. 複数のspecまたはコンポーネントへ影響するか
2. 後から変更するのが高コストか
3. 複数の妥当な選択肢を比較したか
4. なぜその方式を選んだか、コードだけでは分からないか
5. 数か月後にも判断理由を参照する可能性があるか
6. 安全性、互換性、データ形式、外部依存に関係するか

一つしか該当しないなら、通常は`plan.md`や`research.md`で十分です。

## ADRが増えること自体は問題か

適切に選別した結果、長期プロジェクトで数十件になるのは異常ではありません。ADRは1件を短く保ち、状態を明記します。

```
Proposed
Accepted
Deprecated
Superseded
Rejected
```

ただし、あなたの現在の開発規模で、短期間に100件近く増えるなら、明らかに記録対象が細かすぎます。

目安としては、

* featureごとに必ずADRを作らない
* sprintや実装段階ごとにも作らない
* 重要な横断判断が発生したときだけ作る
* feature固有の判断は`plan.md`へ残す

という運用がよいです。

## あなたには最小構成から始めるのがよい

現段階では、次の構成で十分だと思います。

```
.specify/
└── memory/
    └── constitution.md

specs/
└── <feature>/
    ├── spec.md
    ├── plan.md
    ├── tasks.md
    └── research.md

docs/
├── project-status.md
└── roadmap.md
```

`docs/decisions/`は最初から必須にせず、**複数のspecをまたぐ重要判断が初めて発生した時点で追加**すればよいです。

さらに簡素化したければ、最初は1ファイルでも構いません。

```
docs/key-decisions.md
```

ここに本当に重要な判断だけを数件記録し、件数が増えて検索しにくくなった段階で、個別ADRへ分割します。

したがって、あなたの懸念に対する答えは次のとおりです。

> **Spec Kitの`spec.md`、`plan.md`、`tasks.md`を適切に維持すれば、通常の機能開発には十分です。`docs/decisions/`は必須ではなく、現在のspecやplanだけでは失われてしまう、長期的で横断的な「なぜ」を残したい場合にだけ追加します。**

無理にADRを導入すると文書管理の負担が増えるため、あなたのプロジェクトでは、当面はSpec Kit成果物を中心にし、重要判断だけ`constitution.md`または小さな`key-decisions.md`へ残す運用が適しています。

* * *

**You:**

社内AIでパッケージやフォルダ構成を提案されたとして、speckitのワークフローにどうやってそれを反映させれば良いのでしょうか？speckitの仕様をよくわかっていないのですが、specifyの段階で使用パッケージなども指定できるんでしたっけ？それともspecifyは大まかな仕様を決めるだけで、具体的な実装方法を決めるplanの段階からしかパッケージの追加を反映できない？

* * *

**ChatGPT:**

はい。**原則として、使用パッケージや内部フォルダ構成は`specify`ではなく`plan`段階へ反映します。**

Spec Kit公式も、`/speckit.specify`は「何を、なぜ作るか」「利用者に見える挙動と目標」を扱い、技術スタックは`/speckit.plan`に置くと明確に分けています。`/speckit.plan`は、技術スタック、アーキテクチャ、技術的制約を与える段階です。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+1](https://github.github.com/spec-kit/reference/agentic-sdd.html)

## 基本的な分類

社内AIから提案された内容は、そのまま一括してSpec Kitへ入れるのではなく、内容ごとに振り分けます。

| 社内AIの提案内容 | 主な反映先 |
| --- | --- |
| ユーザーが利用できる機能 | `spec.md` |
| ユーザーに見える動作や制約 | `spec.md` |
| 使用言語・フレームワーク | `plan.md` |
| 使用パッケージ・バージョン | `plan.md` |
| ソースコードのフォルダ構成 | `plan.md` |
| アーキテクチャやモジュール分割 | `plan.md` |
| パッケージ調査・比較結果 | `research.md` |
| 実装に必要な作業 | `tasks.md` |
| 全機能に共通する社内ルール | `constitution.md` |
| 実装後のコード配置 | 実コード＋必要に応じて`plan.md`更新 |

Spec Kitの標準`plan.md`テンプレートにも、`Primary Dependencies`と`Project Structure`が明示的に用意されています。したがって、パッケージとフォルダ構成は公式テンプレート上でも`plan`の対象です。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/github/spec-kit/blob/main/templates/plan-template.md)

## `specify`でパッケージを指定してはいけないのか

絶対に書いてはいけないわけではありませんが、通常は避けます。

例えば、次の要求に対して、

> `.nc`ファイルを読み込み、測定面を可視化してPNGとCSVへ保存できること

これは`spec.md`に書きます。

一方、

> GUIにはPySide6を使用し、NetCDF読み込みにはxarrayを使用する

これは`plan.md`です。

なぜ分けるかというと、PySide6をTkinterへ変更しても、利用者に提供する機能が同じなら、仕様そのものは変わっていないからです。

```
spec.md
何を実現するか
    ↓
plan.md
どの技術で実現するか
    ↓
tasks.md
何を作業するか
```

### 例外：パッケージ自体が必須条件の場合

次のような場合は、技術選択が単なる実装案ではなくプロジェクトの制約です。

* 社内承認済みパッケージしか使用できない
* オープンソースのみ使用する
* GPL系ライセンスは禁止
* オフラインでインストール可能であること
* Python 3.11で動作すること
* fparser2を解析基盤として使用することが決定済み
* Windows上で配布可能でなければならない

この場合は、要求・制約として`spec.md`または`constitution.md`にも記録し、具体的な採用方法を`plan.md`へ展開します。

たとえば次のように分けます。

**`spec.md`**

```Markdown
### Constraints

- The application must operate in an offline environment.
- All runtime dependencies must be distributable under the company’s
  approved license policy.
```

**`plan.md`**

```Markdown
### Primary Dependencies

- fparser2: Fortran syntax analysis
- pytest: automated testing
- PyInstaller: Windows distribution

### Dependency constraints

- All wheels must be obtainable for offline installation.
- GPL-licensed runtime dependencies are not permitted.
```

つまり、**制約はspec、具体的な解決策はplan**です。

## 社内AIの提案を反映する実際の流れ

たとえば、社内AIが次の提案をしたとします。

> CLI、解析ロジック、レポート生成を分離し、  
> `src/domain/`、`src/application/`、`src/adapters/`に分ける。  
> CLIにはTyperを使用する。

この場合は、まず提案を次のように分類します。

```
Typerを使用
    → 技術依存関係
    → plan.md

domain/application/adaptersに分ける
    → 内部アーキテクチャとフォルダ構成
    → plan.md

CLIを提供する
    → ユーザーに見える機能
    → spec.mdに既にあるか確認

Typer導入作業、ファイル移動、テスト修正
    → tasks.md
```

### 1. まず現在の仕様と矛盾しないか確認する

社内AIの提案によって、利用者に見える挙動が変わるかを確認します。

たとえば現在の`spec.md`がGUIを必須としているのに、社内AIがCLI専用構成を提案したなら、単なる技術提案ではありません。仕様との矛盾です。

この場合は、まず人間が次を決めます。

* GUIを残しつつCLIも追加する
* GUIを廃止する
* 社内AIの提案を不採用にする

仕様変更を受け入れるなら、先に`spec.md`を更新します。

### 2. パッケージとフォルダ構成を`plan.md`へ反映する

小規模な修正なら、Copilot標準Agentモードで直接編集して構いません。

```
社内AIから次の技術提案を受けました。

- CLIにTyperを使用する
- 解析ロジックをsrc/domainへ分離する
- 入出力処理をsrc/adaptersへ分離する

現在のspec.mdは変更しないでください。
提案が現在の要求・constitution・既存コードと整合するか確認し、
採用可能な内容だけをplan.mdの
- Technical Context
- Primary Dependencies
- Project Structure
- Summary
へ反映してください。

既存の重要な技術判断を削除せず、変更差分を説明してください。
```

大規模に再設計するなら、`/speckit.plan`へ社内AIの提案を入力します。

```
/speckit.plan

現在のspec.mdを要求の正本としてください。

社内AIから以下の技術案が提案されました。
- CLIにはTyperを使用
- domain、application、adaptersの3層に分離
- 既存のpytestテストは維持
- オフライン環境で導入可能な依存関係に限定

この案を無条件には採用せず、既存コード、constitution、
現在のplan.mdとの整合性を確認してください。
採否と理由をresearch.mdへ記録し、
採用した構成だけをplan.mdへ反映してください。
```

`/speckit.plan`は技術コンテキストを埋め、設計成果物を生成する工程なので、この入力方法がSpec Kit本来の使い方に近いです。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+1](https://github.github.com/spec-kit/reference/agentic-sdd.html)

## `research.md`も活用できる

パッケージ案が確定していない場合は、最初から`plan.md`に「採用」と書くのではなく、比較検討を`research.md`へ記録するとよいです。

```Markdown
# Research: CLI Framework

## Options

### Typer

- Advantages: 型ヒントとの親和性、ヘルプ生成
- Risks: 追加依存関係が増える
- Offline availability: 要確認

### argparse

- Advantages: 標準ライブラリ、追加依存なし
- Risks: 実装量が増える

## Decision

現段階ではargparseを採用する。

## Rationale

対象PCへのオフライン配布と依存関係最小化を優先する。
```

そして`plan.md`には決定結果だけを書きます。

```Markdown
Primary Dependencies:

- No external CLI framework
- Use Python standard-library argparse
```

これなら、社内AIの提案を検討した事実と、不採用理由も残せます。

## `tasks.md`へ落とす

`plan.md`を更新したら、次は`tasks.md`です。

```
/speckit.tasks

更新済みplan.mdを基にtasks.mdを更新してください。
完了済みかつ今回の変更に影響されないタスクは保持してください。

以下を明示してください。
- 新規パッケージの追加
- 依存関係設定の更新
- フォルダ作成
- 既存ファイルの移動
- import修正
- テスト修正
- 配布・オフライン導入確認
```

Spec Kitでは`tasks.md`は、設計成果物をもとに依存順で生成されます。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/reference/agentic-sdd.html)

ただし、既存の実装途中プロジェクトでは、全面再生成によって完了状態や手作業の追記が失われる可能性があります。その場合は標準Agentで必要部分だけ追記する方が安全です。

## 最後に整合性を確認する

変更後は次を実行します。

```
/speckit.analyze
```

`/speckit.analyze`は、`spec.md`、`plan.md`、`tasks.md`間の矛盾、欠落、カバレッジ不足を読み取り専用で確認します。問題が見つかった場合、要求の問題ならspec、技術設計の問題ならplan、作業分解の問題ならtasksへ戻します。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/reference/agentic-sdd.html)

最終的な流れは次のようになります。

```
社内AIから提案
    ↓
人間が提案の妥当性と機密性を確認
    ↓
利用者の挙動が変わるか判定
    ├─ 変わる → spec.mdを更新
    └─ 変わらない → spec.mdは維持
    ↓
パッケージ・構成をresearch.mdで比較
    ↓
採用案をplan.mdへ反映
    ↓
tasks.mdへ変更作業を追加
    ↓
/speckit.analyze
    ↓
変更タスクだけimplement
```

## プロジェクト全体へ影響する提案の場合

社内AIが一つのfeatureだけでなく、プロジェクト全体について、

> 全解析器で共通のASTモデルを使用する  
> すべてのCLIを同じフレームワークで統一する  
> `src/core`と`src/plugins`に分割する

と提案した場合は、一つのfeatureの`plan.md`だけに書くと不十分です。

この場合は、

1. プロジェクト共通方針として承認する
2. `constitution.md`またはプロジェクト全体のアーキテクチャ文書へ記録する
3. 影響する各featureの`plan.md`を更新する
4. 各`tasks.md`へ移行作業を追加する
5. `/speckit.analyze`をfeatureごとに実施する

という流れになります。

ただし、特定パッケージを`constitution.md`へ書くと将来変更しにくくなるので、原則としてconstitutionには、

```Markdown
- Use approved open-source dependencies only.
- Keep parsing logic independent from CLI and GUI frameworks.
- All dependencies must support offline installation.
```

のような安定した原則を書き、具体的なパッケージ名は各`plan.md`へ置く方が扱いやすいです。

したがって、今回の疑問は次のように整理できます。

> **通常のパッケージ追加や内部フォルダ構成は`/speckit.plan`から反映する。**  
> **その提案によってユーザーに見える機能や必須制約まで変わる場合だけ、先に`spec.md`を修正する。**

`specify`を毎回やり直す必要はなく、社内AIの提案を技術案として評価し、`research.md → plan.md → tasks.md → analyze → implement`と流すのが基本です。

* * *

**You:**

research.mdファイルって何でしたっけ？

* * *

**ChatGPT:**

`research.md`は、**`/speckit.plan`の途中で生成される、機能ごとの技術調査・選定理由を記録するファイル**です。

論文のような「研究報告書」ではなく、簡単にいうと、

> **実装方法を決める前に、候補を比較し、何を採用したか・なぜ採用したかを残すメモ**

です。

## Spec Kit内での位置付け

通常は、対象機能のフォルダ内に生成されます。

```
specs/
└── 001-feature-name/
    ├── spec.md
    ├── plan.md
    ├── research.md
    ├── data-model.md
    ├── quickstart.md
    ├── contracts/
    └── tasks.md
```

現在のSpec Kitでは、`/speckit.plan`のPhase 0で`research.md`を作り、技術コンテキストに残った`NEEDS CLARIFICATION`、依存パッケージ、外部連携、技術的なベストプラクティスなどを調査します。その結果を、基本的に「Decision」「Rationale」「Alternatives considered」の形でまとめます。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md?plain=1)

## ほかのファイルとの違い

| ファイル | 答えること |
| --- | --- |
| `spec.md` | 何を、なぜ作るか |
| `research.md` | 技術候補をどう比較し、なぜ選んだか |
| `plan.md` | 最終的に、どの技術・構成で作るか |
| `tasks.md` | 具体的に何を作業するか |

流れとしては次の関係です。

```
spec.md
何を実現するか
    ↓
research.md
実装候補を比較・調査する
    ↓
plan.md
採用する技術構成を確定する
    ↓
tasks.md
実装作業へ分解する
```

Spec Kit公式でも、`specify`では技術スタックより「何を・なぜ」に集中し、`plan`で技術スタックやアーキテクチャを決めるよう区別しています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/quickstart.html)

## 具体例

社内AIから次の提案を受けたとします。

> CLIにはTyperを使うべきである。

この段階では、まだ採用決定とは限りません。そこで`research.md`に比較結果を残します。

```Markdown
# Research

## CLI framework

### Decision

Python標準ライブラリのargparseを採用する。

### Rationale

対象環境ではオフライン配布と依存関係の最小化を優先する。
今回のCLI規模では、Typerの追加機能よりも依存を増やさない利点が大きい。

### Alternatives considered

- Typer
  - 型ヒントとの親和性が高い
  - ヘルプを生成しやすい
  - 外部依存が増える

- Click
  - 高機能だが、今回の規模には過剰

- argparse
  - 標準ライブラリ
  - 外部依存が不要
```

そして`plan.md`には、調査過程を全部繰り返さず、採用結果を書きます。

```Markdown
## Technical Context

Language/Version: Python 3.11
Primary Dependencies: fparser2, pytest
CLI Framework: Python standard-library argparse
```

公式の`plan.md`テンプレートにも、言語、主要依存関係、ストレージ、テスト方法、対象プラットフォーム、制約、プロジェクト構成などを記録する欄があります。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/github/spec-kit/blob/main/templates/plan-template.md)

## 必ず詳しく書く必要はない

すでに技術選択が確定していて、比較する必要がほとんどなければ、`research.md`は短くても構いません。

例えば、

```Markdown
## Fortran parser

### Decision

既存方針に従いfparser2を使用する。

### Rationale

プロジェクト全体ですでに解析基盤として採用済みであり、
今回の機能だけ別パーサーを導入する理由がない。

### Alternatives considered

なし。既存アーキテクチャを継続する。
```

程度で十分です。

したがって、`research.md`はすべての技術情報を詰め込む必須の大文書ではありません。**未確定だった技術事項と、その選定理由を必要な範囲で残す補助成果物**です。

## 社内AIとの関係

社内AIからパッケージやフォルダ構成を提案された場合は、

```
社内AIの提案
    ↓
research.mdで比較・評価
    ↓
採用案だけplan.mdへ反映
    ↓
tasks.mdへ実装作業を追加
```

という使い方ができます。

ただし、社内AIの提案をすでに人間が検討・承認しており、比較検討を残す必要もなければ、`/speckit.plan`への入力として直接与え、`plan.md`へ反映しても構いません。

要するに、`research.md`は、

> **仕様書ではなく、plan.mdの技術判断を支える検討記録**

です。

* * *

**You:**

社内AIの回答をspecifyに入力することになると思いますが、その場合の入力プロンプトとしてどのようなものが良いか、テンプレートはあるでしょうか？

* * *

**ChatGPT:**

テンプレートは作れます。ただし、**社内AIの回答全文をそのまま`/speckit.specify`へ貼り付ける方法はおすすめしません**。

社内AIの回答には、通常次の内容が混在するためです。

* 追加すべき機能
* ユーザーに見える振る舞い
* 使用パッケージ
* フォルダ構成
* アーキテクチャ
* 実装タスク
* 単なる参考意見

Spec Kit公式では、`/speckit.specify`は「何を、なぜ作るか」を扱い、技術スタック、API、コード構成などの「どう作るか」は`/speckit.plan`へ渡すよう明確に分けています。生成される`spec.md`も、ユーザーストーリー、受入条件、機能要件、成功基準、前提条件を中心とした構成です。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+3![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+3![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+3](https://github.github.com/spec-kit/quickstart.html)

## 推奨する流れ

```
社内AIの回答
    ↓
人間が採用・不採用を判断
    ↓
Copilot標準Agentで内容を分類
    ├─ 利用者に見える要求 → spec.md
    ├─ 技術案・パッケージ → research.md / plan.md
    ├─ 実装作業 → tasks.md
    ├─ 全体原則 → constitution.md
    └─ 参考意見のみ → 相談記録
```

したがって、まず「社内AI回答の振り分け」を行い、その結果のうち**採用した要求部分だけ**を`/speckit.specify`へ入力します。

## 1. 社内AI回答の振り分けプロンプト

これは`/speckit.specify`ではなく、Copilotの標準Agentモードで使う前処理用です。

以下は、社内AIから得たプロジェクト改善案です。

この回答をそのまま実装したり、Spec Kit成果物へ反映したりせず、まず内容を分類してください。

## 参照する正本

* `.specify/memory/constitution.md`
* `docs/project-status.md`
* `docs/roadmap.md`
* 現在対象となっている`specs/<feature>/spec.md`
* 現在対象となっている`specs/<feature>/plan.md`
* 現在対象となっている`specs/<feature>/tasks.md`

## 分類基準

各提案を次のいずれかに分類してください。

1. `spec`
    * 利用者に提供する機能
    * 利用者から見える動作
    * 必須の制約
    * 対象範囲・対象外範囲
    * 受入条件
2. `research`
    * 未確定のパッケージ候補
    * 技術比較
    * 採用可否の調査が必要な案
    * 代替案とトレードオフ
3. `plan`
    * 使用言語、ライブラリ、フレームワーク
    * アーキテクチャ
    * モジュール・パッケージ・フォルダ構成
    * 内部データモデル
    * 技術的制約
4. `tasks`
    * ファイル作成
    * パッケージ追加
    * コード変更
    * テスト追加
    * 移行作業
5. `constitution`
    * 複数機能で長期間守るべき原則
    * セキュリティ、品質、承認、機密性に関する非交渉ルール
6. `status`
    * 現在の進捗、ブロッカー、未検証事項
7. `advice-only`
    * 参考意見
    * 採用判断が済んでいない提案
    * 現在の要求や制約と矛盾する案

## 出力形式

| ID | 提案の要約 | 分類 | 現在の成果物との整合性 | 推奨する採否 | 反映先 | 理由 |
| --- | --- | --- | --- | --- | --- | --- |

続けて、以下を出力してください。

### 仕様へ反映する候補

利用者視点・技術非依存の要求文へ書き換えてください。

### 技術計画へ反映する候補

採用が妥当な技術案と、追加調査が必要な技術案を分けてください。

### 確認が必要な事項

人間の判断なしに確定してはいけない事項だけを列挙してください。

## 制約

* 社内AIの提案は未承認の助言として扱ってください。
* 現在の`spec.md`、constitution、プロジェクト制約を優先してください。
* 社内AIが提案したパッケージやフォルダ構成を`spec`に含めないでください。
* 現在存在しない要求を推測で追加しないでください。
* 実コードで未検証の事項を、検証済みとして扱わないでください。

## 社内AIの回答

ここに社内AIの回答を貼り付ける。

この処理を挟むことで、「社内AIがPySide6を勧めたので`spec.md`にもPySide6と書いてしまう」といった混在を防げます。

## 2. 新しい機能を`/speckit.specify`へ入力するテンプレート

振り分けた結果、新しい機能としてSpec Kitサイクルを開始する場合のテンプレートです。

/speckit.specify

以下の承認済み内容を基に、新しい機能仕様を作成してください。

## 機能の目的

[この機能によって解決したい問題と、その理由]

## 対象利用者

[誰がこの機能を利用するか]

## 現在の問題

[現在どのような不便、誤り、リスクがあるか]

## 必要な利用者向け機能

* [利用者ができるようになること1]
* [利用者ができるようになること2]
* [システムが利用者に対して保証する振る舞い]

## 代表的な利用シナリオ

1. Given: [初期状態]  
    When: [利用者の操作]  
    Then: [期待される結果]
2. Given: [異常・境界条件]  
    When: [操作または事象]  
    Then: [期待される処理]

## 対象範囲

* [今回含めるもの]
* [今回含めるもの]

## 対象外

* [今回実装しないもの]
* [将来機能として扱うもの]

## 必須制約

* [利用者・運用上の制約]
* [セキュリティ、機密性、互換性などの必須条件]
* [検証環境上の制約]

## 成功条件

* [測定可能な完了条件]
* [利用者が確認できる結果]
* [必要な精度、件数、処理時間など]

## 前提条件

* [既存機能や環境に関する前提]
* [現時点で未検証であること]

## 社内AI提案の扱い

この仕様に含めるのは、人間が承認した利用者向け要求だけです。  
社内AIが提案したパッケージ、フレームワーク、クラス構成、フォルダ構成、アルゴリズムは仕様へ含めないでください。それらは後続の`/speckit.plan`で検討します。

不明点について新しい要求を推測で追加せず、機能範囲や利用者体験に重大な影響があるものだけを明確化事項として示してください。

`/speckit.specify`は自由記述を受け取りますが、このように情報を整理して渡すと、公式テンプレートのユーザーストーリー、受入シナリオ、機能要件、成功基準、前提条件へ変換しやすくなります。Spec Kit側も、要件をテスト可能かつ技術非依存にし、実装詳細を仕様へ混ぜないことを検査します。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+2![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+2](https://github.com/github/spec-kit/blob/main/templates/commands/specify.md?plain=1)

## 3. 既存仕様を変更する場合のテンプレート

すでに対象の`spec.md`がある場合は、新しいfeatureを作る`/speckit.specify`より、**標準Agentで既存の`spec.md`を限定的に編集する方が安全な場合があります**。

公式のLiving Spec運用でも、意図した振る舞いが変わる場合は既存`spec.md`を明示的に編集し、その後`plan.md`と`tasks.md`を整合させ、`/speckit.analyze`を実行する流れが示されています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/guides/evolving-specs.html)

現在の`specs/<feature>/spec.md`を、以下の承認済み仕様変更に基づいて更新してください。

## 変更の背景

[実装結果、利用者確認、試験結果などから判明した問題]

## 現在の仕様

[現在の該当要件または現在の振る舞い]

## 承認済みの変更

[新しい利用者向けの振る舞い]

## 変更理由

[なぜ現在の仕様を変える必要があるか]

## 受入条件

1. Given: [初期状態]  
    When: [操作または事象]  
    Then: [新しく期待する結果]
2. Given: [境界・異常状態]  
    When: [操作または事象]  
    Then: [期待する処理]

## 変更しないもの

* [維持する既存要件]
* [対象外のユーザーストーリー]
* [今回変更しない機能範囲]

## 社内AI回答の扱い

社内AIの回答は変更案を考えるための参考情報です。  
上記の「承認済みの変更」だけを仕様へ反映してください。

社内AIが提案した次の内容は`spec.md`へ書かないでください。

* パッケージ名
* フレームワーク名
* 内部クラス構成
* ソースコードのフォルダ構成
* 具体的なアルゴリズム
* 実装タスク

## 編集ルール

* 既存の要件IDを可能な限り保持してください。
* 無関係なユーザーストーリーや要件を変更しないでください。
* 完了済みの受入条件を削除しないでください。
* 変更によって矛盾する箇所があれば、勝手に広範囲を修正せず一覧化してください。
* 更新後に、変更された要件、追加された要件、削除された要件を差分として報告してください。
* この段階では`plan.md`、`tasks.md`、コードを変更しないでください。

その後、次の順で進めます。

```
1. 更新されたspec.mdを人間が確認
2. /speckit.clarify
3. 社内AIの技術提案をresearch.mdまたはplan.mdで評価
4. plan.mdを更新
5. tasks.mdを更新
6. /speckit.analyze
7. 変更タスクだけをimplement
```

`/speckit.analyze`は`spec.md`、`plan.md`、`tasks.md`間の矛盾やカバレッジ不足を読み取り専用で調べるため、社内AI提案を反映した後の品質ゲートとして使えます。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+1](https://github.github.com/spec-kit/quickstart.html)

## 社内AIの回答内容と使用コマンドの対応

| 社内AIの回答 | 入力先 |
| --- | --- |
| 新しい利用者向け機能 | `/speckit.specify` |
| 既存機能の動作変更 | 既存`spec.md`を限定編集 |
| 要求の曖昧さへの回答 | `/speckit.clarify` |
| パッケージ候補 | `research.md`、その後`/speckit.plan` |
| フォルダ・モジュール構成 | `/speckit.plan` |
| 技術スタック | `/speckit.plan` |
| 実装作業の提案 | `/speckit.tasks`または部分編集 |
| プロジェクト共通原則 | `/speckit.constitution` |
| 単なる助言 | 相談記録だけに保存 |

したがって、社内AIとの連携用テンプレートを社内標準化するなら、**「社内AI回答→`/speckit.specify`」という1本のテンプレートではなく、「振り分け→specify／plan／tasks」という段階別テンプレート**にする方が安全です。

* * *

**You:**

その他のplanやtasksなどでもプロンプトのテンプレートがあれば教えてください。

* * *

**ChatGPT:**

はい。`plan`、`tasks`、`analyze`、`implement`などについても、**社内AIの回答を安全にSpec Kitへ取り込むための入力テンプレート**を用意できます。

Spec Kitの各コマンドは自由記述の引数を受け取れますが、すべてに長いプロンプトが必要なわけではありません。特に重要なのは次の3つです。

* `/speckit.plan`：採用候補、制約、既存構成、未確定事項を詳しく伝える
* `/speckit.tasks`：既存タスクを保持するか、全面再生成するかを明示する
* `/speckit.implement`：実装範囲、停止位置、検証条件を明示する

公式フローでは、`plan`が技術スタックと設計成果物を生成し、`tasks`が依存順の作業へ分解し、`analyze`が`spec.md`・`plan.md`・`tasks.md`を読み取り専用で検査します。`implement`はタスクを実行し、`converge`は実装漏れを確認して、必要な場合だけ`tasks.md`へ作業を追記します。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages+1](https://github.github.com/spec-kit/reference/agentic-sdd.html)

## 1. `/speckit.clarify`と`/speckit.plan`

`plan`では、公式テンプレート上も言語、依存関係、テスト、対象環境、性能、制約、規模、ソース構成などを扱います。また、未確定事項は`research.md`で解消され、必要に応じて`data-model.md`、`contracts/`、`quickstart.md`も生成されます。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md?plain=1)

# Spec Kit：Clarify・Plan入力テンプレート

## A. `/speckit.clarify`入力テンプレート

/speckit.clarify

現在の`spec.md`について、以下の観点に限定して曖昧さを確認してください。

## 今回確認したい範囲

* [利用者の操作]
* [正常時の振る舞い]
* [異常時・境界条件の振る舞い]
* [対象範囲と対象外]
* [成功条件]
* [既存機能との互換性]

## 社内AIから得た要求面の提案

[社内AI回答のうち、利用者に見える機能・制約だけを記載する]

## 人間が承認済みの内容

* [承認した要求1]
* [承認した要求2]

## 未承認または保留中の内容

* [保留中の提案]
* [追加確認が必要な事項]

## 制約

* 技術スタック、パッケージ、フォルダ構成は、この段階では仕様に追加しないでください。
* 社内AIの提案を自動的に採用しないでください。
* 現在の仕様と矛盾する場合は、勝手に書き換えず矛盾を報告してください。
* 重大な曖昧さだけを質問対象とし、実装時に合理的に判断できる細部は増やしすぎないでください。
* 回答を反映した箇所と、未解決の箇所を最後に報告してください。

---

## B. `/speckit.plan`新規作成用テンプレート

/speckit.plan

現在の`spec.md`を要求の正本として、技術実装計画を作成してください。

## 既存プロジェクト情報

* 使用言語・バージョン：[例：Python 3.11]
* 対象環境：[例：WindowsおよびWSL、オフライン環境]
* 現在の主要パッケージ：[既存パッケージ]
* テスト環境：[例：pytest、gfortran]
* 配布方法：[該当する場合]
* 既存のソース構成：[現在のフォルダ構成]
* 変更してはいけない外部インターフェース：[該当する場合]

## 社内AIから提案された技術案

### パッケージ候補

* [パッケージ名]：[提案理由]
* [パッケージ名]：[提案理由]

### アーキテクチャ・フォルダ構成案

[提案された構成]

### その他の技術提案

* [アルゴリズム]
* [データモデル]
* [テスト方法]
* [配布方法]

## 採用方針

* 承認済み：[すでに採用すると決めた内容]
* 比較検討が必要：[まだ確定していない内容]
* 不採用：[採用しない内容と理由]

## 必須制約

* [ライセンス制約]
* [オフライン導入]
* [対象プラットフォーム]
* [性能・メモリ]
* [機密情報]
* [既存コードとの互換性]
* [人間承認が必要な変更]

## 調査ルール

未確定の技術事項は、推測で確定せず`research.md`で比較してください。

各判断について、可能な限り以下を記録してください。

* Decision
* Rationale
* Alternatives considered
* Risks
* Validation method

社内AIの提案は候補として評価し、現在のspec、constitution、既存コード、配布条件に適合する場合だけ採用してください。

## plan.mdへの反映事項

* Technical Context
* Primary Dependencies
* Testing
* Target Platform
* Performance Goals
* Constraints
* Scale/Scope
* Project Structure
* Structure Decision
* Constitution Check

## 成果物の条件

* 実在する現在のフォルダを確認してから構成を決めてください。
* 不要なレイヤーや抽象化を追加しないでください。
* 今回のfeatureだけに必要な構成と、プロジェクト全体の構成を区別してください。
* 実コードで未検証の事項は未検証と明記してください。
* 既存の重要な設計判断を削除しないでください。
* spec.mdは変更しないでください。
* コード実装は行わないでください。

最後に以下を報告してください。

1. 採用した社内AI提案
2. 修正して採用した提案
3. 不採用とした提案
4. 追加調査が必要な事項
5. 生成・更新した成果物

---

## C. 既存`plan.md`の部分更新用テンプレート

現在の`specs/<feature>/plan.md`を、以下の承認済み技術変更に基づいて更新してください。

## 変更内容

* [パッケージの追加・変更]
* [フォルダ構成の変更]
* [アルゴリズムの変更]
* [データモデルの変更]
* [テスト方法の変更]

## 変更理由

[社内AIの提案、試作結果、既存実装で判明した問題など]

## 維持する事項

* [変更しない技術判断]
* [維持するパッケージ]
* [維持する外部インターフェース]
* [維持するフォルダ構成]

## 編集ルール

* spec.mdは変更しないでください。
* 今回の変更に関係しないplan.mdの記述を変更しないでください。
* 既存の重要な判断理由を削除しないでください。
* 採用前に、constitutionと現在のspec.mdに違反しないか確認してください。
* 新しい依存関係について、ライセンス、対応環境、オフライン導入、保守性を確認してください。
* 未確定事項は確定事項として書かず、research.mdへの追加候補として示してください。
* この段階ではtasks.mdとコードを変更しないでください。

更新後、変更箇所と影響を受けるtasks.mdの項目候補を報告してください。

## 2. `/speckit.checklist`、`/speckit.tasks`、`/speckit.analyze`

`/speckit.tasks`は、設計成果物から依存順の`tasks.md`を生成し、Setup、Foundational、ユーザーストーリー、Polishなどのフェーズへ整理します。`/speckit.analyze`は読み取り専用なので、修正させるコマンドではなく、**矛盾を見つけ、どの正本へ戻るべきか判断する品質ゲート**として使います。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/reference/agentic-sdd.html)

既存featureを変更する場合、`plan.md`や`tasks.md`は再生成だけでなく、手動で部分修正してもよいと公式ガイドに明記されています。完了状態や重要な追記を残したい場合は、全面再生成より標準Agentでの限定編集が安全です。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/guides/evolving-specs.html)

# Spec Kit：Checklist・Tasks・Analyze入力テンプレート

## A. `/speckit.checklist`入力テンプレート

/speckit.checklist

現在の`spec.md`について、以下の品質観点を確認するチェックリストを生成してください。

## 重点領域

* 要件が利用者視点で記述されていること
* 各要件に検証可能な受入条件があること
* 正常系、異常系、境界条件が定義されていること
* 対象範囲と対象外が明確であること
* 未検証事項が検証済みとして記述されていないこと
* 社内AIが提案した技術詳細が要求仕様へ混入していないこと
* 機密情報または実コード固有情報が含まれていないこと
* 既存機能との互換性が明確であること

## 今回特に確認したい事項

* [重点事項1]
* [重点事項2]
* [重点事項3]

この段階ではspec.mdを変更せず、仕様品質を評価するチェックリストだけを生成してください。

---

## B. `/speckit.tasks`新規生成用テンプレート

/speckit.tasks

現在のspec.md、plan.md、research.md、data-model.md、contracts、quickstart.mdを基に、実行可能なtasks.mdを生成してください。

## タスク生成方針

* タスクを依存順に配置してください。
* Setup、Foundational、各ユーザーストーリー、統合・仕上げを区別してください。
* 各タスクが、変更対象の具体的なファイルまたはディレクトリを示すようにしてください。
* 並行実行可能なタスクを明示してください。
* 各ユーザーストーリーを独立して検証できる構成にしてください。
* 必要なテストを、対応するユーザーストーリーまたは実装タスクと関連付けてください。

## 必須で含める作業

* 依存パッケージとロックファイルの更新
* 新規フォルダ・モジュール作成
* 既存コードの移動または互換層の追加
* importおよび参照先の更新
* 単体テスト
* 統合テスト
* 異常系・境界条件テスト
* オフライン環境での導入確認
* コンパイルまたは静的解析
* 数値回帰試験
* ドキュメント更新
* project-status更新

## プロジェクト固有制約

* [実コードは利用できない]
* [合成fixtureで検証する]
* [自動修正前に人間承認が必要]
* [既存数値結果との比較条件]
* [その他]

## 完了条件

各実装タスクについて、可能な限り以下を明確にしてください。

* 実装対象
* 前提タスク
* 変更ファイル
* 対応する要件またはユーザーストーリー
* 検証方法
* 完了と判断する条件

この段階ではコードを実装しないでください。

---

## C. 既存`tasks.md`の部分更新用テンプレート

現在の`specs/<feature>/tasks.md`を、更新済みのspec.mdとplan.mdに合わせて部分更新してください。

## 今回の変更

[仕様または技術計画の変更内容]

## 必要な追加作業

* [新規作業1]
* [新規作業2]
* [移行作業]
* [テスト作業]

## 保持ルール

* `[X]`の完了済みタスクを削除、未完了化、再採番しないでください。
* 今回の変更に影響されない既存タスクを変更しないでください。
* 既存のタスクIDを可能な限り保持してください。
* 完了済みタスクの成果物を修正する必要がある場合は、元のタスクを書き換えず、新しい修正タスクを追加してください。
* 廃止されたタスクは削除せず、理由付きでDeprecatedまたはSupersededとして扱ってください。
* コード実装は行わないでください。

## 追加タスクの条件

* specまたはplan上の根拠を示してください。
* 変更するファイルまたはディレクトリを示してください。
* 検証方法を含めてください。
* 依存する既存タスクを示してください。
* 新規タスクであることが分かるようにしてください。

更新後、以下を報告してください。

1. 追加タスク
2. 変更タスク
3. 廃止扱いにしたタスク
4. 維持した完了済みタスク
5. 未解決の作業分解上の問題

---

## D. `/speckit.analyze`入力テンプレート

/speckit.analyze

現在のspec.md、plan.md、tasks.mdについて、以下を重点的に読み取り専用で分析してください。

## 重点確認事項

* すべての機能要件がtasks.mdでカバーされているか
* plan.mdの技術選択がspec.mdの制約と矛盾していないか
* 社内AIから採用した提案が、未承認の要求変更を引き起こしていないか
* plan.mdに追加されたパッケージや構成に対応するタスクがあるか
* tasks.mdに、仕様または計画上の根拠がないタスクがないか
* 完了済みタスクと現在のplan.mdが矛盾していないか
* テスト、配布、移行、互換性、ドキュメント更新が漏れていないか
* 未検証事項が完了扱いになっていないか
* 対象外とされた機能がtasks.mdに混入していないか

## 出力形式

| Severity | Artifact | Location | Problem | Source of truth to fix | Recommended action |
| --- | --- | --- | --- | --- | --- |

続けて以下を出力してください。

### 実装開始を妨げる問題

CriticalまたはHighの問題だけを列挙してください。

### 実装前に修正を推奨する問題

Mediumの問題を列挙してください。

### 実装後でも対応できる問題

Lowの問題を列挙してください。

### 判定

次のいずれかを明示してください。

* Ready for implementation
* Conditionally ready
* Not ready

ファイルは変更しないでください。  
修正案を示しても、自動適用はしないでください。

## 3. `/speckit.implement`と`/speckit.converge`

大規模featureでは、公式にも`implement`をフェーズやユーザーストーリー単位に限定し、各段階を検証してから続ける方法が示されています。`converge`は現在の実装をspec・plan・tasksと照合し、不足があれば`tasks.md`のConvergenceセクションへ追記するだけで、コードを修正しません。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Pages](https://github.github.com/spec-kit/reference/agentic-sdd.html)

# Spec Kit：Implement・Converge入力テンプレート

## A. `/speckit.implement`段階実装用テンプレート

/speckit.implement

現在のtasks.mdのうち、以下の範囲だけを実装してください。

## 今回の実装範囲

* 対象フェーズ：[フェーズ名]
* 対象ユーザーストーリー：[IDまたは名称]
* 対象タスク：[タスクID]
* 対象外：[今回実装しないフェーズ・タスク]

## 実装ルール

* spec.mdを要求の正本として扱ってください。
* plan.mdの採用済み技術方針に従ってください。
* tasks.mdに存在しない追加機能を実装しないでください。
* 完了済みタスクを不要に再実装しないでください。
* 社内AIの未承認提案をコードへ取り込まないでください。
* 既存の外部インターフェースを勝手に変更しないでください。
* 想定外の仕様判断が必要になった場合は、推測で実装せず停止して報告してください。
* 無関係なリファクタリングを同時に行わないでください。

## 検証条件

* [実行する単体テスト]
* [実行する統合テスト]
* [コンパイル確認]
* [静的解析]
* [数値回帰]
* [異常系fixture]
* [配布またはオフライン導入確認]

## プロジェクト固有制約

* [実コードを使用しない]
* [合成fixtureのみを使用]
* [本番適用済みと主張しない]
* [ソース自動修正前に人間承認]
* [その他]

## 停止条件

以下のいずれかが発生した場合は、後続タスクへ進まず報告してください。

* spec.md、plan.md、tasks.md間の重大な矛盾
* 必須依存関係を導入できない
* 既存テストの予期しない失敗
* 数値結果の許容差超過
* 要求されていない公開API変更
* 機密情報または実コードが必要になる
* 仕様上の判断を新たに行う必要がある

## 完了時の報告

1. 完了したタスク
2. 変更したファイル
3. 実行した検証
4. 検証結果
5. 未完了またはブロックされたタスク
6. spec・plan・tasksへ戻す必要がある発見
7. 次に実装可能な範囲

今回指定した範囲が完了した時点で停止してください。

---

## B. 小規模修正用テンプレート

/speckit.implement

tasks.mdに追加された以下の変更タスクだけを実装してください。

対象タスク：

* [タスクID]
* [タスクID]

この変更は、[仕様変更／技術計画変更／不具合修正]に対応するものです。

次を維持してください。

* 無関係な既存挙動
* 完了済みタスクの成果物
* 公開インターフェース
* 既存データとの互換性
* 現在通過している回帰テスト

実装後、対象テストと影響範囲の回帰テストを実行してください。  
対象外のタスクへは進まないでください。

---

## C. `/speckit.converge`入力テンプレート

/speckit.converge

現在の実装をspec.md、plan.md、tasks.mdと照合し、以下を重点的に確認してください。

## 重点確認事項

* 機能要件の実装漏れ
* 受入シナリオの未実装
* plan.mdに記載された構造との不一致
* tasks.mdで完了扱いだが成果物が存在しない項目
* テストされていない異常系・境界条件
* 新規パッケージ追加に伴う設定・配布・ライセンス確認漏れ
* フォルダ移行後に残った古いimportまたは参照
* ドキュメント、quickstart、project-statusの更新漏れ
* 未検証事項を完了扱いしている箇所
* 社内AIの未承認提案が実装へ混入していないか

## 制約

* コードを変更しないでください。
* spec.mdとplan.mdを変更しないでください。
* 既存タスクを削除または書き換えないでください。
* 実際に不足がある場合だけ、tasks.mdのConvergenceセクションへ追加タスクを追記してください。
* 推奨改善と、仕様適合に必須の作業を区別してください。
* 単なる好みや任意のリファクタリングを必須タスクとして追加しないでください。

最後に、次のいずれかを明示してください。

* Converged
* Additional tasks required

追加タスクがある場合は、それぞれについて根拠となるspec、planまたは既存taskを示してください。

---

## D. 実装完了後の状態更新テンプレート

標準Agentモードで、今回の実装結果を基に`docs/project-status.md`を更新してください。

## 参照対象

* 対象featureのspec.md
* plan.md
* tasks.md
* テスト結果
* `/speckit.converge`の結果
* Git差分

## 更新する項目

* Last updated
* Current phase
* Active feature
* Completed
* In progress
* Blocked
* Verification status
* Known limitations
* Next actions

## ルール

* 完了した事実と予定を明確に区別してください。
* 実コードで未検証なら、その状態を維持してください。
* テスト未実行をPassedと記載しないでください。
* tasks.mdのチェック数だけで進捗を判断しないでください。
* 社内AIの提案は、採用・実装・検証されたものだけを完了事項として記載してください。
* 過去の詳細な作業履歴を増やさず、現在の作業再開に必要な短い状態を維持してください。

## 日常運用では全部を毎回使わなくてよい

推奨する使い分けは次のとおりです。

| 状況 | 使用するテンプレート |
| --- | --- |
| 新規feature | `clarify → plan新規 → tasks新規 → analyze → implement → converge` |
| 小さな仕様変更 | `spec部分編集 → plan部分更新 → tasks部分更新 → analyze` |
| パッケージ変更 | `plan部分更新 → tasks部分更新 → analyze` |
| フォルダ再編 | `plan部分更新 → tasks部分更新 → analyze → 段階implement` |
| コードで問題発見 | `tasksまたはコード修正 → 必要なら上流更新 → analyze` |
| 実装範囲が大きい | `implement`をフェーズ単位で複数回実行 |
| 実装完了判定 | `converge` |
| チャット・担当切替 | `project-status`更新 |

これらのテンプレートを社内標準にする場合は、まず`.github/prompts/`の独自`.prompt.md`として保存し、運用が安定してからSpec Kit Presetへ移すのがよいです。Spec KitのPresetは、標準のspec・plan・tasksテンプレートやコマンドを組織向けに上書きする正式な仕組みです。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/github/spec-kit/blob/main/docs/reference/overview.md/)

* * *

**You:**

その他、社内AIへのプロンプトのテンプレートも教えてください。

* * *

**ChatGPT:**

社内AI向けには、**相談内容・与える前提・求める出力形式・採用判断の扱い**を明示するのが重要です。  
以下は、Spec Kitと連携しやすい用途別テンプレートです。

# 社内AI相談用プロンプトテンプレート集

## 共通ルール

以下の各テンプレートでは、必要に応じて次の資料を添付または貼り付ける。

* プロジェクト概要
* 現在の進捗状況
* 対象featureの`spec.md`
* 対象featureの`plan.md`
* 対象featureの`tasks.md`
* テスト結果
* 既知の問題
* 機密情報を除去したコード例
* 社内AIへの質問事項

社内AIの回答は、未承認の技術的助言として扱う。回答内容を自動的に仕様、計画、タスクまたはコードへ反映しない。

---

# 1. プロジェクト全体の進捗と次の方向性を相談する

あなたは、ソフトウェア開発プロジェクトの技術アドバイザーです。

以下に、プロジェクトの目的、現在の進捗、実装済み機能、未実装機能、既知の問題を示します。

## プロジェクトの目的

[プロジェクト全体の目的]

## 現在の進捗

[project-status.mdの機密情報を除去した内容]

## 実装済み

* [実装済み機能]
* [検証済み機能]

## 実装中

* [現在実装中の機能]

## 未実装

* [今後必要な機能]

## 既知の問題・制約

* [技術的問題]
* [利用可能なデータや環境の制約]
* [未検証事項]
* [社内ルール]

## 現在検討している選択肢

1. [選択肢A]
2. [選択肢B]
3. [選択肢C]

## 相談事項

* 次に優先すべき機能は何か
* 現在の開発順序に問題がないか
* 先に解消すべき技術的リスクはあるか
* 小規模なPoCとして区切れる単位はどこか
* 現時点で過剰設計になっている部分はないか

## 出力形式

### 現状評価

現在のプロジェクト状態を簡潔に評価してください。

### 推奨する次の段階

次に行う作業を優先順位順に示してください。

### 理由

各提案の理由、前提、期待効果を説明してください。

### リスク

提案を採用した場合のリスクと、採用しなかった場合のリスクを分けてください。

### Spec Kitへの反映候補

提案を次のいずれへ反映すべきか分類してください。

* 新しい`spec.md`
* 既存`spec.md`の変更
* `research.md`
* `plan.md`
* `tasks.md`
* `constitution.md`
* `project-status.md`
* 反映不要

## 制約

* 実装済みと未実装を混同しないでください。
* 未検証事項を、検証済みとして扱わないでください。
* 現在の制約を無視した理想論だけを提示しないでください。
* 提案は採用候補であり、決定事項として記述しないでください。

---

# 2. パッケージやライブラリを比較する

あなたは、ソフトウェアアーキテクトおよび依存関係管理の専門家です。

以下の機能を実装するために、使用するパッケージまたはライブラリを検討しています。

## 実現したい機能

[ユーザー視点の機能要求]

## 対象環境

* 言語・バージョン：[例：Python 3.11]
* OS：[例：Windows、WSL]
* ネットワーク：[例：オフライン環境]
* 配布方法：[例：Python未導入PCへ配布]
* 既存依存関係：[パッケージ一覧]
* ライセンス制約：[社内ルール]
* 性能要件：[該当する場合]

## 候補

* [候補A]
* [候補B]
* [候補C]
* 標準ライブラリまたは自作

## 評価観点

* 要求適合性
* 保守性
* 学習コスト
* 依存関係の大きさ
* オフライン導入の容易さ
* 対応OS
* ライセンス
* 性能
* 開発の活発さ
* 将来の移行コスト
* ベンダーロックイン
* テスト容易性

## 出力形式

| 候補 | 長所 | 短所 | 制約適合性 | 主なリスク | 推奨度 |
| --- | --- | --- | --- | --- | --- |

続けて、以下を示してください。

### 推奨案

第一候補と、その理由を示してください。

### 条件付き代替案

どのような条件で別候補を選ぶべきか示してください。

### 導入前に確認すべき事項

実際の採用前に検証すべき事項を示してください。

### 小規模検証案

採否を判断するための最小限のPoCを提案してください。

### Spec Kitへの反映

* `research.md`へ残す比較内容
* `plan.md`へ記載する採用結果
* `tasks.md`へ追加する検証・導入作業

## 制約

* パッケージの知名度だけで判断しないでください。
* 現在の制約に適合しない候補を無条件に推奨しないでください。
* 不明なライセンスや対応状況を断定しないでください。
* 必要以上に依存関係を増やさない案も検討してください。

---

# 3. アーキテクチャとフォルダ構成を相談する

あなたは、既存ソフトウェアの段階的な設計改善を支援するアーキテクトです。

以下のプロジェクトについて、パッケージ構成、モジュール分割、フォルダ構成を検討してください。

## プロジェクト概要

[プロジェクトの目的]

## 現在の構成

```
[現在のフォルダ構成]
```

## 現在の問題

* [責務が混在している]
* [循環依存がある]
* [テストが難しい]
* [CLIと解析ロジックが密結合]
* [将来の機能追加が難しい]

## 今後予定している機能

* [機能A]
* [機能B]
* [機能C]

## 制約

* 大規模な一括書き換えは避ける
* 既存テストを維持する
* 公開インターフェースを可能な限り維持する
* 過剰なレイヤー分割を避ける
* [その他の制約]

## 相談事項

1. 現在の構成の問題点
2. 推奨する責務分割
3. 推奨するフォルダ構成
4. 既存構成からの段階的な移行方法
5. 現時点では導入しない方がよい抽象化
6. 将来変更しやすくする境界

## 出力形式

### 現在の構成の評価

### 推奨する構成

```
[提案フォルダ構成]
```

### 各フォルダ・モジュールの責務

| 場所 | 責務 | 依存してよい対象 | 依存してはいけない対象 |
| --- | --- | --- | --- |

### 段階的な移行手順

移行を小さな単位へ分けてください。

### トレードオフ

提案構成の利点と欠点を示してください。

### Spec Kitへの反映候補

* `research.md`へ記録する比較・判断
* `plan.md`へ記録する最終構成
* `tasks.md`へ追加する移行作業
* 複数featureに影響する事項

## 制約

* 新規プロジェクトとして理想構成を提案するだけでなく、現在の構成から移行可能な案にしてください。
* フォルダ数やクラス数を不必要に増やさないでください。
* 提案を採用決定事項として扱わないでください。

---

# 4. 仕様の不足や矛盾をレビューする

あなたは、要求分析とソフトウェアテスト設計の専門家です。

以下の仕様書をレビューし、曖昧さ、矛盾、欠落、検証不能な要件を指摘してください。

## 仕様書

[機密情報を除去したspec.md]

## プロジェクト制約

* [制約]
* [対象外]
* [未検証事項]

## 重点確認項目

* 利用者が誰か明確か
* 利用者の操作と期待結果が明確か
* 正常系が定義されているか
* 異常系が定義されているか
* 境界条件が定義されているか
* 受入条件が検証可能か
* 成功条件が測定可能か
* 対象範囲と対象外が明確か
* 技術実装の詳細が仕様へ混入していないか
* 既存仕様と矛盾していないか

## 出力形式

| 重要度 | 該当箇所 | 問題 | 問題となる理由 | 推奨する修正 |
| --- | --- | --- | --- | --- |

続けて、以下を示してください。

### 実装前に解決すべき問題

### 実装中でも判断可能な問題

### 追加を検討すべき受入シナリオ

### Spec Kitでの反映先

* `spec.md`修正
* `/speckit.clarify`で確認
* `plan.md`で決めるべき事項
* `tasks.md`で対応すべき事項

## 制約

* 仕様に存在しない新機能を勝手に追加しないでください。
* 技術的な好みを要求として扱わないでください。
* 軽微な表現上の問題と、実装結果へ影響する問題を区別してください。

---

# 5. 実装結果が仕様と異なった場合の原因を相談する

あなたは、仕様、設計、実装、テストの不整合を分析する技術レビュー担当者です。

以下の機能について、想定した結果と実装結果が一致しません。

## 当初の期待

[spec.mdに基づく期待]

## 現在の実装結果

[実際の動作]

## 関連する仕様

[関連要件・受入条件]

## 関連する技術計画

[plan.mdの該当部分]

## 実装内容

[機密情報を除去したコードまたは処理概要]

## テスト結果

[テスト内容と結果]

## 相談事項

以下のどれに該当する可能性が高いか分析してください。

1. 仕様が誤っている
2. 仕様が曖昧である
3. 技術計画が仕様を満たしていない
4. tasks.mdの作業分解が不足している
5. 実装が仕様またはplanに従っていない
6. テスト条件が誤っている
7. 前提条件そのものが誤っている

## 出力形式

### 最も可能性の高い原因

### 根拠

### 追加で確認すべき事項

### 推奨する修正順序

### 修正対象の分類

| 修正対象 | 修正が必要か | 理由 |
| --- | --- | --- |
| `spec.md` |  |  |
| `research.md` |  |  |
| `plan.md` |  |  |
| `tasks.md` |  |  |
| コード |  |  |
| テスト |  |  |

### 最小限の再検証方法

## 制約

* 実装結果に合わせて、仕様を安易に変更しないでください。
* 仕様どおりでない実装を、仕様変更として正当化しないでください。
* 情報不足の場合は、断定と仮説を区別してください。

---

# 6. テスト・検証計画を相談する

あなたは、解析ソフトウェアおよび科学技術計算ソフトウェアの検証担当者です。

以下の機能について、現実的な検証計画を提案してください。

## 対象機能

[機能概要]

## 要求・成功条件

[spec.mdの該当箇所]

## 技術構成

[plan.mdの該当箇所]

## 現在利用可能な検証資産

* 単体テスト：[有無]
* 合成fixture：[有無]
* 既存結果：[有無]
* コンパイラ：[利用可能なもの]
* 実データ：[利用可否]
* 実コード：[利用可否]
* 比較対象ソフトウェア：[有無]

## 制約

* [実コードは利用不可]
* [実測データは利用不可]
* [オフライン環境]
* [その他]

## 相談事項

* 単体テストで確認すべき内容
* 統合テストで確認すべき内容
* 境界条件・異常条件
* 数値回帰で確認すべき指標
* 実データなしで確認可能な範囲
* 実データが必要になる検証
* 合否基準
* 誤検出・見逃しの評価方法

## 出力形式

| 検証レベル | 検証対象 | 入力 | 期待結果 | 合否基準 | 現在実施可能か |
| --- | --- | --- | --- | --- | --- |

続けて、以下を示してください。

### 最小検証セット

### 推奨検証セット

### 現時点では実施できない検証

### 残存リスク

### tasks.mdへ追加すべき検証タスク

## 制約

* 実施できない検証を、完了条件として曖昧に扱わないでください。
* 合成データで確認できることと、実データでしか確認できないことを区別してください。
* テスト通過を、本番適用可能性の証明として扱わないでください。

---

# 7. プロジェクトの簡素化・過剰設計をレビューする

あなたは、小規模から中規模の開発プロジェクトの設計レビュー担当者です。

以下の設計と実装計画について、現在の要求に対して過剰な要素がないか評価してください。

## 現在の要求

[spec.mdの要約]

## 現在の技術計画

[plan.mdの要約]

## 現在のフォルダ構成

```
[構成]
```

## 今後想定している拡張

* [拡張A]
* [拡張B]

## 評価してほしい点

* 現時点では不要な抽象化
* 不要なパッケージ
* 過剰なレイヤー
* 細かすぎるモジュール分割
* 将来要件を先取りしすぎている設計
* 逆に、今のうちに分離すべき責務
* 削除または延期できるタスク

## 出力形式

### 維持すべき設計

### 簡素化を推奨する設計

### 将来まで延期できるもの

### 今のうちに対応すべきもの

### plan.mdへの修正候補

### tasks.mdから削除または延期する候補

## 制約

* 将来拡張の可能性だけを理由に、複雑な設計を推奨しないでください。
* 単純化によって失われる保守性や安全性も示してください。
* 提案を自動的な変更指示として扱わないでください。

社内AIの回答を受け取った後は、採用内容をそのままCopilotへ渡すのではなく、次のテンプレートで評価・振り分けてからSpec Kitへ戻します。

# 社内AI回答の評価・Spec Kit反映用テンプレート

以下は、社内AIから受け取った回答です。

この回答は未承認の助言です。現在の仕様、技術計画、プロジェクト制約と比較し、採否と反映先を整理してください。

## 参照対象

* `.specify/memory/constitution.md`
* `docs/project-status.md`
* `docs/roadmap.md`
* 対象featureの`spec.md`
* 対象featureの`research.md`
* 対象featureの`plan.md`
* 対象featureの`tasks.md`
* 現在のコードおよびテスト

## 社内AIへ送った相談内容

[元の相談プロンプト]

## 社内AIの回答

[社内AIの回答]

## 評価基準

* 現在の機能要求に適合するか
* constitutionに違反しないか
* 現在の技術構成と整合するか
* 現在の制約内で実施可能か
* 追加依存関係が妥当か
* オフライン環境で利用可能か
* ライセンス上問題がないか
* 過剰設計にならないか
* 現在の検証環境で確認可能か
* 将来の保守負担が過大でないか

## 提案ごとの分類

各提案を次のいずれかへ分類してください。

* Accept：そのまま採用候補
* Accept with modification：修正して採用候補
* Investigate：追加調査が必要
* Defer：将来へ延期
* Reject：不採用
* Already implemented：すでに対応済み
* Not applicable：現在の対象外

## 出力形式

| ID | 提案 | 判定 | 理由 | リスク | 反映先 | 必要な人間判断 |
| --- | --- | --- | --- | --- | --- | --- |

## Spec Kit成果物への振り分け

### `spec.md`へ反映する候補

利用者に見える機能、動作、必須制約だけを示してください。

### `research.md`へ反映する候補

比較検討、代替案、追加調査が必要な技術事項を示してください。

### `plan.md`へ反映する候補

採用するパッケージ、アーキテクチャ、データモデル、フォルダ構成を示してください。

### `tasks.md`へ反映する候補

具体的な実装、移行、テスト、設定変更を示してください。

### `constitution.md`へ反映する候補

複数featureで長期間守る必要がある原則だけを示してください。

### `project-status.md`へ反映する候補

新たに判明したブロッカー、未検証事項、次の行動を示してください。

## 変更の優先順位

1. 実装前に必須
2. 次の開発段階で対応
3. 将来検討
4. 対応不要

## 最終出力

最後に、以下を作成してください。

### 採用候補一覧

### 不採用候補一覧

### 追加調査事項

### 人間による承認が必要な事項

### 推奨するSpec Kit実行順序

例：

1. 既存`spec.md`の限定更新
2. `/speckit.clarify`
3. `research.md`の更新
4. `plan.md`の限定更新
5. `tasks.md`の追加
6. `/speckit.analyze`
7. 変更タスクのみ`/speckit.implement`

## 制約

* 社内AIの回答を自動的に採用しないでください。
* 社内AIの提案に合わせるためだけに仕様を変更しないでください。
* 現在存在しない機能を実装済みとして扱わないでください。
* 情報不足の場合、推測と確認済み事実を分けてください。
* この段階ではファイルやコードを変更しないでください。

実運用では、まず次の3種類を標準化すると管理しやすくなります。

1. **プロジェクト進捗・次期方針相談**
2. **パッケージ・アーキテクチャ比較**
3. **社内AI回答の評価とSpec Kitへの振り分け**

この3つで運用し、必要性が明確になった段階で、仕様レビュー、実装不一致分析、検証計画などを追加するのが扱いやすい構成です。

* * *

**You:**

speckit、プロジェクトフォルダ状況管理、社内AIとの連携を行うのに必要なファイルを全て生成していただけませんか？

* * *

**ChatGPT:**

社内横断で使える雛形として、Spec Kit本体を壊さず追加できる「連携パック」を作ります。公式の現在の配置を確認したうえで、Markdown、Copilotプロンプト、状態管理スクリプト、社内AI受け渡しテンプレートをまとめてZIP化します。

![](https://www.google.com/s2/favicons?domain=github.com&sz=128)Searched 19 websites

構成は、既存の`.specify/`を上書きしない追加型にします。`spec.md`・`plan.md`・`tasks.md`を正本として保ちつつ、全体状況、社内AIの未承認回答、承認済み反映を明確に分離します。