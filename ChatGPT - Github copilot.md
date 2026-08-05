**You:**

github copilotのクレジット節約の観点で、仕様や実装計画の考案には高級なLLMのモデルを、実装段階には低級なモデルを用いるのが良いとのことでしたが、それぞれについておすすめのモデルはありますか？

* * *

**ChatGPT:**

はい。ただし、**「設計＝最高級、実装＝最安モデル」だけでは不十分**です。実装でも複数ファイルを自律的に変更させる場合、安価なモデルが迷走して再試行を重ね、かえってクレジットを消費することがあります。

## 私の推奨構成

| 作業 | 第一候補 | 節約重視 | 使用場面 |
| --- | --- | --- | --- |
| 仕様・アーキテクチャ設計 | **GPT-5.4** | Claude Sonnet 5 | 要件整理、責務分割、段階計画 |
| 特に難しい設計判断 | **GPT-5.6 Sol** | Gemini 3.1 Pro | 大規模コード調査、方式比較、計画レビュー |
| 複数ファイルの実装 | **GPT-5.3-Codex** | GPT-5.6 Terra | Agentモード、実装・テスト・修正 |
| 小さく明確な実装 | **GPT-5.6 Luna** | GPT-5.4 nano | 単一関数、テスト追加、定型修正 |
| 軽量なコード作業 | Kimi K2.7 Code | MAI-Code-1-Flash | 比較的単純な生成、説明、修正 |
| 最終レビュー | **GPT-5.4またはSol** | Claude Sonnet 5 | 計画との乖離、見落とし、設計劣化の確認 |

GitHub自身も、GPT-5.4を「多段階の問題解決・アーキテクチャレベルのコード分析」、GPT-5.6 Solを「大規模コードベースの複雑な推論」、GPT-5.3-Codexを「Agentic software development」、GPT-5.6 Lunaを「小さく高速な反復作業」向けに分類しています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/en/copilot/reference/ai-models/model-comparison)

## 最もおすすめする組み合わせ

### 1. 費用対効果を重視する標準構成

**仕様・実装計画：GPT-5.4**  
**実装：GPT-5.6 Luna**  
**Lunaで失敗した実装のみ：GPT-5.3-Codex**

GPT-5.4は設計用途として十分強力ですが、GPT-5.6 Solほど高価ではありません。現在の従量課金では、100万トークン当たりの出力価格がGPT-5.4は15ドル、Solは30ドルです。一方、Lunaは1.20ドルと大幅に安価です。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)

ただし、Lunaに渡すタスクは次のように限定します。

> `COMMON検出機能を実装してください`

では広すぎます。

> `src/common/parser.py の parse_common_statement() のみを修正してください。仕様は docs/tasks/TASK-012.md、完了条件は tests/test_common_parser.py の全テスト成功です。他ファイルは変更しないでください。`

この程度まで計画側で分解できていれば、Lunaを使いやすくなります。

### 2. 実装の安定性を優先する構成

**仕様・実装計画：GPT-5.4またはGPT-5.6 Sol**  
**実装：GPT-5.3-Codex**  
**定型作業だけLuna**

GPT-5.3-Codexは単価上はLunaより高いものの、GitHubは複雑な機能実装、テスト、デバッグ、リファクタリング、レビューなどのAgent作業向けとして推奨しています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/en/copilot/reference/ai-models/model-comparison)

特に以下では、最初からCodexを選んだ方が総消費量を抑えやすいです。

* 3ファイル以上にまたがる変更
* ASTやfparser2を使う処理
* 既存テストを壊さずに機能追加する作業
* コンパイル・pytest・回帰比較を繰り返す作業
* エラー原因を調査してから修正する作業

Lunaにこのような作業を丸ごと任せると、探索や修正の往復が増える可能性があります。

## あなたのFortranツール開発に当てはめると

### 高級モデルを使う場面

GPT-5.4を基本とし、特に難しいときだけSolを使うのがよいです。

* COMMON修正ワークフロー全体の設計
* CALL解析と引数整合の方式決定
* 暗黙型解消の安全性ルール
* gfortran移植までの段階分割
* 承認候補と自動修正の境界
* FortranからPythonへの責務分割
* 数値回帰テストの判定基準
* 計画変更の判断

**Solを毎回使う必要はありません。**  
設計上の選択肢が複数あり、間違えると後工程を大きく作り直す場面だけで十分です。SolはGPT-5.6系で最も高い推論能力を持つ一方、高コストモデルです。[![](https://www.google.com/s2/favicons?domain=https://github.blog&sz=128)The GitHub Blog+1](https://github.blog/changelog/2026-07-09-openais-gpt-5-6-sol-terra-and-luna-are-now-available-in-github-copilot/)

### Lunaでよい場面

* dataclassの追加
* CLIオプションの追加
* JSONレポート項目の追加
* 既存パターンに沿ったテストケース追加
* docstringやREADME更新
* 型ヒントの追加
* 既に設計済みの小関数実装
* エラーメッセージの改善

### CodexまたはTerraに上げる場面

* パーサと解析器とレポート出力を同時に変更
* テスト失敗の原因調査が必要
* fparser2のASTを探索する必要がある
* 複数の既存実装から適切な修正箇所を判断する
* 実装後にコンパイル・pytest・回帰比較まで自律実行する

Terraは日常的な対話・Agentコーディングのバランス型、Codexはより明確にAgent型ソフトウェア開発向けとされています。[![](https://www.google.com/s2/favicons?domain=https://github.blog&sz=128)The GitHub Blog+1](https://github.blog/changelog/2026-07-09-openais-gpt-5-6-sol-terra-and-luna-are-now-available-in-github-copilot/)

## クレジット節約を最優先にした実運用

私なら次の順番にします。

1. **GPT-5.4で、直近1段階の仕様・完了条件・変更可能ファイルを決める**
2. その内容を`TASK.md`に保存する
3. **Lunaで1タスクずつ実装する**
4. Lunaが2回失敗したら、それ以上再試行せず**GPT-5.3-Codexへ上げる**
5. 段階の最後だけGPT-5.4でレビューする

ポイントは、安価なモデルに何度もやり直させないことです。  
「最大2回で上位モデルへ切り替える」と決めておくと、迷走による浪費を抑えられます。

さらに、単純な一行修正や定型コードはChatやAgentではなく**インライン補完**を使うのが有利です。有料プランではコード補完とNext Edit SuggestionsはAIクレジット課金の対象外です。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)

## 現在Freeプランを利用している場合

2026年6月24日以降、Copilot FreeとStudentは**Autoモデル選択のみ**になっており、自分で「設計はSol、実装はLuna」のようにモデルを切り替えることはできません。[![](https://www.google.com/s2/favicons?domain=https://github.blog&sz=128)The GitHub Blog+1](https://github.blog/changelog/2026-06-24-changes-to-model-selection-for-free-and-student-plans/)

この場合は、モデル選択ではなく次の分離が現実的です。

* ChatGPTや別サービスの高性能モデルで仕様・計画を作る
* Copilot Autoには、十分に細分化した実装タスクだけを渡す
* 実装の大部分をインライン補完で進める
* Agentモードを使うのは、変更範囲が明確なタスクだけにする

有料プランでもAutoはタスクの複雑度に応じてモデルを振り分け、手動選択よりモデル費用が10％割引されます。そのため、細かくモデルを管理するのが面倒なら、**Auto＋明確なTASK.md**もかなり合理的です。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/en/copilot/concepts/models/auto-model-selection)

結論として、あなたの開発では **「GPT-5.4で計画、Lunaで小規模実装、Codexで複雑実装、Solは重要な設計判断だけ」** が最もバランスのよい構成です。

* * *

**You:**

仕様や計画が明確なら、複数ファイルの編集もLunaで実装可能になるわけではないの？

* * *

**ChatGPT:**

はい、その理解で合っています。**仕様と実装計画が十分に具体的なら、複数ファイルの編集でもLunaに任せられます。**

前回の「3ファイル以上ならCodex」という表現は、やや単純化しすぎでした。GitHub公式もLunaを「小さく高速なタスク向け」と位置づけていますが、**単一ファイル限定とはしていません**。判断基準はファイル数ではなく、タスクの不確実性と依存関係の複雑さです。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/en/copilot/reference/ai-models/model-comparison?search-overlay-input=agents&search-overlay-open=true&utm_source=chatgpt.com)

## Lunaで複数ファイルを編集しやすい条件

例えば、次のように変更内容が明確なら、5～10ファイルにまたがっていてもLunaの候補になります。

* 変更対象ファイルが指定されている
* 各ファイルで何を変更するか決まっている
* 新しいクラスや関数のシグネチャが決まっている
* 既存の実装パターンを横展開するだけ
* 完了条件がテストとして定義されている
* アーキテクチャ上の判断を実装中に求めない
* 変更禁止の範囲が明示されている

たとえば、あなたのFortranツールなら、次のようなタスクです。

> `CommonIssue`に`source_span`を追加する。  
> `models.py`でフィールドを定義し、`analyzer.py`で値を設定し、`json_reporter.py`でJSONに出力する。  
> 既存JSONとの後方互換性を維持する。  
> `tests/test_common_analyzer.py`と`tests/test_json_reporter.py`にテストを追加する。  
> 上記以外のファイルは変更しない。

これは複数ファイルですが、変更経路がほぼ一本道なのでLuna向きです。

## ファイル数が少なくても上位モデル向きの例

逆に、1～2ファイルだけでも次のような依頼はLunaには重い可能性があります。

> COMMON文の検出漏れの原因を調査し、固定形式、継続行、型宣言との相互作用を考慮して最適な修正方法を判断してください。

ここでは実装前に、

* 原因箇所の探索
* fparser2 ASTの調査
* 複数方式の比較
* 後方互換性の判断
* 想定外ケースへの対応

が必要です。これはファイル数ではなく、**実装中に推論と設計が発生する**ため、TerraやGPT-5.3-Codexの方が安定しやすいでしょう。GitHubはTerraを日常的なAgent作業のバランス型、Solを大規模コードベースの複雑な推論向けとしています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/en/copilot/reference/ai-models/model-comparison?search-overlay-input=agents&search-overlay-open=true&utm_source=chatgpt.com)

## 判断軸は「作業量」と「判断量」

次のように考えると分かりやすいです。

| タスク | ファイル数 | 実装中の判断 | 適したモデル |
| --- | --- | --- | --- |
| 同じフィールドを複数層に追加 | 6 | 少ない | Luna |
| CLIオプションを追加して既存関数へ渡す | 4 | 少ない | Luna |
| 既存パターンに従うテスト追加 | 8 | 少ない | Luna |
| パーサの検出漏れ原因を調査 | 2 | 多い | Terra / Codex |
| モジュール境界を再設計 | 5 | 非常に多い | GPT-5.4 / Sol |
| 原因不明の回帰エラーを自律修正 | 3 | 多い | Codex / Terra |

つまり、

> **多数ファイル × 機械的変更**

はLuna向きですが、

> **少数ファイル × 探索・設計・デバッグ**

は上位モデル向きです。

## ただし、クレジット面ではコンテキスト量に注意

現在のCopilotはモデルだけでなく、入出力トークン量によってAIクレジット消費が変わります。そのため、Lunaで処理可能な簡単な変更でも、リポジトリ全体を無制限に探索させると、読み込むコンテキストが増えて消費量も増えます。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/copilot/reference/copilot-billing/models-and-pricing?utm_source=chatgpt.com)

Lunaを使う場合は、例えば次の制限が効果的です。

```
実装対象:
- src/models.py
- src/analyzers/common.py
- src/reporters/json.py
- tests/test_common.py

参照してよいファイル:
- src/reporters/base.py
- tests/fixtures/common/

上記以外は、必要性が明確になるまで読み込まないでください。
アーキテクチャや公開APIは変更しないでください。
```

## あなたのプロジェクトに適した分担

実際には、次の運用が最も合理的だと思います。

### 高級モデル

以下を作成します。

* 変更対象ファイル
* 各ファイルの変更内容
* 公開APIとデータ構造
* 実装順序
* 完了条件
* テストケース
* 変更禁止事項
* 判断が必要になった場合の停止条件

### Luna

その計画をもとに、

* 複数ファイルの編集
* テスト追加
* テスト実行
* 明白なエラー修正
* レポート更新

まで行います。

### TerraまたはCodexへ切り替える条件

Lunaが次の状態になった場合だけ上げます。

* 計画外のファイルを多数調査し始める
* API設計の変更を提案し始める
* 同じテストを2回以上異なる方法で壊す
* 原因を特定できず場当たり的な修正を繰り返す
* fparser2のAST構造などを新たに調査する必要が出る
* 計画自体に矛盾が見つかる

したがって、より正確な方針は、

> **仕様が明確なら、Lunaに複数ファイルを編集させてよい。ただし、実装中に探索・設計判断・難しいデバッグが発生した時点で上位モデルへ切り替える**

です。

あなたのように、先に仕様・段階計画・承認条件・回帰テストまで整備する開発方法は、かなりLunaを活用しやすい進め方です。

* * *

**You:**

クレジット節約方法として、実装段階ではlunaを用いることで単純な節約になりそうなのですが、実装・計画段階で節約する方法を知りたいです。例えばspeck-kitのようなSDDの手順がきっちり組まれている場合については、specifyやplan,taskの段階でも軽量モデルを用いても性能が落ちにくいようなことはあるの？

* * *

**ChatGPT:**

はい。**Spec Kitのように手順・入力・出力形式が固定されたSDDでは、軽量モデルでも性能低下を抑えやすくなります。**  
ただし、効果は全工程で同じではありません。

重要なのは、

> **SDDはモデルの「手順遵守」を助けるが、「曖昧な要求の発見」や「アーキテクチャ判断」まで自動的に簡単にするわけではない**

という点です。Spec Kitによって高級モデルと軽量モデルの差が完全に消えるという公式な検証結果はありませんが、**入力が十分に明確な工程ほど軽量モデルへ移しやすい**と考えられます。

## 工程別の軽量モデル適性

| 工程 | 軽量モデル適性 | 推奨 |
| --- | --- | --- |
| `constitution` | 中 | 最初だけ上位モデル、以後は再利用 |
| `specify` | 中～高 | 要求が具体的ならLuna |
| `clarify` | 低～中 | TerraやGPT-5.4 mini |
| `plan` | 条件付き | 設計済みならLuna、設計判断が残るならTerra以上 |
| `tasks` | 非常に高い | Lunaが第一候補 |
| `checklist` | 高い | Luna |
| `analyze` | 中～高 | 形式的一貫性はLuna、意味的矛盾は上位モデル |
| `implement` | 高い | タスク単位でLuna、問題発生時に昇格 |

Spec Kitは`specify → clarify → plan → checklist → tasks → analyze → implement`という工程を用意しており、`clarify`、`checklist`、`analyze`は「意味のある曖昧さがある場合」に追加する品質ゲートとして位置づけています。つまり、毎回すべて実行する必要はありません。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/github/spec-kit/blob/main/docs/reference/overview.md/)

## `specify`はLunaでもよいのか

