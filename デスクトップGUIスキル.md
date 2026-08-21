**You:**

デスクトップGUIソフトウェア開発に有用なスキル群を教えて

* * *

**ChatGPT:**

デスクトップGUIソフトウェア開発なら、スキルは「GUIデザイン用を1個入れる」より、**開発プロセス・UX・GUIフレームワーク・GUIテストを分業させる**のがかなり有効です。2026年8月時点では、特に **Python + PySide6/Qt** ならかなり良い構成が作れます。

| 役割 | 推奨スキル/ツール | 優先度 |
| --- | --- | --- |
| 開発全体の進め方 | **Superpowers** | ★★★★★ |
| UX・操作フロー設計 | **product-design-and-ux** | ★★★★★ |
| Qt向けUI設計 | **qt-ui-design** | ★★★★★ |
| PySide6/Qt Widgets実装 | **pyqt / pyqt-widgets / pyqt-core** | ★★★★★ |
| GUI非同期処理 | **pyqt-threading** | ★★★★★ |
| GUI自動テスト | **pyqt-testing** | ★★★★★ |
| ダイアログ | **pyqt-dialogs** | ★★★★☆ |
| QSS・見た目 | **pyqt-styling** | ★★★★☆ |
| 実GUIをAIに操作させる | **pyside6-mcp** | ★★★★★ |
| Fluent風の見た目 | **pyside6-fluent-ui** | ★★★☆☆ |
| QMLを使う場合 | **qt-qml / qt-qml-review / qt-qml-test** | ★★★★★ |

### 1. Superpowers

これはGUI専用ではありませんが、**GUI開発でも土台としてかなり有効**です。brainstorming、実装計画、TDD、systematic-debugging、コードレビューなどが含まれています。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/obra/superpowers?_bhlid=744629a2ccf9ef54a58ab3bf0a102eaca461bd46&utm_source=chatgpt.com)

ただしGUIでは一つ注意があります。Superpowers の `brainstorming` はかなり強制力が強く、「ちょっとボタンの位置を変える」といった変更でも設計フェーズを要求しやすい構造です。実際、現在の skill には実装前の設計承認が hard gate として定義されています。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/obra/superpowers/blob/main/skills/brainstorming/SKILL.md?plain=1&utm_source=chatgpt.com)

なので私は、

**大きなGUI変更**  
→ Superpowers

**小さな見た目・挙動調整**  
→ GUI専用スキルだけ

という運用を勧めます。

---

### 2. `product-design-and-ux`

これはかなり相性が良いです。

「どのボタンをどこに置くか」以前に、

* ユーザーが何をしたいか
* 操作フロー
* 画面遷移
* 状態
* エラー時の復帰
* 情報アーキテクチャ

を整理するスキルです。

つまり、

**機能仕様 → UX仕様**

を作る役です。

特定のWebフレームワークなどに依存しておらず、MITライセンスで、デスクトップGUIにも使いやすい設計になっています。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/magnus919/agent-skills/blob/main/product-design-and-ux/SKILL.md?utm_source=chatgpt.com)

---

### 3. Qt公式 `qt-ui-design`

これは今回かなり注目してよいものです。

Qt Company のR&Dチームが、現在 **公式のAgent Skillsセット**を公開しており、その中に

`qt-ui-design`

があります。

画面構成、ナビゲーション、UXレビュー、入力方法、画面サイズ、ロケールなどを考慮してQt UIを設計するためのスキルです。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/theqtcompanyrnd/agent-skills?utm_source=chatgpt.com)

しかも公式リポジトリでは、

> Claude Code / Codex / GitHub Copilot / Gemini

向けとして明示されています。GitHub Copilot なら `.github/skills/`、個人用なら `~/.copilot/skills/` に配置できます。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/theqtcompanyrnd/agent-skills?utm_source=chatgpt.com)

これはかなり有力です。

---

### 4. `pyqt` スキル群

**PySide6 + Qt Widgetsを使うなら、実装面では一番重要かもしれません。**

`CodeAtCode/oss-ai-skills` にかなり細分化されたQtスキルがあります。

* `pyqt-core`
* `pyqt-widgets`
* `pyqt-threading`
* `pyqt-dialogs`
* `pyqt-testing`
* `pyqt-styling`
* `pyqt-multimedia`

という構成です。PyQtという名前ですが **PySide6も対象**です。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub+1](https://github.com/codeatcode/oss-ai-skills?utm_source=chatgpt.com)

特に重要なのが、

**`pyqt-widgets`**

です。

QMainWindow、QSplitter、QTabWidget、QTreeView、レイアウトなど、実際のデスクトップGUI構築を担当させられます。

---

### 5. `pyqt-threading`

これはGUIソフトでは意外と重要です。

AIに普通にGUIを書かせると、

```
ボタンを押す
↓
重い解析処理
↓
GUIが固まる
```

というコードを作ってしまうことがあります。

Qtでは

* `QThread`
* `QThreadPool`
* `QRunnable`
* signals/slots

などを正しく使う必要があります。

`pyqt-threading` はこの領域に特化しています。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/codeatcode/oss-ai-skills?utm_source=chatgpt.com)