ユーザー側で以下を提示できるなら、かなりLuna向きです。

* 何を実現するか
* なぜ必要か
* 対象ユーザーまたは利用場面
* 入出力
* 正常系・異常系
* 非対象範囲
* 完了条件

例えば、

> COMMON文を安全にmoduleへ変換できるようにする

だけでは、Lunaには曖昧です。

一方、次のようならLunaでも十分期待できます。

```
対象:
COMMONブロックの使用箇所を解析し、module変数への置換候補を生成する。

今回の範囲:
- 変換候補の生成まで
- ソースコードの自動書換えはしない
- 固定形式と自由形式の両方を対象とする

完了条件:
- ブロック名、変数順、型、宣言箇所をJSON出力する
- 不整合がある場合は変換不可として理由を出力する
- 既存テストを壊さない
```

この状態では、`specify`は「要求を考える作業」よりも、**与えられた要求をテンプレートへ整理する作業**になります。これが、SDDで軽量モデルを使いやすくなる理由です。

## `plan`は最も注意が必要

`plan`には二種類あります。

### 1. 計画を文章化するだけ

以下が既に決まっている場合です。

* 使用ライブラリ
* 変更対象モジュール
* 新しいデータ構造
* API
* 実装順序
* テスト方針
* 既存コードとの境界

この場合、Lunaでも可能です。

### 2. 計画そのものを考える

以下が未決定の場合です。

* 正規表現かfparser2 ASTか
* COMMONを引数化するかmodule化するか
* どこまで自動修正するか
* 既存APIを維持するか変更するか
* どの段階で数値回帰を行うか
* 失敗時にロールバックする単位

これは単なる文書生成ではなく、**技術的意思決定**です。LunaよりTerra、GPT-5.4 mini、GPT-5.4などが向いています。GitHubもLunaを小規模・高速・反復的な作業向け、Terraを日常的なAgent作業、GPT-5.4を多段階問題やアーキテクチャ分析向けと分類しています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/en/copilot/reference/ai-models/model-comparison?search-overlay-input=agents&search-overlay-open=true)

さらに、標準の`/speckit.plan`は単に`plan.md`を書くものではありません。未確定事項ごとに調査タスクを生成し、`research.md`、`data-model.md`、`contracts/`、`quickstart.md`なども作る設計です。`NEEDS CLARIFICATION`、依存技術、統合対象が多いほど、調査と出力が膨らみます。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md?plain=1&utm_source=chatgpt.com)

したがって、クレジット節約には、モデル変更以上に**`plan`へ渡す前に未確定事項を減らすこと**が効きます。

## `tasks`はLunaへ移しやすい

`tasks`は、明確な`spec.md`と`plan.md`を実行可能な単位へ分割する工程です。これは通常、

* ファイルごとの変更内容
* 依存順序
* 並列実行可能性
* テスト追加
* 完了条件

への変換なので、SDDの中で最も軽量モデル向きです。Spec Kit公式でも、`tasks`は実装計画から実行可能なタスクリストを生成する工程として定義されています。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/github/spec-kit/blob/main/README.md)

ただし、`tasks`実行時にアーキテクチャを再検討させないことが重要です。

```
plan.mdの設計判断は変更しないでください。
未確定事項を発見した場合は、独自判断せずBLOCKEDとして記録してください。
各タスクは1つの検証可能な変更単位にしてください。
```

これを入れるだけでも、軽量モデルの迷走を減らせます。

## 最も効率のよい「安価生成＋高級レビュー」

すべてを高級モデルで生成するのではなく、次の流れが有効です。

### 仕様

1. 人間が要件の素材を書く
2. Lunaが`spec.md`を生成
3. TerraまたはGPT-5.4が「重大な不足だけ」をレビュー
4. Lunaが指摘箇所だけ修正

### 計画

1. 上位モデルが重要な技術判断だけ行う
2. Lunaがテンプレートへ展開
3. 上位モデルが最大5件の重大問題だけ指摘
4. Lunaが差分修正

### タスク

1. Lunaが`tasks.md`を生成
2. LunaまたはGPT-5.4 miniが整合確認
3. 問題がある箇所だけ修正

高級モデルに次のように依頼するのがポイントです。

```
文書全体を書き直さないでください。
実装失敗につながる重大な問題を最大5件だけ指摘してください。
問題がなければPASSと回答してください。
```

高級モデルによる長い再生成を避け、**短い品質ゲートとして利用する**方法です。

## Spec Kit自体の出力を軽量化する

現在のCopilotはモデルと入出力トークン数によってAIクレジットが決まり、1 AIクレジットは0.01ドル相当です。Lunaは100万トークン当たり入力0.20ドル・出力1.20ドル、GPT-5.4は入力2.50ドル・出力15ドルなので、同じトークン量なら約12.5倍の差があります。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/copilot/reference/copilot-billing/models-and-pricing)

ただし、Lunaで大量の文書を生成すると、当然その分は消費します。次の削減策が有効です。

### 不要な成果物を作らない

Fortran解析用CLIであれば、毎回以下が必要とは限りません。

* UI仕様
* Web API契約
* 詳細なデータモデル文書
* 長いquickstart
* 技術選定の一般論
* 全機能を対象にした包括的チェックリスト

Spec Kitはpresetによりテンプレートやコマンドを上書きできるため、プロジェクト向けに簡略版を用意できます。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/github/spec-kit/blob/main/docs/reference/overview.md?utm_source=chatgpt.com)

### `NEEDS CLARIFICATION`を事前に減らす

標準の`plan`では、未確定事項ごとに調査タスクを作ります。そのため、次を事前入力すると節約できます。

```
Language: Python 3.12
Package manager: uv
Parser: fparser2
Testing: pytest + gfortran compilation
Target platform: WSL Ubuntu
Architecture: existing src/ layoutを維持
New dependencies: 原則追加しない
External API: なし
Performance requirement: 今回は対象外
```

調査対象が減り、Agentの検索・読取り・文書生成が減ります。

### 小さな機能単位に限定する

「COMMON解析機能全体」ではなく、

* COMMONブロックの収集
* 宣言型の解決
* 不整合検出
* JSONレポート
* 修正候補生成

のように分けます。ただし、各小機能について毎回完全なSpec Kit一式を生成すると逆に高コストです。

目安としては、

* 小修正：直接タスク
* 中規模機能：簡易spec＋tasks
* 設計判断を伴う機能：spec＋plan＋tasks
* 大規模・高リスク変更：完全なSDD

と段階を変えるのが合理的です。

### 長い会話を引き継がない

Copilotは入力、出力、キャッシュ済み入力のトークンに応じて課金されます。キャッシュ済み入力は通常入力より安価ですが、不要な過去会話や大量のファイルが毎回コンテキストに入る状態は避けるべきです。また、Lunaは20万トークン、GPT-5.4は27.2万トークンを超えると長文コンテキスト料金になります。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/copilot/reference/copilot-billing/models-and-pricing)

したがって、

* 機能単位で会話を分ける
* 引き継ぎは`spec.md`、`plan.md`、`tasks.md`に集約
* 現在のタスクに必要なファイルだけ指定
* 過去の議論をチャット履歴から再読させない

という運用が有効です。

## あなた向けの推奨モデル分担

Fortranリファクタリング・Python移行ツールなら、次の構成が現実的です。

| 作業 | 推奨モデル |
| --- | --- |
| プロジェクトconstitutionの初期作成 | GPT-5.4 |
| 具体的な要望からspec生成 | Luna |
| 曖昧性・安全性レビュー | TerraまたはGPT-5.4 mini |
| COMMON・CALL・型解析方式の決定 | GPT-5.4 |
| 決定済み方式からplan文書生成 | Luna |
| tasks生成 | Luna |
| 複数ファイル実装 | Luna |
| 原因不明の失敗調査 | TerraまたはCodex |
| 段階完了時の設計レビュー | GPT-5.4、短い指摘のみ |

つまり、**高級モデルを各工程の「生成担当」にするのではなく、重要な意思決定と短い品質ゲートに限定する**のが最も節約効果が高いです。

特にSpec Kitでは、

> **specify＝要求が具体的ならLuna**  
> **plan＝設計済みならLuna、未決定なら上位モデル**  
> **tasks＝基本的にLuna**

という使い分けが適しています。

* * *

**You:**

「COMMON・CALL・型解析方式の決定」の部分をGPT-5.4にすればよいとのことですが、この段階ではcopilotのエージェントモードとしては何を用いれば良いの？公式のagentモードやplanモード？
前の「曖昧性・安全性レビュー」は./spekit.clarify、次の「決定済み方式からplan文書生成」が./speckit.planですよね？

* * *

**ChatGPT:**

はい。ただし、**前の私の工程分けはSpec Kitの実際の役割に照らすと、少し細かく分けすぎていました。**

結論から言うと、通常は次のようにします。

> **COMMON・CALL・型解析方式の決定は、GPT-5.4を選んだ上で`/speckit.plan`を実行する**  
> Copilotの組み込み**Planモードは併用せず、AgentモードまたはSpec Kitの `speckit.plan`エージェントを使う**

## Copilot Planモードと`/speckit.plan`は別物

名前が似ていますが、役割が重複しています。

### Copilotの組み込みPlanモード

CopilotのPlanモードは、コードを変更せずにリポジトリを調査し、詳細な実装計画を作るためのモードです。読み取り専用ツールでコードベースを調査し、計画を作った後、Agentモードへ引き渡します。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/copilot/using-github-copilot/asking-github-copilot-questions-in-your-ide)

### Spec Kitの`/speckit.plan`

`/speckit.plan`は、`spec.md`を基に技術設計を行い、以下を生成します。

* `plan.md`
* `research.md`
* `data-model.md`
* `contracts/`
* `quickstart.md`

特に`research.md`では、技術上の未決定事項について、

* Decision
* Rationale
* Alternatives considered

を記録するよう標準プロンプトで指定されています。つまり、**解析方式の比較・決定自体が`/speckit.plan`の仕事に含まれています**。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md?plain=1&utm_source=chatgpt.com)

したがって、

```
Copilot Planモード
    ↓
/speckit.plan
```

と連続で実行すると、計画作業を二重に行う可能性があります。

## おすすめの対応関係

| 工程 | Spec Kit | Copilot側 | モデル |
| --- | --- | --- | --- |
| 機能要求の整理 | `/speckit.specify` | Spec Kitエージェント | Luna |
| 要求の曖昧さ解消 | `/speckit.clarify` | Spec Kitエージェント | Lunaまたは中位モデル |
| 解析方式の調査・決定 | `/speckit.plan` | **Spec Kitのplanエージェント** | **GPT-5.4** |
| 要求品質の確認 | `/speckit.checklist` | Spec Kitエージェント | Luna |
| 実装タスク分解 | `/speckit.tasks` | Spec Kitエージェント | Luna |
| 文書間の矛盾確認 | `/speckit.analyze` | Spec Kitエージェント | Luna |
| 実装 | `/speckit.implement` | Agent | Luna |

Spec Kit公式でも、`/speckit.clarify`は計画前の要求曖昧性解消、`/speckit.plan`は技術スタック・アーキテクチャ・技術制約を扱う工程とされています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Docs+1](https://github.github.com/spec-kit/reference/agentic-sdd.html?utm_source=chatgpt.com)

## 実際の入力例

Fortranツールなら、GPT-5.4を選び、`/speckit.plan`に次のように入力するのが自然です。

```
/speckit.plan

以下の技術方式を調査・比較し、実装方式を決定してください。

対象:
- COMMON文の検出と不整合判定
- CALL文と仮引数・実引数の対応解析
- 暗黙型と明示宣言の型解決

比較対象:
- 正規表現・字句解析による方式
- fparser2 ASTによる方式
- 両者を組み合わせた段階的方式

評価基準:
- 固定形式と自由形式への対応
- 継続行への対応
- 誤検出と検出漏れ
- 解析不能なprogram unitへのフォールバック
- ソース位置情報の保持
- 自動修正時の安全性
- 合成fixtureでの検証可能性

research.mdには、各論点について以下を記録してください。
- 採用方式
- 採用理由
- 却下した代替方式
- 適用限界
- フォールバック条件

この段階ではソースコードを実装しないでください。
```

これは`/speckit.plan`のPhase 0研究にそのまま対応します。

## `/speckit.clarify`の役割

ご認識どおり、前段の曖昧性解消は`/speckit.clarify`です。ただし、ここで扱うのは主に**要求の曖昧さ**です。

例えば、

* COMMON不整合を検出するだけか、自動修正もするか
* 型が確定できない場合に停止するか、候補を提示するか
* どの修正を人間の承認対象にするか
* 数値回帰に失敗した場合にどう扱うか
* 解析不能なprogram unitを全体失敗にするか、部分成功にするか

などです。

```
/speckit.clarify

以下を重点的に明確化してください。

- 自動修正と要承認修正の境界
- 型を一意に決定できない場合の挙動
- COMMON不整合が複数ファイルにまたがる場合の扱い
- 解析不能なprogram unitがある場合の部分成功条件
- gfortranコンパイルおよび数値回帰の合格条件
```

`/speckit.clarify`は質問を通して未規定部分を解消し、その回答を`spec.md`へ反映する工程です。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Docs+1](https://github.github.com/spec-kit/quickstart.html)

## 「安全性レビュー」はclarifyだけではない

ここは少し補足が必要です。

* `/speckit.clarify`：安全要件を明確化する
* `/speckit.checklist`：安全要件が漏れなく定義されているか確認する
* `/speckit.analyze`：安全要件がplan・tasksまで反映されているか確認する

という三段構えが適切です。

例えば、

```
/speckit.checklist

Fortran自動変換の安全性に重点を置いてください。

- すべての自動修正に変換前後の検証方法があるか
- 型を確定できない場合に自動修正しないことが明記されているか
- COMMON変数の順序・型・個数を検証する要件があるか
- コンパイル回帰と数値回帰の両方が定義されているか
- 元コードを保持してロールバックできるか
```

Spec Kitでは、`checklist`は要求品質を検証する「requirements向けの単体テスト」、`analyze`は`spec.md`、`plan.md`、`tasks.md`間の矛盾や欠落を検出する読み取り専用工程とされています。[![](https://www.google.com/s2/favicons?domain=https://github.github.com&sz=128)GitHub Docs](https://github.github.com/spec-kit/quickstart.html)

## Copilot組み込みPlanモードを使うケース

組み込みPlanモードを使うなら、次のような**例外的なケース**です。

* `/speckit.plan`を実行する前に、大規模コードベースの現状だけ調査したい
* 既存実装が複雑で、どのモジュールが関係するか分からない
* Spec Kitの計画とは別に、第三者的な方式レビューを行いたい
* 既存の`plan.md`を変更せず、別の視点から批評したい

この場合は、

```
組み込みPlanモード + GPT-5.4
```

で調査し、出力を「実装計画」ではなく、短い技術判断メモに限定します。

```
実装計画は作成しないでください。
COMMON・CALL・型解析の技術方式だけを比較してください。

出力:
1. 現状構造
2. 候補方式
3. 推奨方式
4. リスク
5. /speckit.planへ渡す確定事項

コードやMarkdownファイルは変更しないでください。
```

その後、確定事項をLunaの`/speckit.plan`へ渡すことはできます。ただし、**GPT-5.4による調査とLunaによる計画生成の二回分のコンテキスト読み込みが発生する**ため、必ずしも最安ではありません。

## 最も実用的な節約構成

あなたのプロジェクトでは、次が最も分かりやすいと思います。

```
/speckit.specify   → Luna
/speckit.clarify   → Luna
/speckit.plan      → GPT-5.4
/speckit.checklist → Luna
/speckit.tasks     → Luna
/speckit.analyze   → Luna
/speckit.implement → Luna
```

つまり、**高級モデルを使うのは`/speckit.plan`の一回に集約する**形です。

なお、コマンド表記は通常、

```
/speckit.clarify
/speckit.plan
```

であり、`./speckit.clarify`ではありません。

要点をまとめると、**解析方式をまだ決めていないなら、GPT-5.4でSpec Kitの`/speckit.plan`を実行するのが第一候補です。組み込みPlanモードを先に使う必要はありません。**

* * *

**You:**

/speckit.planは複数のmdファイルを生成するんですよね？出力のクレジットが高いことを考えると/speckit.planはなるべく形容モデルにした方が良く、そのプロンプトを詳細にした方が良いというのがあなたの意図なのかと思っていましたが違うのでしょうか？そしてそのプロンプトの詳細化方法を知りたかったのです。

* * *

**ChatGPT:**

はい。**ご理解のとおりです。**  
前の回答で私が`/speckit.plan → GPT-5.4`としたのは、クレジット節約という目的には適切ではありませんでした。

本来意図していたのは、次の分離です。

> **GPT-5.4：少量の技術判断だけを行う**  
> **Luna：確定済みの判断を受け取り、`/speckit.plan`で複数の成果物へ展開する**

公式の`/speckit.plan`は、`plan.md`だけではなく、`research.md`、`data-model.md`、`quickstart.md`を生成し、外部インターフェースがあれば`contracts/`も生成します。また、未確定事項、依存技術、統合先があると、それぞれについて調査タスクを作る設計です。したがって、**出力の多い`/speckit.plan`を軽量モデルに担当させ、事前に未確定事項を潰す**のは合理的です。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md?plain=1)

## 推奨フロー

```
/speckit.specify
    ↓ Luna

/speckit.clarify
    ↓ Lunaまたは中位モデル

技術方式の決定
    ↓ GPT-5.4、短い出力

/speckit.plan
    ↓ Luna、詳細な確定事項を入力

/speckit.tasks
    ↓ Luna

/speckit.analyze
    ↓ Lunaまたは必要時だけ上位モデル

/speckit.implement
    ↓ Luna
```

## GPT-5.4は何モードで使うか

この「技術方式の決定」には、原則として**CopilotのPlanモードを使わない方がよい**です。

CopilotのPlanモード自体が、リポジトリを調査して詳細な実装計画を生成するモードだからです。`/speckit.plan`と役割が重なり、長い計画を二度生成することになります。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/copilot/using-github-copilot/asking-github-copilot-questions-in-your-ide)

推奨は次のどちらかです。

### コード調査が少ない場合

**Askモード＋GPT-5.4**

`spec.md`、`constitution.md`、必要な設計情報だけを参照させ、短い「技術決定メモ」を作らせます。

### リポジトリ調査が必要な場合

**読み取り専用のカスタムエージェント＋GPT-5.4**

またはAgentモードで、以下を明示します。

```
コードや仕様ファイルを変更しないでください。
実装計画やタスクリストは生成しないでください。
指定した技術論点の判断だけを行ってください。
```

通常のAgentモードはファイル編集やコマンド実行も行えるため、可能なら読み取り専用カスタムエージェントの方が安全です。Copilotの通常Agentモードは自律的な編集向け、Planモードは詳細計画向けと公式に区別されています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/copilot/using-github-copilot/asking-github-copilot-questions-in-your-ide)

## GPT-5.4に作らせるもの

GPT-5.4には、長い`plan.md`を作らせるのではなく、次のような小さな「Decision Packet」を作らせます。

```
1. 採用方式
2. 採用理由
3. 適用範囲
4. フォールバック方式
5. 安全停止条件
6. 変更してはいけない制約
7. 未解決事項
```

出力先は例えば次です。

```
specs/xxx-feature/technical-decisions.md
```

あるいは、出力をチャット内に留めて、確認後に`/speckit.plan`の引数へ直接貼っても構いません。

## GPT-5.4への依頼例

Fortran解析ツールの場合は、次の程度に絞ります。

```
spec.mdとconstitution.md、および以下の関連ファイルだけを調査し、
COMMON・CALL・型解析の技術方式を決定してください。

参照対象:
- src/parser/
- src/analysis/
- tests/fixtures/
- pyproject.toml

この段階の目的:
技術選択を確定し、後続の軽量モデルによる
/speckit.planに入力できるDecision Packetを作ること。

決定する論点:
1. COMMON解析の主方式
2. CALL解析の主方式
3. 暗黙型を含む型解決方式
4. fparser2で解析不能な場合のフォールバック
5. ソース位置情報の保持方式
6. 自動修正を禁止する条件
7. コンパイル・数値回帰の検証境界

制約:
- Python 3.12
- uv
- fparser2
- pytest
- gfortran
- 新規依存は原則追加しない
- 実コードは利用できない
- 合成fixtureで検証する
- 既存の公開APIを維持する
- この段階ではコードを変更しない
- 実装計画、tasks.md、テストコードは作らない

出力形式:
各論点について以下だけを記載する。
- Decision
- Rationale
- Fallback
- Safety stop condition

全体を1200語以内にしてください。
重大な未解決事項がある場合だけ最後に列挙してください。
```

この工程では、GPT-5.4の出力を意図的に短くします。

## Lunaへ渡す`/speckit.plan`プロンプト

その後、Lunaを選択して`/speckit.plan`を実行します。

重要なのは、単に「添付のDecision Packetに従ってください」だけにせず、**確定事項・非対象・探索禁止範囲を明示すること**です。

```
/speckit.plan

spec.md、constitution.md、および
specs/xxx-feature/technical-decisions.mdに基づいて
技術実装計画を生成してください。

technical-decisions.mdに記載されたDecisionは確定事項です。
代替案の再比較や技術選定のやり直しは行わないでください。

【確定技術】

言語・環境:
- Python 3.12
- uv
- pytest
- gfortran
- fparser2

COMMON解析:
- fparser2 ASTを主方式とする
- 固定形式の前処理では既存normalizerを利用する
- 正規表現単独で意味解析しない
- AST解析不能時は自動修正せず、診断結果だけを出力する

CALL解析:
- program unit単位でシンボル情報を収集する
- call siteとprocedure definitionを明示的に対応付ける
- 解決不能な外部procedureはunknownとして保持する
- 推測による引数型の確定は行わない

型解析:
- 明示宣言を最優先する
- IMPLICIT規則を次に適用する
- COMMON内の対応宣言を照合する
- 型を一意に確定できない場合は要承認候補にする

安全条件:
- 型、順序、配列形状のいずれかが不明なら自動修正しない
- parser failureをテキスト置換で隠さない
- 変換前後でgfortranコンパイルを行う
- 実行可能fixtureでは数値回帰も行う
- 既存公開APIを変更しない

【対象範囲】

今回含む:
- COMMON情報収集
- CALL関係収集
- 型解決結果の統合
- 診断モデル
- JSONレポート
- 合成fixtureによる検証

今回含まない:
- COMMONのmoduleへの自動変換
- Pythonコード生成
- GUI
- 並列解析
- 性能最適化
- 新規外部依存の追加

【成果物の制約】

- plan.mdは実装境界、コンポーネント責務、処理フローを中心にする
- research.mdには確定済みDecisionを簡潔に転記する
- 決定済み技術について一般的なベストプラクティス調査を繰り返さない
- data-model.mdには今回追加または変更する解析モデルだけを記載する
- contracts/にはCLIまたはJSON形式に変更がある場合だけ記載する
- quickstart.mdは検証コマンドと期待結果に限定する
- 同じ説明を複数ファイルへ重複記載しない
- 完全な実装コードやテストコードは生成しない
- 各文書は必要最小限の長さにする

新しい技術判断が必要になった場合:
- 独自に決定しない
- NEEDS CLARIFICATIONとして記録する
- その論点を最大5件、完了報告に列挙する
```

## 詳細化で重要な項目

`/speckit.plan`のプロンプトは、長くすればよいわけではありません。次の情報を埋めることが重要です。

| 項目 | 節約につながる理由 |
| --- | --- |
| 技術スタック | 技術選定の調査を減らす |
| 採用方式 | 代替案の比較を減らす |
| フォールバック | 例外時の独自判断を防ぐ |
| 新規依存の可否 | パッケージ調査を減らす |
| 対象範囲 | 不要な設計を防ぐ |
| 非対象範囲 | 機能の膨張を防ぐ |
| 外部インターフェース | 不要なcontracts生成を抑える |
| データモデル対象 | data-model.mdの肥大化を防ぐ |
| 検証方法 | quickstart.mdの推測を減らす |
| 未確定時の停止条件 | Lunaの迷走を防ぐ |
| 文書ごとの内容制限 | 重複出力を減らす |

特に大事なのは、以下の3点です。

```
確定事項は再検討しない
未確定事項は独自判断しない
成果物間で説明を重複させない
```

## 標準Spec Kitでは削れない出力もある

現在の標準`/speckit.plan`では、`research.md`、`data-model.md`、`quickstart.md`の生成はワークフローに組み込まれています。`contracts/`のみ、外部インターフェースがなければスキップできます。[![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/github/spec-kit/blob/main/templates/commands/plan.md?plain=1)

したがって、プロンプトだけでできるのは、

* 内容を簡潔にする
* 調査を減らす
* 重複を減らす
* 不要なcontractsを作らない
* 未確定事項の展開を減らす

ところまでです。

さらに削減したければ、Spec Kitの`plan.md`コマンドテンプレートをカスタマイズし、例えば小規模CLI向けに、

```
plan.md
research.md
quickstart.md
```

だけを生成する軽量版コマンドを作る方が効果的です。ただし、まずは標準版を使い、**高級モデルによるDecision Packet＋Lunaによる`/speckit.plan`**でどの程度削減できるか確認するのが安全です。

したがって、修正した推奨は次のとおりです。

> **`/speckit.plan`そのものはLuna**  
> **その前にGPT-5.4をAskモードまたは読み取り専用エージェントで使い、短い技術判断だけを確定する**  
> **確定事項を構造化した詳細プロンプトとしてLunaへ渡す**

これが、出力単価と計画品質の両方を考えた分担です。

* * *

**You:**

Askモードはリポジトリ調査可能でかつ何も生成しないという認識であっていますか？ちなみにネット上の情報も確認してどのパッケージを用いるべきかなどの判断も柔軟にしてくれるのでしょうか？

* * *

**ChatGPT:**

概ね合っていますが、正確には次の理解です。

> **Askモードはリポジトリを読み取り・調査できるが、ワークスペースのファイルを自律的には変更しない。**  
> ただし、チャット内には説明文、コード例、設計案などを生成します。

## Askモードでできること

Askモードは、コードベースの理解、技術概念の質問、アイデア探索に最適化されています。現在のVS Codeでは、必要に応じて以下の読み取り系ツールを使ってリポジトリを調査できます。

* セマンティック検索
* 文字列検索・grep
* ファイル名検索
* 定義・参照箇所の追跡
* ディレクトリ一覧の確認
* ファイル内容の読み取り

明示的に`#codebase`を付けなくても、必要と判断すれば自動的に検索します。したがって、「現在のCOMMON解析処理はどのファイルに分散しているか」「既存の型解決処理を再利用できるか」といった調査はAskモードで可能です。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/ja/copilot/how-tos/chat-with-copilot/chat-in-ide)

一方、Askモードも**回答自体は生成します**。回答には文章、コードブロック、ファイルツリー、URIなどが含まれることがありますが、Agentモードのように複数ファイルを直接編集してテストまで繰り返すモードではありません。コードブロックを手動で挿入する操作が提示されることはあります。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/ja/copilot/how-tos/chat-with-copilot/chat-in-ide)

したがって、今回の用途では、

```
Ask + GPT-5.4
→ リポジトリを調査
→ 短い技術判断をチャットに出力
→ 内容を確認
→ Lunaの /speckit.plan に渡す
```

という使い方が適しています。

## インターネット検索も可能か

**可能ですが、初期状態では無効です。**

GitHubのCopilot設定で「Copilot access to Bing」を有効にすると、Copilot ChatがBingを使用して、質問に関連するインターネット情報を検索できます。GitHubは、新しい技術や専門性の高いテーマを調べる用途に有効だと説明しています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/copilot/how-tos/manage-your-account/managing-copilot-policies-as-an-individual-subscriber)

設定場所はGitHub上の次の項目です。

```
GitHub
→ Settings
→ Copilot
→ Copilot access to Bing
→ Enabled
```

会社からCopilot BusinessまたはEnterpriseライセンスを付与されている場合は、組織ポリシーによって使用できない可能性があります。

さらにVS Codeでは、特定のWebページを参照させる`#fetch`や、GitHub上の別リポジトリを検索する`#githubRepo`、`#githubTextSearch`も利用できます。公式ドキュメントや上流ライブラリの実装を確認する用途に向いています。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/agents/reference/workspace-context)

## パッケージ選定もできるか

はい。次の情報を組み合わせた判断が可能です。

* 現在のリポジトリ構成
* `pyproject.toml`などの既存依存関係
* PythonやFortranの対象バージョン
* Web上のパッケージ公式ドキュメント
* GitHub上の実装・リリース状況
* ライセンス
* メンテナンス状況
* 既存コードへの導入負荷

例えば、正規表現、fparser2、tree-sitter-fortran、独自字句解析のどれを採用するかを、リポジトリの現状と外部情報の両方から比較させられます。

ただし、単に、

```
最適なパッケージを選んでください
```

と依頼すると、モデルの既知情報だけで回答したり、評価基準が曖昧になったりします。次のように、Web調査を明示した方が確実です。

```
Askモードで、コードやファイルを変更せずに調査してください。

目的:
COMMON・CALL・暗黙型解析で利用する技術方式を決定する。

リポジトリ調査範囲:
- pyproject.toml
- src/parser/
- src/analysis/
- tests/fixtures/
- .github/copilot-instructions.md

外部調査:
Web検索を使用し、候補パッケージの最新の公式ドキュメント、
公式GitHubリポジトリ、直近のリリース状況を確認してください。

候補:
- fparser2
- tree-sitter-fortran
- 正規表現と独自字句解析
- 上記のハイブリッド

評価基準:
- Fortran 77固定形式
- 継続行
- IMPLICIT規則
- COMMON文
- CALL文と引数対応
- ソース位置情報
- 不完全なコードへの耐性
- 商用利用可能なライセンス
- メンテナンス状況
- 新規依存の追加コスト
- 合成fixtureでの検証容易性

制約:
- 実コードは利用できない
- Python 3.12、uv、pytest、gfortranを使用
- 既存のfparser2実装を可能な限り再利用する
- この段階では実装計画を作らない
- ファイルを変更しない

出力:
1. 現行実装の要約
2. 候補比較表
3. 推奨方式
4. 採用理由
5. フォールバック方式
6. 安全停止条件
7. /speckit.planへ渡す確定事項

出力は1500語以内とし、
参照した外部情報には出典を付けてください。
```