数値解析、ファイル読み込み、大量データ処理などを伴うGUIなら重要度はかなり高いです。

---

### 6. `pyqt-testing`

これも強く推奨します。

`pytest-qt` を使って、

```
ボタンを押す
↓
signalが発火する
↓
画面の値が更新される
```

といったGUI挙動をテストできます。

`qtbot.mouseClick()`、`waitSignal()`、keyboard input、dialog testなどのパターンがスキルに含まれています。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/CodeAtCode/oss-ai-skills/blob/master/frameworks/pyqt/testing/SKILL.md?utm_source=chatgpt.com)

つまり、

**Superpowers TDD**  
+  
**pyqt-testing**

という組み合わせができます。

---

### 7. `pyside6-mcp`

GUI開発において、私はこれをかなり面白いと思っています。

AIによるGUI開発が難しい最大の理由の一つは、

> AIが実際のGUIを見て触れない

ことだからです。

`pyside6-mcp` は、

**PySide6版Playwright**

のようなものです。

AIが実際に、

```
GUI起動
↓
スクリーンショット取得
↓
Widget tree取得
↓
ボタンを探す
↓
クリック
↓
入力
↓
スクリーンショット
↓
ログ確認
```

という操作をできます。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/com55/pyside6-mcp/blob/master/skills/pyside6-mcp/SKILL.md)

たとえば、

```
find_widget(text="Apply")
click(...)
wait_for_idle()
screenshot()
```

のように操作できます。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/com55/pyside6-mcp/blob/master/skills/pyside6-mcp/SKILL.md)

これは**GUI調整のAI自律性を大きく上げる可能性があります**。

ただし、これは単純なSKILL.mdだけではなくMCPとの組み合わせなので、導入環境は確認した方がよいです。

---

### 8. `pyside6-fluent-ui`

見た目をWindows 11 / Fluent 2風にしたい場合の選択肢です。

PySide6 + Qt Widgetsに対して、

* Fluent 2
* light / dark
* high contrast
* QSS
* accessibility
* Activity Bar
* Sidebar
* Status Bar

などを扱います。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/ejacques11/pyside6-fluent-ui)

MITライセンスです。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/ejacques11/pyside6-fluent-ui)

ただし、これは**非常に新しい小規模プロジェクト**で、現時点のGitHub表示では4 commits、0 starsです。そのため、私は「基盤スキル」ではなく**見た目を整える補助スキル**として使います。 [![](https://www.google.com/s2/favicons?domain=https://github.com&sz=128)GitHub](https://github.com/ejacques11/pyside6-fluent-ui)

---

## 私ならこう構成します

Python + PySide6 + GitHub Copilotなら、まずはこの6つです。

```
Superpowers
        │
        ├── product-design-and-ux
        │
        └── qt-ui-design
                │
                ▼
        pyqt / pyqt-widgets
                │
        ┌───────┴───────┐
        ▼               ▼
 pyqt-threading    pyqt-testing
        │               │
        └───────┬───────┘
                ▼
          pyside6-mcp
        実GUIで動作確認
```

これなら、

**①何を作るか**  
→ Superpowers

**②どう操作させるか**  
→ product-design-and-ux

**③Qt GUIとしてどう設計するか**  
→ qt-ui-design

**④どう実装するか**  
→ pyqt系

**⑤固まらないようにする**  
→ pyqt-threading

**⑥挙動をテスト**  
→ pyqt-testing

**⑦実際のGUIをAI自身が見て調整**  
→ pyside6-mcp

というきれいな役割分担になります。

GitHub Copilot自体も現在 `.github/skills`、`.claude/skills`、`.agents/skills` をProject Skillとして正式サポートしているので、このような複数スキル構成はかなりやりやすくなっています。 [![](https://www.google.com/s2/favicons?domain=https://docs.github.com&sz=128)GitHub Docs](https://docs.github.com/ja/copilot/concepts/agents/about-agent-skills?utm_source=chatgpt.com)

**結論として、デスクトップGUI向けの最重要セットは `Superpowers + product-design-and-ux + qt-ui-design + pyqt系 + pyside6-mcp` です。**

特に前回話題になった「AIにGUIを作らせても微調整がなかなか思い通りにならない」という問題には、**デザインスキル追加以上に `pyside6-mcp` のような視覚フィードバックループを入れることが効く**と思います。