## Askモードだけでは不十分なこと

Askモードによるパッケージ選定は、**調査と判断**には向いていますが、実際に導入して成立するかまでは保証しません。

例えば以下は別途検証が必要です。

* 指定したPythonバージョンでインストールできるか
* WSLのオフライン環境でwheelを用意できるか
* 実際のFortran fixtureを解析できるか
* 既存テストに影響しないか
* 性能やメモリ消費が許容範囲か

これらは、後続の`/speckit.plan`で「技術スパイク」や検証タスクとして定義し、LunaのAgentモードで実行するのがよいです。

例えば、Askで「tree-sitter-fortranが有望」と判断されても、即採用確定ではなく、

```
採用候補:
tree-sitter-fortran

確定前に必要な検証:
- 固定形式継続行fixtureを解析
- COMMON文のノード構造を確認
- エラー回復時のソース位置情報を確認
- fparser2との解析結果を比較

検証成功条件:
上記fixtureで必要な情報を欠損なく取得できること
```

のような条件付きDecisionにしても構いません。

## クレジット節約の観点

Askモードでも、リポジトリ検索やWeb検索で大量の情報を読み込めばクレジットは消費します。生成ファイル、ログ、ビルド成果物などを検索対象から除外すると、検索結果のトークン数を抑えられます。VS Code公式も、`.gitignore`、`files.exclude`、`search.exclude`による検索範囲の制限が、AIクレジット消費の削減に役立つと説明しています。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/agents/reference/workspace-context)

したがって、技術判断工程では次が有効です。

```
調査対象:
- src/parser/
- src/analysis/
- tests/fixtures/
- pyproject.toml

調査対象外:
- build/
- dist/
- .venv/
- htmlcov/
- generated/
- 大規模なテスト出力
```

また、外部調査も「関連情報を広く調べる」ではなく、

```
公式ドキュメントと公式GitHubリポジトリに限定する
候補は最大4件
評価基準は10項目以内
出力は1500語以内
```

と制限した方がよいです。

結論として、今回の工程には**Askモード＋GPT-5.4**が適しています。ただし役割は「何も生成しない調査モード」ではなく、**リポジトリとWebを読み取り、短い技術判断をチャットに生成するモード**です。その結果を確定事項として、**Lunaの`/speckit.plan`に詳細入力する**のが、品質とクレジット節約のバランスがよい構成です。

* * *

**ChatGPT:**

以下は、**Spec Kitを使わずに、社内AIチャットとGitHub Copilotを役割分担するためのテンプレート集**です。高性能モデルの出力を短くし、軽量モデルに文書展開と実装を任せる構成にしています。

# GitHub Copilotクレジット節約用

# 開発プロンプトテンプレート集

## 1. 基本方針

各ツールの役割を次のように分ける。

| 工程 | 使用先 | モード | モデルの目安 |
| --- | --- | --- | --- |
| 要求整理・論点洗い出し | 社内AIチャット | 通常チャット | 利用制限が少ない高性能モデル |
| 要求・安全性レビュー | 社内AIチャット | 通常チャット | 高性能モデル |
| リポジトリの現状調査 | Copilot | Ask | 探索向け中軽量モデル |
| 重要な技術判断 | Copilot | Ask | 高性能モデル |
| 実装計画への展開 | Copilot | Plan | 軽量モデル |
| 計画済みの実装 | Copilot | Agent | 軽量モデル |
| 原因不明のデバッグ | Copilot | Ask／Agent | 中上位モデル |
| コードに基づくレビュー | Copilot | Ask | 中位モデル |
| 要件・進め方のレビュー | 社内AIチャット | 通常チャット | 高性能モデル |

原則は次のとおり。

```
高性能モデル:
判断、比較、重大なリスクの発見だけを担当する。

軽量モデル:
決定済み内容の文書化、タスク分解、実装、定型レビューを担当する。```

---

# 2. 推奨する管理ファイル

機能ごとに以下のファイルを用意する。

```
docs/current/
├── feature-brief.md
├── repository-findings.md
├── technical-decisions.md
├── implementation-plan.md
├── implementation-log.md
└── review-pack.md```

役割は次のとおり。

| ファイル | 内容 |
| --- | --- |
| `feature-brief.md` | 目的、要件、対象範囲、受入条件 |
| `repository-findings.md` | 現在のコード構造と関連箇所 |
| `technical-decisions.md` | 確定した技術判断 |
| `implementation-plan.md` | ファイル単位の実装計画 |
| `implementation-log.md` | 実装結果、テスト結果、計画との差異 |
| `review-pack.md` | 社内AIや別セッションへ渡す要約 |

すべての工程で長いチャット履歴を読み直させず、これらのMarkdownを引き継ぎ情報として使う。

---

# 3. 共通コンテキストテンプレート

各プロンプトの冒頭に必要な部分だけ付ける。

```
【プロジェクト】
<プロジェクト名>

【今回の目的】
<今回追加・修正したい機能>

【現在の状態】
<実装済みの内容、既知の問題>

【技術環境】
- 言語:
- パッケージ管理:
- 主要ライブラリ:
- テスト:
- コンパイラ:
- 対象OS:

【重要な制約】
- <例: 既存公開APIを維持する>
- <例: 新規依存を原則追加しない>
- <例: 実コードは提供できず、合成fixtureで検証する>
- <例: 数値結果を変えない>

【今回の対象範囲】
- 

【今回の非対象範囲】
- 

【確定済み事項】
- 

【未確定事項】
- ```

すべてを毎回貼らず、今回の処理に必要な項目だけ残す。

---

# 4. 社内AIチャット：要求整理

## 使用タイミング

* 開発したい内容がまだ文章化されていない
* 要望が思いつきの列挙になっている
* Copilotにリポジトリを読ませる前
* 技術方式ではなく「何を実現すべきか」を整理したい

## プロンプト

```
以下の開発要望を、実装前の要求定義として整理してください。

【開発したい内容】
<やりたいことを自由に記載>

【背景】
<なぜ必要なのか>

【現在困っていること】
<現在の問題や失敗例>

【既知の制約】
- 
- 

以下の構成で出力してください。

1. 背景
2. 目的
3. 想定する利用場面
4. 機能要件
5. 安全性・信頼性要件
6. 対象範囲
7. 非対象範囲
8. 正常系
9. 異常系
10. 受入条件
11. 未確定事項
12. 人間の承認が必要な判断

注意:
- まだ実装方式や使用ライブラリを決定しないでください。
- 実装コードは生成しないでください。
- 要求と実装方法を混同しないでください。
- 不明な点を勝手に補完せず、「未確定事項」に分類してください。
- 重複した要件は統合してください。
- 出力は1500語以内にしてください。```

出力は`docs/current/feature-brief.md`へ保存する。

---

# 5. 社内AIチャット：曖昧性・安全性レビュー

## 使用タイミング

* `feature-brief.md`を作成した後
* 自動修正、数値計算、データ変換など失敗時の影響が大きい機能
* Copilotの技術調査へ進む前

## プロンプト

```
以下の要求定義をレビューしてください。

【要求定義】
<feature-brief.mdの内容>

目的は文章をきれいにすることではなく、
実装後の手戻りや危険な自動処理につながる曖昧さを発見することです。

以下の観点で確認してください。

1. 成功条件が測定可能か
2. 失敗時の挙動が定義されているか
3. 自動処理を禁止すべき条件があるか
4. 人間の承認が必要な条件が明確か
5. 入力データが不完全な場合の扱い
6. 後方互換性
7. ロールバック方法
8. テスト可能性
9. 対象範囲の膨張
10. 要求間の矛盾

出力形式:

## 重大な不足
実装前に必ず解消すべき事項。最大5件。

## 軽微な不足
後から修正可能な事項。最大5件。

## 推奨する追記
要求定義へそのまま追加できる短い文章。

## 判定
- READY
- READY WITH CONDITIONS
- NOT READY

要求定義全体を書き直さないでください。
重大な問題がなければ、無理に問題を作らないでください。```

---

# 6. Copilot Ask：リポジトリ現状調査

## 推奨

* モード：Ask
* モデル：探索向けの中軽量モデル
* ファイル変更：禁止
* Web検索：原則不要

## プロンプト

```
Askモードで、読み取り専用のリポジトリ調査を行ってください。

【今回の要求】
docs/current/feature-brief.mdを参照してください。

【調査対象】
- <対象ディレクトリ>
- <対象ディレクトリ>
- pyproject.toml
- tests/

【必要に応じて参照してよい範囲】
- 

【調査対象外】
- .venv/
- build/
- dist/
- generated/
- 大規模なログや結果ファイル
- 今回の機能と無関係なディレクトリ

調査目的:
今回の機能に関係する現在のコード構造を把握し、
技術判断が必要な論点を特定すること。

出力してください。

1. 関連ファイル一覧
2. 現在の処理フロー
3. 主要なクラス・関数・データ構造
4. 再利用できる既存処理
5. 変更が必要そうな箇所
6. 既存テストとfixture
7. 後方互換性上の注意
8. 技術判断が必要な論点
9. 追加調査が必要な箇所

制約:
- ファイルを変更しないでください。
- 実装計画はまだ作成しないでください。
- コードは生成しないでください。
- 調査対象を無制限に広げないでください。
- 推測と、コードから確認できた事実を区別してください。
- 出力は1200語以内にしてください。```

結果を`docs/current/repository-findings.md`へ保存する。

---

# 7. Copilot Ask：外部パッケージ調査

## 使用タイミング

* 新しいパッケージを導入する可能性がある
* 既存ライブラリで対応可能か判断したい
* 最新の公式仕様や対応バージョンを確認する必要がある

## 推奨

* モード：Ask
* モデル：高性能モデル
* Web検索：利用可能なら有効化
* 出力：短い比較と決定候補のみ

## プロンプト

```
Askモードで、リポジトリと外部情報を読み取り専用で調査してください。

【目的】
<解決したい技術課題>

【現在の実装】
- docs/current/feature-brief.md
- docs/current/repository-findings.md
- pyproject.toml
- <関連ファイル>

【比較候補】
- <候補1>
- <候補2>
- <候補3>
- 既存実装の拡張

【評価基準】
- 必要機能への対応
- 現在の言語・ランタイムへの対応
- 既存コードへの導入負荷
- APIの安定性
- メンテナンス状況
- ライセンス
- オフライン導入可能性
- テスト容易性
- 不完全な入力への耐性
- ソース位置情報など必要な付加情報

外部調査では、原則として次を優先してください。

1. 公式ドキュメント
2. 公式GitHubリポジトリ
3. 公式リリースノート
4. 原著論文または一次情報

制約:
- ファイルを変更しないでください。
- 実装コードを生成しないでください。
- 候補を必要以上に増やさないでください。
- Web情報の確認日と出典を示してください。
- メンテナンス状況を推測だけで断定しないでください。
- 導入可能性と実コードでの動作確認を区別してください。

出力:

1. 現在の実装で不足している点
2. 候補比較表
3. 推奨候補
4. 推奨理由
5. 採用前に必要な技術スパイク
6. 不採用または保留とする候補
7. 後続の技術判断で確定すべき事項

全体を1500語以内にしてください。```

---

# 8. Copilot Ask：重要な技術判断

## 使用タイミング

* 間違えると後工程を作り直す判断
* アーキテクチャ、データ構造、解析方式
* 自動処理と要承認処理の境界
* フォールバック方式

## 推奨

* モード：Ask
* モデル：高性能モデル
* 出力：短いDecision Packet
* Planモードはまだ使わない

## プロンプト

```
Askモードで、以下の技術判断だけを行ってください。

【参照資料】
- docs/current/feature-brief.md
- docs/current/repository-findings.md
- <外部パッケージ調査結果があれば記載>
- <関連する実装ファイル>

【今回決定する論点】
1. <論点1>
2. <論点2>
3. <論点3>
4. <論点4>

【評価基準】
- 正確性
- 既存コードとの整合性
- 検証可能性
- 後方互換性
- 失敗時の安全性
- 保守性
- 今回の対象範囲

【固定制約】
- 
- 
- 

【この段階で行わないこと】
- 実装計画の作成
- タスクリストの作成
- コード変更
- テストコード生成
- 一般的なベストプラクティスの長い説明

各論点について次の形式で回答してください。

## <論点名>

Decision:
採用する方式。

Rationale:
採用理由。最大5項目。

Alternatives rejected:
不採用とした方式と短い理由。

Fallback:
主方式が利用できない場合の扱い。

Safety stop condition:
自動処理や実装を停止し、人間へ確認すべき条件。

Required validation:
採用確定または実装完了のために必要な検証。

最後に以下を付けてください。

## 未解決事項
重大なものだけ。最大5件。

## Planへの入力用確定事項
後続のPlanモードへそのまま渡せる箇条書き。

全体を1200語以内にしてください。```

結果を`docs/current/technical-decisions.md`へ保存する。

---

# 9. Copilot Plan：決定済み内容から実装計画を作る

## 推奨

* モード：Plan
* モデル：軽量モデル
* 前提：技術方式が確定済み
* 出力：`implementation-plan.md`

## プロンプト

```
以下の資料に基づいて、実装計画を作成してください。

【参照資料】
- docs/current/feature-brief.md
- docs/current/repository-findings.md
- docs/current/technical-decisions.md

technical-decisions.mdのDecisionは確定事項です。

次のことを行わないでください。

- 確定済みの技術方式を再検討する
- 新しいアーキテクチャを独自に導入する
- 新規依存を独自に追加する
- 要求範囲を拡張する
- この段階でコードを実装する

計画を、独立して検証可能なステップに分割してください。

各ステップに以下を記載してください。

1. ステップ名
2. 目的
3. 変更対象ファイル
4. 追加または変更するクラス・関数・データ
5. 具体的な変更内容
6. 変更してはいけない内容
7. 追加するテスト
8. 実行する検証コマンド
9. 完了条件
10. 停止条件
11. 依存する先行ステップ

計画上の制約:

- 1ステップで扱う主要責務は1つにする
- 各ステップ終了時にテスト可能にする
- 大規模な一括書換えを避ける
- 最初にテストfixtureまたは診断機能を整える
- 既存挙動を維持する回帰テストを含める
- 最大15ステップにする
- 同じ説明を繰り返さない
- 実装コードは出力しない

未確定事項が見つかった場合:

- 独自に判断しない
- 計画内で「BLOCKED」とする
- 最後に最大5件を列挙する

最後に以下を付けてください。

## 実装バッチ案
Agentモードへ一度に渡すステップの組合せ。

## 全体完了条件
機能全体として合格と判断する条件。```

結果を`docs/current/implementation-plan.md`へ保存する。

---

# 10. 社内AIチャット：実装計画レビュー

## 使用タイミング

* Planモードが作った計画を実装する前
* コードそのものではなく、計画の抜けを確認したい
* Copilotの追加クレジットを使わずにレビューしたい

## プロンプト

```
以下の要求、技術判断、実装計画をレビューしてください。

【要求】
<feature-brief.md>

【技術判断】
<technical-decisions.md>

【実装計画】
<implementation-plan.md>

目的は計画全体を書き直すことではなく、
実装中の手戻りにつながる重大な不足を発見することです。

確認観点:

1. すべての要求が計画へ反映されているか
2. 技術判断と矛盾していないか
3. ステップの依存順序が妥当か
4. 各ステップが独立して検証可能か
5. 回帰テストが不足していないか
6. 自動処理の停止条件があるか
7. 後方互換性を確認できるか
8. 一度に変更する範囲が広すぎないか
9. 対象外機能が混入していないか
10. 人間の承認ポイントが不足していないか

出力:

## 重大な問題
最大5件。該当する計画ステップも示す。

## 軽微な改善
最大5件。

## 実装開始判定
- READY
- READY WITH CONDITIONS
- NOT READY

## 実装前に修正すべき最小差分
計画全体を書き直さず、修正箇所だけ示す。```

---

# 11. Copilot Agent：計画済みステップを実装する

## 推奨

* モード：Agent
* モデル：軽量モデル
* 一度に渡す量：2～4ステップ
* 原因不明の問題が出たら上位モデルへ切り替える

## プロンプト

```
承認済みの実装計画に従い、
ステップ<開始番号>～<終了番号>だけを実装してください。

【参照資料】
- docs/current/feature-brief.md
- docs/current/technical-decisions.md
- docs/current/implementation-plan.md

【今回実装するステップ】
- Step <番号>: <名称>
- Step <番号>: <名称>

【変更可能な範囲】
- 
- 
- 

【参照のみ可能な範囲】
- 
- 

【変更禁止】
- 公開API
- 計画外のファイル
- 新規依存の追加
- 無関係なリファクタリング
- フォーマットだけの大規模変更
- 既存テストの削除または弱体化

実装ルール:

1. 計画の順番に実装する
2. 各ステップ後に対象テストを実行する
3. 最後に関連する回帰テストを実行する
4. テスト失敗を無効化や期待値変更で隠さない
5. 計画との矛盾を発見した場合は独自判断せず停止する
6. 型や解析結果を推測で確定しない
7. 同じ原因に対する修正を2回試して解決しない場合は停止する
8. 実装範囲を勝手に次のステップへ広げない

完了報告には以下を含めてください。

- 実装したステップ
- 変更したファイル
- 主な変更内容
- 実行したテスト
- テスト結果
- 計画からの差異
- 残った問題
- 次に実装可能なステップ

実装結果を
docs/current/implementation-log.mdへ追記してください。```

---

# 12. Copilot Agent：軽量モデルの停止・昇格ルール

Agentプロンプトへ必要に応じて追加する。

```
以下のいずれかに該当した場合は、
場当たり的な修正を続けずに停止してください。

- 同じテスト失敗を2回解決できなかった
- 計画外のAPI変更が必要になった
- 計画外の新規依存が必要になった
- 原因不明の回帰が発生した
- 複数の設計方式から選択する必要が生じた
- 関連ファイルの特定からやり直す必要が生じた
- 仕様と計画に矛盾が見つかった
- 入力情報から型や意味を一意に確定できない
- 自動修正の安全性を証明できない

停止時は次だけを報告してください。

1. 停止した場所
2. 発生した問題
3. 確認済みの事実
4. 未確認の推測
5. 必要な技術判断
6. 上位モデルへ渡す最小限の情報```

---

# 13. Copilot Ask：実装差分レビュー

## 推奨

* モード：Ask
* モデル：中位モデル
* ファイル変更：禁止
* 大規模な再生成：禁止

## プロンプト

```
Askモードで、今回の実装差分を読み取り専用でレビューしてください。

【参照資料】
- docs/current/feature-brief.md
- docs/current/technical-decisions.md
- docs/current/implementation-plan.md
- docs/current/implementation-log.md
- 今回のgit diff
- 関連テスト

レビュー対象:
- 実装計画のStep <番号>～<番号>

確認観点:

1. 要求を満たしているか
2. 技術判断に従っているか
3. 計画外の変更がないか
4. 呼び出し元の変更漏れがないか
5. 型やデータ構造の不整合がないか
6. 例外系と境界条件が扱われているか
7. テストが実装詳細だけでなく振る舞いを検証しているか
8. テストを弱体化していないか
9. 既存機能への回帰リスクがないか
10. 自動処理の安全停止条件が守られているか

出力:

## BLOCKER
マージまたは次段階へ進む前に必ず修正する問題。

## MAJOR
重大だが、条件付きで次へ進める問題。

## MINOR
後から修正可能な問題。

## TEST GAP
不足しているテスト。

## 判定
- PASS
- PASS WITH CONDITIONS
- FAIL

制約:
- ファイルを変更しないでください。
- 差分全体を書き直さないでください。
- 好みだけの指摘は避けてください。
- 各指摘に対象ファイルと理由を付けてください。
- 重大な問題を最大7件に限定してください。```

---

# 14. 社内AIチャット：実装結果の要件レビュー

## 使用タイミング

* コードを直接見せる必要がないレビュー
* 実装結果が当初目的を満たすか確認したい
* 次の段階へ進む判断をしたい

## 入力用レビュー資料

```
【当初の目的】
-

【受入条件】
-

【確定した技術判断】
-

【実装した内容】
-

【変更したファイルの役割】
-

【実行したテスト】
-

【テスト結果】
-

【計画から変更した点】
-

【未解決問題】
-

【今回レビューしてほしい点】
-```

## プロンプト

```
以下の実装結果を、要求と安全性の観点からレビューしてください。

<上記レビュー資料>

コードの細かな書き方ではなく、次を確認してください。

1. 当初の目的を達成しているか
2. 受入条件を客観的に確認できるか
3. 安全要件に抜けがないか
4. テスト観点に偏りがないか
5. 計画からの変更が妥当か
6. 人間による確認が必要な項目が残っていないか
7. 次段階へ進んだ場合のリスク
8. 実コード適用前に必要な追加検証

出力:

## 達成済み
## 未達成
## 追加検証が必要
## 人間の承認が必要
## 次段階へ進む条件
## 判定
- PROCEED
- PROCEED WITH CONDITIONS
- HOLD

入力情報だけでは判断できないことを、
実装済みであるかのように推測しないでください。```

---

# 15. 原因不明の不具合：上位モデルへの調査依頼

## 推奨

* 最初：Ask＋高性能モデル
* 原因が特定できた後：Agent＋軽量または中位モデル
* いきなりAgentへ修正させない

## プロンプト

```
Askモードで、以下の不具合を読み取り専用で調査してください。

【期待する挙動】
-

【実際の挙動】
-

【再現手順】
1.
2.
3.

【エラー出力】
-

【直前の変更】
-

【既に試したこと】
-

【関連ファイル】
-

目的は、修正コードをすぐ生成することではなく、
原因候補を絞り込み、最小の検証手順を作ることです。

出力:

1. 確認できた事実
2. 原因候補
3. 各候補を支持する根拠
4. 各候補を否定または確認する最小テスト
5. 最も可能性が高い原因
6. 修正方針
7. 修正による回帰リスク
8. Agentへ渡す実装指示

制約:
- ファイルを変更しないでください。
- 原因を確認せずに修正案を断定しないでください。
- 原因候補は最大5件にしてください。
- 無関係なリファクタリングを提案しないでください。```

---

# 16. セッション切替用の引き継ぎテンプレート

長いCopilot会話を継続せず、新しい会話へ切り替えるために使う。

```
# Development Handoff

## 1. Current objective

<現在の目的>

## 2. Completed work

- 
- 

## 3. Current implementation state

- 完了:
- 部分完了:
- 未着手:

## 4. Confirmed technical decisions

- 
- 

## 5. Files changed

| File | Purpose |
|---|---|
|  |  |

## 6. Tests executed

| Command | Result |
|---|---|
|  |  |

## 7. Known failures

- 

## 8. Remaining risks

- 

## 9. Next approved step

<次に行う実装計画のステップ>

## 10. Files needed for the next step

- 

## 11. Files that should not be inspected

- 

## 12. Stop conditions

- 

## 13. Important constraints

- 実コードへの適用は未検証
- 合成fixtureで検証
- 既存APIを維持
- 新規依存を原則追加しない
<必要に応じて修正>```

新しいCopilot会話では次のように開始する。

```
以下の引き継ぎ文書を前提に作業を継続してください。

- docs/current/handoff.md
- docs/current/implementation-plan.md
- docs/current/technical-decisions.md

過去の経緯をリポジトリ全体から再調査しないでください。
まずNext approved stepと、その実装に必要なファイルだけを確認してください。

不整合を発見した場合は実装を開始せず報告してください。```

---

# 17. タスク規模別の簡易ワークフロー

## 小さな修正

例：

* エラーメッセージの変更
* フィールド追加
* 既存パターンに沿ったテスト追加

```
Copilot Ask／軽量モデルで関連箇所確認
↓
インライン補完またはAgent／軽量モデル
↓
対象テスト```

Planモードと社内AIは原則不要。

---

## 中規模機能

例：

* 複数ファイルにまたがる機能追加
* CLIオプション追加
* 新しい診断モデル追加

```
社内AI：要求整理
↓
Copilot Ask：リポジトリ調査
↓
Copilot Plan／軽量モデル
↓
Copilot Agent／軽量モデル
↓
Copilot Ask：差分レビュー```

---

## 技術判断を伴う機能

例：

* 解析方式の変更
* 新規ライブラリ導入
* データ構造の変更
* 自動修正境界の決定

```
社内AI：要求整理・安全性レビュー
↓
Copilot Ask／中軽量モデル：現状調査
↓
Copilot Ask／高性能モデル：技術判断
↓
Copilot Plan／軽量モデル：計画展開
↓
Copilot Agent／軽量モデル：分割実装
↓
Copilot Ask：コードレビュー
↓
社内AI：要件レビュー```

---

## 原因不明の不具合

```
Copilot Ask／高性能モデル：原因調査
↓
人間が原因と修正方針を確認
↓
Copilot Agent／軽量または中位モデル：修正
↓
対象テストと回帰テスト```

---

# 18. クレジットを抑える共通追記

必要に応じて各プロンプトの末尾へ追加する。

```
クレジット消費を抑えるため、以下を守ってください。

- 必要なファイルだけを参照する
- リポジトリ全体を無制限に探索しない
- 一般論を長く説明しない
- 同じ内容を複数の形式で繰り返さない
- 確定事項を再検討しない
- 未確定事項を勝手に決定しない
- ファイル全体ではなく関連箇所を優先して読む
- 出力は指定した形式と上限に従う
- 完成した文書を毎回全文再生成せず、変更箇所だけ示す
- 重大な問題がなければ追加提案を増やさない```

---

# 19. 最小構成

運用を簡単にしたい場合は、次の4ファイルだけでもよい。

```
feature-brief.md
technical-decisions.md
implementation-plan.md
handoff.md```

最小フローは次のとおり。

```
1. 社内AIでfeature-brief.mdを作る
2. Copilot Askで現状を調査する
3. 重要判断があればAsk＋高性能モデルで決める
4. Plan＋軽量モデルでimplementation-plan.mdを作る
5. Agent＋軽量モデルで2～4ステップずつ実装する
6. セッションを切り替える前にhandoff.mdを更新する```

高性能モデルには、長い計画やコードを出力させない。

```
高性能モデルの成果物:
短い判断、重大リスク、停止条件

軽量モデルの成果物:
計画文書、タスクリスト、コード、テスト、実装報告```

このテンプレートを、現在開発中のFortran統合ツール向けに具体化した版へ置き換えることもできます。

* * *

**You:**

askモードがファイル生成ができないことを考えると、feature-brief.mdの編集や、technical-decisions.mdの作成は手動でやらないといけないのでしょうか？また、planモードも記憶量機のplan.md以外はファイル生成ができないことを考えると、implementation-plan.mdはどうやって生成させれば良いでしょうか？

* * *

**ChatGPT:**

はい。ここは**「内容を考えるモード」と「ワークスペースへ保存する手段」を分けて考える**必要があります。

結論は次のとおりです。

* **Askモード**：内容はチャットに生成できるが、ファイルは直接変更しない
* **Planモード**：計画をセッション内の`/memories/session/plan.md`に自動保存する
* **Agentモード**：新規Markdownの作成や既存Markdownの編集ができる
* **最もクレジットを節約する方法**：AskやPlanの出力をエディターで開き、人間が保存する
* **手作業を減らす方法**：保存作業だけLunaのAgentへ渡す

Askモードの回答にはコードブロックや挿入操作が含まれることがありますが、Ask自体はコードベースの質問・理解・アイデア探索向けです。ファイル編集を自律的に行うのはAgentモードです。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs+1](https://docs.github.com/copilot/using-github-copilot/asking-github-copilot-questions-in-your-ide)

## おすすめの運用

### 1. `feature-brief.md`

社内AIチャットで内容を作る場合、社内AIはVS Codeのワークスペースへ直接書き込めないため、**社内AIからVS Codeへの一度の受け渡しは必要**です。

最もクレジット効率がよいのは、次の方法です。

1. 社内AIに完成版Markdownだけを出力させる
2. VS Codeで`docs/current/feature-brief.md`を作る
3. 内容をコピー＆ペーストする

これは手作業といっても、文章を自分で編集する必要はなく、ファイル作成と貼り付けだけです。

社内AIには次のように依頼します。

```
以下の回答は、そのままfeature-brief.mdとして保存します。

説明や前置きは付けず、完成したMarkdown本文だけを
1つのコードブロック内に出力してください。

構成:
# Feature Brief
## Background
## Objective
## Scope
## Out of Scope
## Functional Requirements
## Safety Requirements
## Acceptance Criteria
## Open Questions
```

### コピー＆ペーストも避けたい場合

社内AIの出力をCopilot Agentへ貼り、Lunaに保存だけさせます。

```
以下のMarkdownを
docs/current/feature-brief.md
として新規作成してください。

内容の要約、修正、再構成は行わず、そのまま保存してください。
他のファイルは参照・変更しないでください。
コマンドは実行しないでください。

--- BEGIN CONTENT ---

<社内AIチャットの出力>

--- END CONTENT ---
```

これはLunaの短い1リクエストで済みます。ただし、**クレジット節約だけを優先するなら自分で貼り付けた方が安価**です。

---

## 2. `feature-brief.md`のレビュー後の修正

Askモードや社内AIにレビューさせた後、既存の`feature-brief.md`を修正する方法は3つあります。

### 最安：人間が差分を反映

社内AIに「変更後の全文」ではなく、修正文だけを出させ、手動で反映します。

### 制御しやすい：Editモード

利用中のVS CodeにEditモードが表示されていれば、対象ファイルを`feature-brief.md`だけに限定して編集させます。Editモードは、ユーザーが作業対象ファイルを指定し、各編集を承認する用途に向いています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/copilot/using-github-copilot/asking-github-copilot-questions-in-your-ide)

```
docs/current/feature-brief.mdだけを編集してください。

以下のレビュー指摘を反映してください。
指摘されていない節は変更しないでください。
文章全体の言い換えや再構成はしないでください。

<レビュー結果>
```

### Editモードがない場合：Agent＋Luna

```
docs/current/feature-brief.mdだけを編集してください。

以下のレビュー指摘を最小差分で反映してください。

禁止:
- 他のファイルの変更
- 要求の追加
- 章構成の変更
- 文体だけを理由とする全面書換え
- コマンド実行

<レビュー結果>
```

---

## 3. `technical-decisions.md`

Ask＋高性能モデルで技術判断を行った場合も、Askは判断結果を**チャットには生成できます**。ただし、その結果をワークスペースに永続化するには別の操作が必要です。

### 最もおすすめする方法

Askへのプロンプトで、最初から保存可能なMarkdownとして出力させます。

```
調査結果はtechnical-decisions.mdへ保存します。

回答は説明や前置きを付けず、完成したMarkdown本文だけを
1つのコードブロック内に出力してください。

構成:
# Technical Decisions

## Decision 1: <name>
### Decision
### Rationale
### Alternatives Rejected
### Fallback
### Safety Stop Conditions
### Required Validation

## Unresolved Issues
## Confirmed Inputs for Planning
```

その後は、次のどちらかです。

### クレジット最小

1. `docs/current/technical-decisions.md`を作成
2. Ask回答のコードブロックにある挿入操作を使う、またはコピー＆ペースト
3. 保存

Askの応答には、コードブロックの挿入などを行う対話要素が付く場合があります。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/ja/copilot/how-tos/chat-with-copilot/chat-in-ide)

### 操作最小

同じチャットセッションのまま、

1. モデルをGPT-5.4からLunaへ変更
2. AskからAgentへ切り替える
3. 次の指示を送る

```
直前に確定したTechnical Decisionsを、
内容を変更せず次のファイルへ保存してください。

docs/current/technical-decisions.md

他のファイルは変更しないでください。
追加調査、技術判断の再検討、コマンド実行は不要です。
```

VS Codeでは、同じセッションの途中でAsk、Plan、Agentなどを切り替えられます。Agentモードはファイルの新規作成・編集を行えるため、**高性能モデルで判断し、Lunaで保存だけ行う**運用が可能です。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/agents/overview)

## どちらがよいか

| 方法 | 操作 | AIクレジット |
| --- | --- | --- |
| コードブロックを手動保存 | 少し必要 | 追加なし |
| Luna Agentで保存 | ほぼ不要 | Luna 1リクエスト |
| GPT-5.4 Agentでそのまま保存 | 不要 | 高価なので非推奨 |

---

## 4. `implementation-plan.md`

ここは、Planモードで作られた内容を**別のモデルに再生成させる必要はありません**。

現在のVS CodeのPlanエージェントは、確定した計画を自動的に次へ保存します。

```
/memories/session/plan.md
```

ただし、これはセッションメモリであり、会話終了時に消去されます。公式ドキュメントでは、`Chat: Show Memory Files`コマンドから`plan.md`を開けると説明されています。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/agents/planning)

### 推奨方法A：Planの「エディターで開く」を使う

Planの作成が完了したら、表示される**「エディターで開く」**を選択します。

公式説明では、この操作により計画の詳細を含むMarkdownがエディタータブに生成され、後で使うMarkdownファイルとして保存できます。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/ja/copilot/how-tos/chat-with-copilot/chat-in-ide)

そのタブを次の名前で保存します。

```
docs/current/implementation-plan.md
```

この方法なら、Planをもう一度生成する必要はありません。

### 推奨方法B：メモリファイルを開いて保存

1. コマンドパレットを開く
2. `Chat: Show Memory Files`
3. `plan.md`を選択
4. `Save As`
5. `docs/current/implementation-plan.md`として保存

これも追加のAIリクエストは不要です。

### 操作もAgentへ任せる場合

Plan完成後、同じセッションでAgent＋Lunaへ切り替えます。

```
現在確定しているPlanエージェントの実装計画を、
次の永続ファイルへ保存してください。

docs/current/implementation-plan.md

要件:
- 計画内容を再生成しない
- 技術判断を変更しない
- 内容を追加・省略しない
- 他のファイルを変更しない
- 実装を開始しない
- コマンドを実行しない
```

ただし、メモリファイルの直接保存よりも1リクエスト余分にかかります。

---

# 修正後の現実的なワークフロー

## クレジットを最優先する場合

```
社内AI
  feature-brief.md本文を生成
       ↓ コピー＆ペースト
feature-brief.md

Copilot Ask＋高性能モデル
  技術判断をMarkdownコードブロックで生成
       ↓ Insertまたはコピー＆ペースト
technical-decisions.md

Copilot Plan＋Luna
  計画を生成
       ↓ 「エディターで開く」→ Save As
implementation-plan.md

Copilot Agent＋Luna
  実装し、implementation-log.mdを直接更新
```

この構成では、**ファイル保存だけのためにCopilotクレジットを使いません**。

## 手操作を最小にする場合

```
社内AI
  feature brief生成
       ↓ 出力をAgentへ貼る
Copilot Agent＋Luna
  feature-brief.mdを作成

Copilot Ask＋GPT-5.4
  技術判断
       ↓ 同じセッションでモード・モデル切替
Copilot Agent＋Luna
  technical-decisions.mdを作成

Copilot Plan＋Luna
  実装計画
       ↓ 同じセッションでAgentへ切替
Copilot Agent＋Luna
  implementation-plan.mdへ保存

Copilot Agent＋Luna
  実装
```

こちらは操作が少ない代わりに、文書保存ごとにLunaのリクエストが増えます。

## 私の推奨する折衷案

| 文書 | 作成方法 |
| --- | --- |
| `feature-brief.md` | 社内AI出力を手動で1回貼り付け |
| `repository-findings.md` | 必要な場合だけAsk出力を貼り付け |
| `technical-decisions.md` | Ask出力をコードブロックから挿入 |
| `implementation-plan.md` | Planの「エディターで開く」から保存 |
| `implementation-log.md` | 実装Agentに直接更新させる |
| `handoff.md` | 実装Agentにセッション終了時に生成させる |

つまり、**Askの成果物は「保存可能な完成Markdown」として出力させ、Planの成果物は内蔵のplan.mdをそのまま永続化する**のがよいです。

前回のテンプレート中の、

> 結果を`technical-decisions.md`へ保存する  
> 結果を`implementation-plan.md`へ保存する

という表現は、モード自身が直接保存するように読めるため不正確でした。正しくは、

> Askで保存可能なMarkdownを生成し、エディターへの挿入またはLunaのAgentで永続化する  
> Planのセッションメモリ`plan.md`を「エディターで開く」またはSave Asで永続化する

となります。

* * *

**You:**

結局askモードで調査する場合でも.mdファイルの変更案をchatとして生成させるなら、agentモードで編集・生成対象ファイルを対象の.mdファイルに指定した上で調査・編集させるのと、クレジット消費量は変わらないのではないの？それともchatの生成はクレジットとして加算されないなどあるの？

* * *

**ChatGPT:**

はい。**その理解がほぼ正しいです。**  
Askモードで完成したMarkdown全文をチャットに生成させるなら、Agentモードで同じMarkdownをファイルへ直接書かせる場合と、クレジット消費は大きく変わらない可能性があります。

**チャットに表示される文章も出力トークンとして加算されます。無料ではありません。** 現行のAIクレジット方式では、入力トークン、出力トークン、キャッシュ関連トークンをモデルごとの単価で計算します。Ask・Plan・Agentというモード名そのものに固定の割引があるわけではありません。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/copilot/reference/copilot-billing/models-and-pricing)

## 同じ内容なら、おおむねこうなります

### Askで作る場合

```
リポジトリを検索
→ 関連ファイルを読む
→ technical-decisions.md全文をチャットに出力
→ 人間がファイルへ貼り付け
```

消費対象は主に、

* プロンプト
* 検索結果と読み込んだファイル
* 会話履歴
* チャットに生成したMarkdown全文

です。

### Agentで直接作る場合

```
リポジトリを検索
→ 関連ファイルを読む
→ technical-decisions.mdを作成・編集
→ 短い完了報告
```

消費対象は主に、

* プロンプト
* 検索結果と読み込んだファイル
* 会話履歴
* ファイル編集のためにモデルが生成した内容
* 完了報告
* 必要に応じた再読込や追加ツール呼び出し

です。

Agentがファイルへ書いた文章も、モデルが生成している以上、基本的には出力トークンを使います。したがって、

> AskでMarkdown全文を出力するだけなら安い  
> Agentでファイルを作ると特別に高い

という単純な違いではありません。

## Agentの方が高くなりやすい理由

Agentは、自律的に検索・編集・確認を繰り返せます。VS Code公式でも、Agentは必要なツールを自動選択し、検索、編集、コマンド実行などを繰り返す仕組みです。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/chat/chat-tools)

例えば次のような追加処理が発生することがあります。

1. 対象ファイルを検索
2. 関連ファイルを読む
3. Markdownを作成
4. 作成したMarkdownを再度読む
5. 内容を修正
6. `git diff`を確認
7. 完了報告を生成

一方、Askを厳しく制限すれば、

1. 対象ファイルを読む
2. Markdown全文を一度だけ回答する

で終了しやすいため、やや安くなる可能性があります。

ただし、**Agentも対象ファイルとツールを厳しく制限すれば、この差は小さくできます。**

## 既存ファイルの修正ではAgentが安い場合もある

既存の`feature-brief.md`へ数行だけ追記する場合は、Agentに最小差分で編集させた方が安いこともあります。

Askで次のように依頼すると、

```
修正後のfeature-brief.md全文を出力してください。
```

変更していない部分まで全て出力するため、出力トークンが増えます。

Agentなら、

```
feature-brief.mdのAcceptance Criteria節だけを最小差分で修正してください。
```

と指定でき、モデルがパッチ相当の差分だけを生成できる可能性があります。この場合は、Agentの編集ツールの方が効率的です。

したがって、比較は次のようになります。

| 作業 | 有利になりやすい方法 |
| --- | --- |
| 新しいMarkdown全文を作る | AskとAgentで大差なし |
| 既存Markdownの小さな修正 | Agentの最小差分編集 |
| 技術判断だけ確認する | Ask |
| 調査結果をそのまま保存したい | Agent |
| 調査結果を人間が選別してから保存したい | Ask |
| 実装やコマンド実行を絶対に防ぎたい | Askまたは読み取り専用Agent |

## 今回の用途なら、Agentへ統一してよい

クレジット節約を目的として、わざわざ

```
Askで調査・全文生成
→ 手動でコピー
```

にする強い理由はありません。

特に`technical-decisions.md`を作る目的が最初から明確なら、**Agent＋高性能モデルで、編集権限と出力範囲を厳しく制限して直接生成する**方が、操作が少なく合理的です。

```
Agentモードで読み取り調査を行い、
docs/current/technical-decisions.mdだけを作成してください。

参照可能:
- docs/current/feature-brief.md
- pyproject.toml
- src/parser/
- src/analysis/
- tests/fixtures/

変更可能:
- docs/current/technical-decisions.mdのみ

禁止:
- ソースコードやテストの変更
- コマンド実行
- 新規依存の追加
- 実装計画やタスクリストの生成
- 確定していない事項の推測
- 作成後の全面的な再読込・再生成

出力内容:
- Decision
- Rationale
- Alternatives rejected
- Fallback
- Safety stop conditions
- Required validation
- Planningへの確定入力

作成後は、変更したファイル名と未解決事項だけを
簡潔に報告してください。
```

これなら、GPT-5.4の出力はほぼ`technical-decisions.md`だけに限定できます。

## Askを残すべき場面

Askの利点は主に**安全性と対話性**であって、必ずしもクレジット単価ではありません。

Askが向いているのは、

* 調査結果を見てから採用案を人間が選びたい
* 複数の候補を比較して議論したい
* まだファイルへ確定事項を書き込みたくない
* 読み取り専用を確実にしたい
* 一度の技術判断では確定できない

場合です。

逆に、次の条件ならAgentで直接よいです。

* 成果物の保存先が決まっている
* 出力形式が決まっている
* 調査対象が限定されている
* 対象Markdown以外を変更禁止にできる
* 判断結果をそのまま正式文書として採用する

## 修正した推奨フロー

これまでの説明を踏まえると、Spec Kitを使わない場合は次の方が実用的です。

```
社内AI
要求の素材やレビュー観点を作る
        ↓
Agent＋高性能モデル
リポジトリを調査し、
technical-decisions.mdだけを直接作成
        ↓
Plan＋軽量モデル
セッションのplan.mdを生成
        ↓
Agent＋軽量モデル
計画に従って実装
```

`feature-brief.md`についても、社内AIの出力を手動で貼るのが面倒なら、LunaのAgentへ渡して直接作成させて問題ありません。

要するに、**Askはクレジット節約用モードではありません。**  
チャット出力も課金されます。クレジット面で重要なのはモードより、

* 使用モデル
* 読み込ませるコンテキスト
* 生成する文章量
* ツール呼び出し回数
* 再試行回数
* 全文生成か最小差分か

です。GitHubも、モデル選択、コンテキストの削減、明確なガードレール、調査・計画・実装の分離を主要な節約策として挙げています。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/copilot/tutorials/optimize-ai-usage)

* * *

**You:**

では改めてプロンプトのテンプレートを作成して

* * *

**ChatGPT:**

以下は、これまでの議論を反映した**修正版テンプレート集**です。  
Askモードを標準工程から外し、Markdown成果物はAgentモードで直接作成・編集する構成にしています。

# GitHub Copilotクレジット節約用プロンプトテンプレート

## 1. 基本ワークフロー

Spec Kitを使用しない場合は、次の分担を基本とする。

```
社内AIチャット
要求・安全条件・テスト観点の整理
        ↓
Copilot Agent＋軽量モデル
feature-brief.mdの作成
        ↓
Copilot Agent＋高性能モデル
リポジトリ調査と重要な技術判断
technical-decisions.mdの作成
        ↓
Copilot Plan＋軽量モデル
決定済み事項から実装計画を生成
        ↓
Copilot Agent＋軽量モデル
計画を永続化し、分割実装
        ↓
必要な場合のみ上位モデル
原因不明の問題・計画変更の判断```

### モデル選択の原則

| 作業 | モデル |
| --- | --- |
| 文章整理・書式変換 | 軽量モデル |
| 明確な要求から計画を展開 | 軽量モデル |
| 計画済みの実装 | 軽量モデル |
| リポジトリを踏まえた方式決定 | 高性能モデル |
| 新規パッケージの比較・採否 | 高性能モデル |
| 原因不明の不具合調査 | 中～高性能モデル |
| 定型的なテスト追加・文書更新 | 軽量モデル |

重要なのは、ファイル数ではなく、モデルが作業中に行う必要がある判断の量である。

---

# 2. 管理ファイル

```
docs/current/
├── feature-brief.md
├── technical-decisions.md
├── implementation-plan.md
├── implementation-log.md
├── review.md
└── handoff.md```

| ファイル | 用途 |
| --- | --- |
| `feature-brief.md` | 目的、範囲、要件、受入条件 |
| `technical-decisions.md` | 技術方式、代替案、停止条件 |
| `implementation-plan.md` | 承認済み実装計画の永続版 |
| `implementation-log.md` | 変更内容、テスト結果、計画との差異 |
| `review.md` | 実装レビュー結果 |
| `handoff.md` | 次のセッションへの引き継ぎ |

---

# 3. 社内AIチャット：要求整理

## 使用目的

* Copilotへ渡す前に要望を整理する
* 安全要件や受入条件を洗い出す
* リポジトリを見なくても考えられる部分を社内AIに任せる
* Copilotの高価な調査時間を減らす

## プロンプト

```
以下の開発要望を、ソフトウェア機能の要求定義として整理してください。

【開発したい内容】
[自由記述]

【背景】
[なぜ必要か]

【現在の問題】
[現在発生している問題、失敗例、面倒な作業]

【既知の制約】
- [制約1]
- [制約2]
- [制約3]

以下の構成で、完成したMarkdownを出力してください。

# Feature Brief

## 1. Background
## 2. Objective
## 3. Intended Use
## 4. Scope
## 5. Out of Scope
## 6. Functional Requirements
## 7. Safety and Reliability Requirements
## 8. Normal Cases
## 9. Error and Boundary Cases
## 10. Acceptance Criteria
## 11. Human Approval Points
## 12. Open Questions

注意事項:
- 技術方式や使用パッケージはまだ決定しないでください。
- 実装コードは生成しないでください。
- 不明な点を勝手に補完せず、Open Questionsに記載してください。
- 受入条件は、できるだけテスト可能な表現にしてください。
- 要求と実装方法を混同しないでください。
- 前置きや説明は付けず、Markdown本文だけを出力してください。```

---

# 4. Copilot Agent＋軽量モデル：feature-brief.mdの作成

社内AIの出力を貼り付け、軽量モデルに保存と必要最小限の整形だけをさせる。

```
Agentモードで、次のファイルだけを作成または編集してください。

変更可能:
- docs/current/feature-brief.md

変更禁止:
- ソースコード
- テストコード
- 設定ファイル
- その他のMarkdownファイル

以下の社内AIチャットの出力を、
docs/current/feature-brief.mdとして保存してください。

要件:
- 内容を勝手に追加しない
- 意味を変更しない
- 重複した記述だけを必要に応じて統合する
- 見出しと箇条書きのMarkdown形式を整える
- 未確定事項はOpen Questionsに残す
- コマンドは実行しない
- リポジトリ全体を調査しない

--- BEGIN INPUT ---

[社内AIチャットの出力]

--- END INPUT ---

完了後は、変更したファイル名だけを簡潔に報告してください。```

要求が既に十分明確なら、社内AIを省略して、同じAgentプロンプトへ要望を直接入力してもよい。

---

# 5. 社内AIチャット：要求の安全性レビュー

```
以下のFeature Briefをレビューしてください。

[feature-brief.mdの内容]

目的は文章をきれいにすることではなく、
実装後の手戻りや危険な自動処理につながる曖昧さを発見することです。

確認観点:
1. 成功条件が客観的に判定できるか
2. 失敗時の挙動が定義されているか
3. 自動処理を禁止すべき条件があるか
4. 人間の承認が必要な条件が明確か
5. 不完全な入力の扱いが定義されているか
6. 後方互換性が考慮されているか
7. ロールバックまたは元データ保持が必要か
8. テストできない要求が含まれていないか
9. 対象範囲が広がりすぎていないか
10. 要求間に矛盾がないか

出力形式:

## Critical Issues
最大5件。

## Minor Issues
最大5件。

## Proposed Changes
feature-brief.mdに追加・修正すべき文章だけを示す。

## Readiness
- READY
- READY WITH CONDITIONS
- NOT READY

Feature Brief全体を書き直さないでください。
重大な問題がなければ、無理に指摘を作らないでください。```

レビュー結果は、軽量モデルのAgentで最小差分として反映する。

```
Agentモードで、
docs/current/feature-brief.mdだけを最小差分で編集してください。

以下のレビュー指摘を反映してください。

[社内AIのレビュー結果]

禁止:
- 指摘されていない節の全面的な書き換え
- 新しい要求の独自追加
- ソースコードや他ファイルの変更
- リポジトリ調査
- コマンド実行

未確定事項は推測せず、Open Questionsへ追加してください。```

---

# 6. Copilot Agent＋高性能モデル：技術方式の決定

## 使用目的

* リポジトリを調査する
* 既存実装を踏まえて方式を決定する
* 必要な場合は外部パッケージを比較する
* 結果を直接`technical-decisions.md`へ保存する

## プロンプト

```
Agentモードで読み取り調査を行い、
docs/current/technical-decisions.mdだけを作成または編集してください。

【参照必須】
- docs/current/feature-brief.md
- pyproject.toml
- [関連ディレクトリ1]
- [関連ディレクトリ2]
- [関連テスト・fixture]

【必要な場合のみ参照可能】
- [追加の参照範囲]

【調査対象外】
- .venv/
- build/
- dist/
- generated/
- 大規模なログや解析結果
- 今回の機能と無関係なディレクトリ

【変更可能】
- docs/current/technical-decisions.mdのみ

【変更禁止】
- ソースコード
- テストコード
- feature-brief.md
- 設定ファイル
- 依存パッケージ
- その他のファイル

【今回決定する論点】
1. [技術論点1]
2. [技術論点2]
3. [技術論点3]
4. [技術論点4]

【評価基準】
- 正確性
- 既存コードとの整合性
- 後方互換性
- 不完全な入力への耐性
- 検証可能性
- 失敗時の安全性
- 保守性
- 新規依存の導入負荷

【固定制約】
- [例: Python 3.12]
- [例: uvを使用]
- [例: pytestとgfortranで検証]
- [例: 実コードは使用できず合成fixtureで検証]
- [例: 既存の公開APIを維持]
- [例: 新規依存は原則追加しない]

Web検索が利用可能で、外部パッケージの判断に必要な場合は、
公式ドキュメント、公式リポジトリ、公式リリース情報などの
一次情報を優先してください。

technical-decisions.mdの構成:

# Technical Decisions

## Context

## Decision 1: [論点名]
### Decision
### Rationale
### Alternatives Rejected
### Fallback
### Safety Stop Conditions
### Required Validation

[各論点について繰り返す]

## Cross-Cutting Constraints

## Unresolved Issues

## Confirmed Inputs for Planning

制約:
- 実装計画は作成しない
- タスクリストは作成しない
- 実装コードは生成しない
- 一般的なベストプラクティスを長く説明しない
- 確認できた事実と推測を区別する
- 未確定事項を勝手に決定しない
- 文書は必要最小限とする
- 作成後に内容を何度も全面再生成しない

完了後は次だけを報告してください。
- 作成・変更したファイル
- 未解決事項の件数
- Planへ進めるかどうか```

---

# 7. 外部パッケージを比較するときの追記

前のプロンプトへ次を追加する。

```
【比較対象】
- [候補パッケージ1]
- [候補パッケージ2]
- [候補パッケージ3]
- 既存実装の拡張

各候補について以下を確認してください。
- 必要機能への対応
- 対象言語・ランタイムへの対応
- ライセンス
- メンテナンス状況
- 公式リリースの新しさ
- オフライン導入のしやすさ
- 既存依存との競合
- ソース位置情報など必要な付加情報
- 不完全な入力への耐性
- 技術スパイクで確認すべき内容

候補を必要以上に増やさないでください。
Web上の評判だけで採用を決めず、現在のリポジトリとの適合性を重視してください。
未検証のパッケージは「採用確定」ではなく「条件付き候補」としてください。```

---

# 8. Copilot Plan＋軽量モデル：実装計画

technical-decisions.mdの判断を再検討させず、実装計画への展開だけを行わせる。

```
Planモードで、以下の資料に基づく実装計画を作成してください。

【参照資料】
- docs/current/feature-brief.md
- docs/current/technical-decisions.md

technical-decisions.mdのDecisionは確定事項です。

次を行わないでください。
- 技術方式の再比較
- 新しいアーキテクチャの独自導入
- 新規依存の独自追加
- 要求範囲の拡張
- 実装コードの生成

必要な場合だけ、実装対象を特定するために
以下の範囲を読み取ってください。

【参照可能範囲】
- [関連ソースディレクトリ]
- [関連テスト]
- pyproject.toml

【参照対象外】
- .venv/
- build/
- dist/
- generated/
- 今回の機能と無関係な範囲

実装計画を、独立して検証可能なステップに分割してください。

各ステップに以下を含めてください。
1. ステップ名
2. 目的
3. 変更対象ファイル
4. 追加・変更するクラス、関数、データ構造
5. 具体的な変更内容
6. 変更してはいけない内容
7. 追加するテスト
8. 実行する検証コマンド
9. 完了条件
10. 停止条件
11. 依存する先行ステップ

計画上の制約:
- 1ステップで扱う主要責務は1つにする
- 各ステップの終了時に検証可能にする
- 大規模な一括書き換えを避ける
- 回帰テストを含める
- 一度に実装するバッチは2～4ステップを想定する
- 最大15ステップとする
- 同じ説明を繰り返さない
- 未確定事項を独自判断しない

未確定事項が見つかった場合:
- 該当ステップをBLOCKEDとする
- 最後に最大5件を列挙する

最後に以下を記載してください。

## Recommended Implementation Batches
Agentへ一度に渡すステップの組合せ。

## Overall Completion Criteria
機能全体として合格と判断する条件。```

---

# 9. Copilot Agent＋軽量モデル：計画の保存と実装

Planモードで計画を確認・承認した後、同じセッションでAgentモードへ切り替える。

計画保存だけで1回のリクエストを消費しないよう、**計画の永続化と最初の実装を同時に依頼する**。

```
Agentモードで、直前に承認したPlanを
docs/current/implementation-plan.mdへ内容を変更せず保存してください。

その後、承認済みPlanのStep [開始番号]～[終了番号]だけを実装してください。

【参照資料】
- docs/current/feature-brief.md
- docs/current/technical-decisions.md
- docs/current/implementation-plan.md

【今回実装するステップ】
- Step [番号]: [名称]
- Step [番号]: [名称]
- Step [番号]: [名称]

【変更可能範囲】
- [変更可能ディレクトリ・ファイル]
- docs/current/implementation-plan.md
- docs/current/implementation-log.md

【参照のみ可能】
- [参照のみの範囲]

【変更禁止】
- 計画外のソースファイル
- 公開API
- 新規依存
- 無関係なリファクタリング
- フォーマットだけの大規模変更
- 既存テストの削除
- 既存テストの弱体化
- 次の実装バッチへの着手

実装ルール:
1. Planの順番に実装する
2. 各ステップ後に対象テストを実行する
3. 最後に関連する回帰テストを実行する
4. テスト失敗を無効化や期待値変更で隠さない
5. 計画との矛盾があれば独自判断せず停止する
6. 型や解析結果を推測で確定しない
7. 同じ原因に対する修正を2回試して解決しなければ停止する
8. 計画外の機能を追加しない

docs/current/implementation-log.mdへ以下を追記してください。
- 実装したステップ
- 変更したファイル
- 主な変更内容
- 実行したテスト
- テスト結果
- Planからの差異
- 残った問題
- 次に実装可能なステップ

チャット上の完了報告は簡潔にし、
詳細はimplementation-log.mdへ記録してください。```

2回目以降は、計画保存部分を削除する。

---

# 10. 軽量モデルの停止条件

Agentプロンプトへ毎回追加する。

```
以下のいずれかに該当した場合は、
場当たり的な修正を続けず停止してください。

- 同じテスト失敗を2回解決できなかった
- 計画外のAPI変更が必要になった
- 計画外の新規依存が必要になった
- 原因不明の回帰が発生した
- 複数の技術方式から選択する必要が生じた
- 関連ファイルの特定からやり直す必要が生じた
- feature-brief.mdとPlanに矛盾が見つかった
- technical-decisions.mdと実装が両立しない
- 入力情報から型や意味を一意に確定できない
- 自動修正の安全性を保証できない

停止時は、ファイルを追加修正せず次を報告してください。

1. 停止したステップ
2. 発生した問題
3. 確認済みの事実
4. 未確認の推測
5. 必要な技術判断
6. 上位モデルへ渡すべき関連ファイル
7. 再現コマンドとエラー出力```

---

# 11. 原因不明の問題：上位モデルへの昇格

## Agent＋高性能モデル

```
Agentモードで、以下の問題を調査してください。

この段階では、原因を確認する前にソースコードを変更しないでください。

【参照資料】
- docs/current/feature-brief.md
- docs/current/technical-decisions.md
- docs/current/implementation-plan.md
- docs/current/implementation-log.md
- [関連ソース]
- [関連テスト]

【期待する挙動】
[記載]

【実際の挙動】
[記載]

【再現手順】
1. [手順]
2. [手順]

【エラー出力】
[記載]

【既に試したこと】
[記載]

第1段階として、以下を調査してください。
1. 確認できた事実
2. 原因候補
3. 各候補を確認する最小テスト
4. 最も可能性が高い原因
5. technical-decisions.mdまたはPlanの変更が必要か

原因が十分に特定できた場合だけ、
最小差分の修正を実施してください。

変更可能:
- [問題に直接関係するファイル]
- docs/current/implementation-log.md

変更禁止:
- 無関係なファイル
- 新規依存
- 公開API
- 大規模なリファクタリング
- technical-decisions.mdの独自変更
- implementation-plan.mdの独自変更

設計判断の変更が必要な場合は、コードを変更せず停止してください。

完了報告:
- 根本原因
- 根拠
- 修正内容
- 実行したテスト
- 回帰リスク
- 計画変更の要否```

---

# 12. 実装レビュー

レビュー結果を永続化したい場合は、Agentで`review.md`だけを編集する。

```
Agentモードで、今回の実装差分を読み取り専用でレビューし、
結果をdocs/current/review.mdへ保存してください。

【参照資料】
- docs/current/feature-brief.md
- docs/current/technical-decisions.md
- docs/current/implementation-plan.md
- docs/current/implementation-log.md
- 今回のgit diff
- 関連テスト

【変更可能】
- docs/current/review.mdのみ

【変更禁止】
- ソースコード
- テストコード
- その他のMarkdown
- 設定ファイル

確認観点:
1. 要求を満たしているか
2. 技術判断に従っているか
3. 計画外の変更がないか
4. 呼び出し元の変更漏れがないか
5. 型やデータ構造に不整合がないか
6. 境界条件と異常系が扱われているか
7. テストが振る舞いを検証しているか
8. 既存テストを弱体化していないか
9. 回帰リスクがないか
10. 安全停止条件が守られているか

review.mdの構成:

# Implementation Review

## Scope Reviewed

## Blockers
次へ進む前に必ず修正する問題。

## Major Issues

## Minor Issues

## Test Gaps

## Requirement Coverage

## Final Assessment
- PASS
- PASS WITH CONDITIONS
- FAIL

制約:
- 実装を修正しない
- 差分全体を書き直さない
- 好みだけの指摘を避ける
- 各指摘に対象ファイルと理由を付ける
- 重大な指摘は最大7件とする

チャット上では判定とBlocker件数だけを報告してください。```

保存が不要であれば、この工程だけAskモードを使ってもよい。

---

# 13. 社内AIチャット：実装結果の要件レビュー

コード詳細ではなく、目的・安全性・進め方をレビューさせる。

```
以下の実装結果を、要求と安全性の観点からレビューしてください。

【当初の目的】
[記載]

【受入条件】
[記載]

【確定した技術判断】
[記載]

【実装した内容】
[記載]

【実行したテスト】
[記載]

【テスト結果】
[記載]

【Planから変更した点】
[記載]

【未解決問題】
[記載]

以下を確認してください。
1. 当初目的を達成しているか
2. 受入条件を客観的に確認できるか
3. 安全要件に抜けがないか
4. テスト観点に偏りがないか
5. Planからの変更が妥当か
6. 人間による確認が必要な項目が残っていないか
7. 次段階へ進むリスク
8. 実コード適用前に必要な追加検証

出力:

## Achieved
## Not Achieved
## Additional Validation Required
## Human Approval Required
## Conditions for Next Stage
## Decision
- PROCEED
- PROCEED WITH CONDITIONS
- HOLD

入力情報から判断できないことを、
確認済みであるかのように推測しないでください。```

---

# 14. セッション終了時のhandoff.md生成

実装を担当した軽量モデルへ、最後の実装リクエストと同時に依頼する。

```
今回の作業終了時に、
docs/current/handoff.mdを現在の状態に更新してください。

構成:

# Development Handoff

## Current Objective

## Completed Steps

## Current Implementation State

## Confirmed Technical Decisions

## Files Changed

## Tests Executed

## Known Failures

## Remaining Risks

## Next Approved Step

## Files Needed for the Next Step

## Files That Should Not Be Inspected

## Stop Conditions

## Important Constraints

要件:
- implementation-log.mdと実際の変更内容を基にする
- 未実施の作業を完了済みとしない
- 過去の長い経緯を繰り返さない
- 次のセッションに必要な情報だけを残す
- 1000語以内とする```

新しいセッションは次で開始する。

```
以下の文書を前提に作業を継続してください。

- docs/current/handoff.md
- docs/current/feature-brief.md
- docs/current/technical-decisions.md
- docs/current/implementation-plan.md

過去の経緯をリポジトリ全体から再調査しないでください。

まず次だけを確認してください。
- Next Approved Step
- そのステップに必要なファイル
- 現在のテスト状態
- Stop Conditions

不整合がある場合は、実装を開始せず報告してください。```

---

# 15. クレジット節約用の共通追記

各Copilotプロンプトの末尾へ必要に応じて追加する。

```
クレジット消費を抑えるため、以下を守ってください。

- 指定された範囲だけを参照する
- リポジトリ全体を無制限に探索しない
- 一般論を長く説明しない
- 確定済み事項を再検討しない
- 未確定事項を勝手に決定しない
- 同じ内容をチャットとファイルへ重複して詳述しない
- 詳細は指定Markdownへ保存し、チャット報告は短くする
- 文書全体ではなく最小差分で編集する
- 無関係な整形やリファクタリングを行わない
- 同じ失敗を繰り返さない
- 重大な問題がなければ追加提案を増やさない```

---

# 16. 最小構成

運用負担を抑える場合は、次の4ファイルだけでもよい。

```
docs/current/
├── feature-brief.md
├── technical-decisions.md
├── implementation-plan.md
└── handoff.md```

最小フロー:

```
1. 社内AIで要求の素材を整理
2. Agent＋軽量モデルでfeature-brief.mdを作成
3. Agent＋高性能モデルでtechnical-decisions.mdを作成
4. Plan＋軽量モデルで実装計画を作成
5. Agent＋軽量モデルで計画を保存し、2～4ステップずつ実装
6. 問題が起きた場合だけ上位モデルへ昇格
7. 各セッションの最後にhandoff.mdを更新```

Askモードは必須工程ではない。

```
Askモードを使う場面:
- まだ判断を正式なMarkdownへ反映したくない
- 人間と候補を議論してから決めたい
- ファイル変更を確実に防ぎたい
- 一度だけ短い質問をしたい```

成果物の保存先と形式が決まっている場合は、対象Markdownだけを変更可能にしたAgentモードで直接作成・編集する。

この版では、**高性能モデルが直接`technical-decisions.md`を作成し、Planの出力は最初の実装と同時に`implementation-plan.md`へ保存する**ため、保存だけの追加リクエストを避けられます。

* * *

**You:**

copilotへのプロンプトテンプレートって、./github内かどこかに保存してプロンプトを打つ際に呼び出せないの？

* * *

**ChatGPT:**

はい。**そのための機能がCopilotの「Prompt files」です。**  
プロジェクト内の次の場所に保存します。

```
.github/prompts/
```

ファイル名は、次のように`.prompt.md`で終わらせます。

```
.github/prompts/technical-decisions.prompt.md
```

Copilot Chatで、

```
/technical-decisions
```

と入力すると呼び出せます。後ろに今回固有の指示も追加できます。

```
/technical-decisions COMMON・CALL・暗黙型解析の方式を決定する
```

Prompt fileには、使用するモード、モデル、ツール、引数のヒントなどもYAML frontmatterで設定できます。組み込みの`ask`、`agent`、`plan`またはカスタムエージェントを指定できます。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/agent-customization/prompt-files)

## 今回おすすめする構成

```
.github/
├── copilot-instructions.md
├── prompts/
│   ├── create-feature-brief.prompt.md
│   ├── technical-decisions.prompt.md
│   ├── create-implementation-plan.prompt.md
│   ├── implement-batch.prompt.md
│   ├── review-implementation.prompt.md
│   └── update-handoff.prompt.md
└── agents/
    └── technical-decision-writer.agent.md
```

それぞれの役割は次のようになります。

| 呼び出し | 役割 |
| --- | --- |
| `/create-feature-brief` | 要求資料から`feature-brief.md`を作る |
| `/technical-decisions` | 調査して`technical-decisions.md`だけを作る |
| `/create-implementation-plan` | 確定事項からPlanを作る |
| `/implement-batch` | 指定した計画ステップだけ実装する |
| `/review-implementation` | 差分をレビューする |
| `/update-handoff` | 次セッション用の引き継ぎを更新する |

## 例：technical-decisions.prompt.md

```Markdown
---
name: technical-decisions
description: リポジトリを調査し、重要な技術判断を文書化する
agent: agent
argument-hint: "決定する論点や調査範囲を入力"
---

リポジトリを調査し、
[../../docs/current/technical-decisions.md](../../docs/current/technical-decisions.md)
だけを作成または編集してください。

## 参照必須

- [../../docs/current/feature-brief.md](../../docs/current/feature-brief.md)
- [../../pyproject.toml](../../pyproject.toml)
- ユーザーが今回の入力で指定したソースおよびテスト

## 変更可能

- `docs/current/technical-decisions.md`のみ

## 変更禁止

- ソースコード
- テストコード
- 設定ファイル
- その他のMarkdown
- 依存パッケージ

## 今回の追加指示

${input:decisionTopic:決定する技術論点を入力してください}

## 調査内容

各論点について、以下を決定してください。

- 主方式
- 採用理由
- 不採用とする代替方式
- フォールバック
- 安全停止条件
- 実装前または実装後に必要な検証

外部パッケージの判断が必要な場合は、利用可能なWeb検索を使い、
公式ドキュメント、公式リポジトリ、公式リリース情報を優先してください。

## technical-decisions.mdの構成

# Technical Decisions

## Context

## Decision: <論点名>

### Decision

### Rationale

### Alternatives Rejected

### Fallback

### Safety Stop Conditions

### Required Validation

## Cross-Cutting Constraints

## Unresolved Issues

## Confirmed Inputs for Planning

## 制約

- 実装計画を作らない
- タスクリストを作らない
- ソースコードを変更しない
- 一般論を長く説明しない
- 確認済みの事実と推測を区別する
- 未確定事項を独自判断しない
- チャット上の完了報告は簡潔にする
```

`${input:decisionTopic:...}`のような入力変数を使用すると、呼び出すたびに今回の論点を入力できます。Prompt fileから相対パスのMarkdownリンクで、参照資料も指定できます。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/agent-customization/prompt-files)

## 例：create-implementation-plan.prompt.md

```Markdown
---
name: create-implementation-plan
description: 確定済みの要求と技術判断から実装計画を作る
agent: plan
argument-hint: "今回計画する機能や対象範囲を入力"
---

以下の資料に基づいて実装計画を作成してください。

- [../../docs/current/feature-brief.md](../../docs/current/feature-brief.md)
- [../../docs/current/technical-decisions.md](../../docs/current/technical-decisions.md)

今回の計画対象:

${input:planningScope:計画対象を入力してください}

`technical-decisions.md`のDecisionは確定事項です。

以下を行わないでください。

- 技術方式の再検討
- 新規アーキテクチャの独自導入
- 新規依存の独自追加
- 要求範囲の拡張
- 実装コードの生成

実装計画を独立して検証可能なステップに分割してください。

各ステップには以下を含めてください。

1. 目的
2. 変更対象ファイル
3. 変更するクラス・関数・データ構造
4. 具体的な変更内容
5. 変更禁止事項
6. 追加するテスト
7. 検証コマンド
8. 完了条件
9. 停止条件
10. 先行ステップ

制約:

- 1ステップの主要責務は1つ
- 各ステップ終了時に検証可能
- 最大15ステップ
- 一度に実装する単位は2～4ステップ
- 未確定事項は独自判断せずBLOCKEDとする
```

`agent: plan`と書いておけば、呼び出す際に手動でPlanへ切り替えなくても、そのPrompt fileはPlanエージェントで実行されます。`agent: agent`なら編集可能なAgent、`agent: ask`ならAskを指定できます。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/prompt-files)

## モデルも固定できる

frontmatterには`model`も指定できます。

```YAML
---
name: technical-decisions
agent: agent
model: GPT-5.4
---
```

軽量モデル用なら、モデルピッカーに表示されるLunaの正確な名称を指定できます。

```YAML
---
name: implement-batch
agent: agent
model: GPT-5.6 Luna
---
```

ただし、モデル名や利用可否は契約・組織設定・今後の更新で変わり得ます。そのため、最初は`model`を省略して手動選択し、正しく動くことを確認してから固定するのが安全です。`model`を省略した場合は、現在モデルピッカーで選んでいるモデルが使われます。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/agent-customization/prompt-files)

## Prompt fileと他のファイルの使い分け

### `.github/prompts/*.prompt.md`

**必要なときだけ呼び出す具体的な作業手順**です。

```
/technical-decisions
/implement-batch
```

今回作成した長いプロンプトテンプレートは、基本的にここへ置きます。

### `.github/copilot-instructions.md`

すべてのチャットへ自動的に適用する短いルールです。

```Markdown
- Python環境はuvで管理する
- テストにはpytestを使う
- 実コードへの適用が検証済みであると主張しない
- 新規依存は承認なしに追加しない
- 数値挙動を変える変更には回帰テストを追加する
```

ここへ長いワークフロー全体を書くと、毎回コンテキストへ入り、クレジット面でも不利です。プロジェクト全体に常に適用する規約だけに絞るのがよいです。`.github/copilot-instructions.md`はワークスペース内の全チャット要求に自動適用されます。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)

### `.github/agents/*.agent.md`

特定の役割、使用モデル、使用可能ツールをまとめて固定するものです。

例えば、

* 技術判断専用エージェント
* 読み取り専用レビュアー
* 軽量モデル実装エージェント
* Fortran解析専用エージェント

などです。カスタムエージェントはモデルやツールを固定でき、チャットのAgent選択欄に表示されます。Prompt fileからそのカスタムエージェントを指定することも可能です。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/agent-customization/custom-agents)

## 作成方法

手動で`.github/prompts`へ置く以外にも、次の方法があります。

1. Copilot Chatで`/prompts`を入力
2. **New Prompt (Workspace)**を選択
3. ファイル名を入力
4. 内容を保存

または、

```
/create-prompt 技術判断をtechnical-decisions.mdへ保存するプロンプト
```

と入力すると、Copilotが`.prompt.md`を生成します。現在の会話から「このワークフローを再利用可能なPrompt fileとして保存して」と依頼する方法もあります。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code+1](https://code.visualstudio.com/docs/agent-customization/prompt-files)

したがって、先ほど作ったテンプレートを毎回コピーする必要はありません。**`.github/prompts`に工程別に保存し、`/technical-decisions`や`/implement-batch`として呼び出す運用が最も自然です。**

* * *

**You:**

今回のプロンプトテンプレートに対して、すべての.prompt.mdを作成して

* * *

**ChatGPT:**

作成しました。リポジトリのルートへ展開すると、`.github/prompts/`以下に12個のPrompt fileが配置されます。

### 収録したSlash command

```
/create-feature-brief
/apply-feature-review
/technical-decisions
/package-research
/create-implementation-plan
/persist-plan-and-implement-batch
/implement-batch
/diagnose-and-fix
/review-implementation
/fix-review-findings
/update-handoff
/resume-from-handoff
```

主な設計は次のとおりです。

* Markdown成果物を作る工程は`agent: 'agent'`
* 実装計画の生成だけ`agent: 'plan'`
* モデル名は固定せず、実行時にモデルピッカーでLunaなどの軽量モデル、または高性能モデルを選択
* `${input:...}`で、論点・Step範囲・変更可能範囲などを実行時に入力
* 高性能モデル向けの`technical-decisions`でも、変更可能ファイルを`technical-decisions.md`だけに制限
* Planの永続化と最初の実装を一度に行うPromptを用意
* 軽量モデルが同じ失敗を2回繰り返した場合の停止条件を実装Promptへ組み込み
* 社内AIチャット用の要求整理・安全性レビュー・計画レビューも同梱

Prompt filesのワークスペース標準配置先は`.github/prompts`で、VS CodeのCopilot Chatから`/ファイル名`として呼び出せます。frontmatterでは`agent`、`name`、`description`、`model`などを指定でき、`model`を省略すると現在モデルピッカーで選択中のモデルが使われます。[![](https://www.google.com/s2/favicons?domain=https://code.visualstudio.com&sz=128)Visual Studio Code](https://code.visualstudio.com/docs/copilot/customization/prompt-files)

なお、Prompt filesは現在Public Previewのため、今後VS Code側の仕様変更が入る可能性があります。[![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)docs.github.com](https://docs.github.com/en/copilot/tutorials/customization-library/prompt-files/your-first-prompt-file)