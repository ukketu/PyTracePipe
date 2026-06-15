AIエージェントによる保守改修は今まさに研究が動いている領域なので、最新のベンチマーク結果や手法を確認してから体系化します。いくつか調べますね。

十分な材料が集まりました。ベンチマーク・実証研究・実務ツールの3層から、保守改修を軸に体系化します。

---

## 1. 現在地:ベンチマークが示す「得意/苦手」の輪郭

まず前提として、定番だったSWE-bench Verifiedはほぼ飽和しています。2026年6月時点で上位モデルは95%前後に達しており [BenchLM](https://benchlm.ai/benchmarks/sweVerified) 、さらに2026年2月にはOpenAIが「学習データ汚染により最前線の進歩を測れなくなった」としてVerifiedのスコア報告を停止しました [Morph](https://www.morphllm.com/swe-bench-pro) 。つまり「単一ファイル中心・小さなバグ修正」はもう解けた領域です。

一方、御社の課題に近い「複数ファイル・長時間の改修」を測るSWE-bench Proでは様相が一変します。プロのエンジニアでも数時間〜数日かかる長期タスクで構成され、発表時点では統一環境下でどのモデルも45%未満 [arXiv](https://arxiv.org/html/2509.16941v2) でした。重要なのは失敗原因の内訳で、軌跡分析によると、意味理解の失敗(Opus 4.1の失敗の35.9%)、コンテキスト溢れ(Sonnet 4の失敗の35.6%)、ツール使用の非効率(小型モデルの失敗の42%)に大別され、コーディングエージェントは時間の60%以上をコンテキスト探索に費やしている [Morph](https://www.morphllm.com/swe-bench-pro) ことが分かっています。

時間軸で見るとMETRの研究が示唆的です。人間が4分以内で終えるタスクではほぼ100%成功する一方、4時間超のタスクでは成功率が10%を切ります。ただし「50%の確率で成功できるタスク時間」は約7か月ごとに倍増し続けています [Zartis](https://www.zartis.com/the-compounding-errors-problem-why-multi-agent-systems-fail-and-the-architecture-that-fixes-it/) 。「大規模訂正でミスが多い」という体感は、この長期タスクの壁そのものです。

## 2. ボトルネックの正体 — なぜドキュメントは高精度で、品質改善は崩れるのか

ご指摘の非対称性は研究的にきれいに説明できます。本質は4つです。

**(a) タスクの数学的性質が違う。** ドキュメント生成は「読み取り→非可逆圧縮」で、許容される正解が無数にあり、1箇所の誤りも局所被害で済みます。対して品質改善は「振る舞いを厳密に保存したまま書き換える」制約付き問題で、正解集合が極端に狭い。さらに逐次実行の宿命として、1ステップの成功率が99%でも、100ステップ連鎖すると全体成功率は約37%まで落ちます [arxiv](https://arxiv.org/pdf/2512.03549) 。改修規模が大きいほど指数的に壊れるわけです。

**(b) 自己汚染(self-conditioning)。** コンテキスト内に自分自身の過去の誤りが含まれると、後続の誤り発生率が測定可能なレベルで上昇する [Zartis](https://www.zartis.com/the-compounding-errors-problem-why-multi-agent-systems-fail-and-the-architecture-that-fixes-it/) ことが示されています。長いセッションほど誤りが誤りを呼ぶ構造です。コンテキストロットは信号対雑音比の構造問題であり、ウィンドウを大きくしても劣化の開始が遅れるだけで根本解決にはなりません [MindStudio](https://www.mindstudio.ai/blog/context-rot-ai-coding-agents-how-to-prevent) 。

**(c) 検証オラクルの不在。** ドキュメントの誤りは人間が読めば検出できますが、リファクタリングの正しさはテストという機械的オラクルがないと誰も保証できません。EM-Assistの研究ではLLM単体のExtract Method提案は最大76.3%がハルシネーションにより不正確で、IDEの静的解析で文脈を与えると成功率が大きく改善した [arxiv](https://arxiv.org/pdf/2510.03914) と報告されています。

**(d) エージェントの行動傾向そのもの。** 15,451件の実改修を分析した大規模実証研究(Agentic Refactoring, 2025)によると、エージェントのリファクタリングは変数型変更(11.8%)・引数リネーム(10.4%)・変数リネーム(8.5%)といった低レベルで局所的な編集に偏っており、人間に多い高レベルの設計変更は少ない [Hugging Face](https://huggingface.co/papers/2511.04824) 。結果として構造メトリクスには統計的に有意だが小さな改善しか生まず、既知の設計スメルの総数を一貫して減らすことには失敗しています [arxiv](https://arxiv.org/pdf/2511.04824) 。さらにエージェントのリファクタリングの53.9%は機能追加やバグ修正と混在した「もつれたコミット」で発生し、レビュー負荷を隠れたコストとして増大させる [LinearB](https://linearb.io/blog/ai-coding-agents-code-refactoring) ことも分かっています。

加えて見落とされがちな要因が「コード側の健全性」です。CodeSceneのベンチマークでは、業界平均のCode Healthは5.15程度に過ぎず、AI導入が期待通りに機能しないのはLLMが弱いからではなく、平均的なコード構造がAIには不健全すぎて安全に変更できないため [Codescene](https://codescene.com/blog/making-legacy-code-ai-ready-benchmarks-on-agentic-refactoring) と結論しています。同調査では健全化されたコードベースでは同等タスクのトークン消費が約50%減る [Codescene](https://codescene.com/blog/making-legacy-code-ai-ready-benchmarks-on-agentic-refactoring) という実利も示されており、「品質改善のためにまず品質が要る」という鶏卵問題が存在します。

## 3. 有効と判明している手法の体系(保守改修向け4層モデル)

### 第1層:検証ハーネスを先に作る(オラクルの確立)

レガシー改修の古典(Feathersの特性試験)がエージェント時代に再発見されています。Red Hatのエージェントメッシュ事例では、コーディングエージェントがレガシーコードを解析し、何かを変更する前にまず元の振る舞いを捕捉する特性試験(characterization tests)を生成し、依存マッピングや移行計画は別エージェントが担い、実行フローは決定的なオーケストレーション論理が統制する [Red Hat](https://www.redhat.com/en/blog/refactoring-speed-mission-agent-mesh-approach-legacy-system-modernization-red-hat-ai) 構成を取っています。実務指針としてもテストなしでは振る舞い保存を検証する手段がないため、エージェントはまずテスト生成を行い、カバレッジの低いコードベースにはリファクタリング前のテスト追加を推奨すべき [CallSphere](https://callsphere.ai/blog/refactoring-agent-ai-powered-code-improvement-technical-debt-reduction) とされます。**「AIはテストを書くのは得意、テストがないと改修が壊れる」ので、得意分野で苦手分野の足場を作るのが最初の一手です。**

### 第2層:決定的ツールとのハイブリッド(LLMに直接コードを書き換えさせない)

大規模一括変更で最も成績が良いのは「LLMが計画・ルール生成、決定的エンジンが実行」の分業です。象徴的な反例として、OpenRewrite公式のLLM統合実験は出力が非決定的で、変更不要な箇所への誤変更も起き、単純なswitch並べ替えですら20回の反復実行で一貫した結果にならず、本番利用非推奨 [GitHub](https://github.com/openrewrite/rewrite-generative-ai) と自ら明記しています。逆に成功例では、Amazon QのJava大量アップグレードは実態としてOpenRewriteの決定的レシピが変更の大部分を担い、LLMは残りの複雑な箇所の補助に回っていた [Substack](https://ecosystem4engineering.substack.com/p/automated-refactoring-ai-versus-deterministic) と分析されています。Moderneも数千の既存OpenRewriteレシピをLLMのツールとして呼び出すハイブリッド [OpenRewrite](https://www.moderne.ai/blog/introducing-moderne-multi-repo-ai-agent-for-transforming-code-at-scale) に移行し、Codemod社はast-grepを決定的エンジンとし、その間にLLMを挟むことで、LLM単独より信頼性とスケーラビリティの高い変換を実現する [Codemod](https://codemod.com/blog/codemod2) アプローチを打ち出しています。

IDEレベルでも同じ思想が主流化しており、Kiroはエージェントがワークフローを統制しつつ実際のリファクタリング操作は言語サーバ(LSP)に委ね、「F2キーで動くものはエージェントでも動く」という実証済みインフラへの信頼と言語非依存性を確保する [Kiro](https://kiro.dev/blog/refactoring-made-right/) 設計です。**「リネーム・シグネチャ変更・移動」はLSP/レシピに、「判断・例外処理・レビュー」はLLMに**、が現時点の最適解です。

### 第3層:分割統治とコンテキスト工学

長期タスクの指数的失敗への対処は、構造的にしか解けません。複雑タスクを原子的サブタスクの有向非巡回グラフ(DAG)に分解し、各出力を下流のコンテキストに入る前に独立検証することで誤り連鎖を断つ [Zartis](https://www.zartis.com/the-compounding-errors-problem-why-multi-agent-systems-fail-and-the-architecture-that-fixes-it/) のが収束しつつある設計です。探索をサブエージェントに隔離する効果も定量化されており、専用検索サブエージェント(WarpGrep v2)を追加すると全モデルで2ポイント強の精度向上に加え、Opus 4.6ではコスト15.6%減・時間28%減となりました。メインモデルが棄却済みファイルを一切見ないためコンテキストが汚れない [Morph](https://www.morphllm.com/swe-bench-pro) のが効く理由です。なお反復的な仕様変更下でコード品質が劣化していく現象は、プロンプト側の圧力では抑制できない [arXiv](https://arxiv.org/html/2603.24755v1) ことも示されており、「丁寧に頼む」では解決しない=セッション分割・コンテキストリセットというアーキテクチャ対処が必要です。

### 第4層:ドキュメント→仕様アンカー(あなたの強みを改修の入口にする)

ここが一番お伝えしたい点です。「ドキュメントは高精度」という現状は、そのまま改修精度を上げる資産になります。2026年に急速に普及したSpec-Driven Development (SDD)のブラウンフィールド適用がまさにこれで、2026年2月のSDD基礎論文は「レガシーコードから仕様を抽出することで、モダナイゼーションが必要機能を保存しつつ未文書化の挙動を排除できることを検証可能になる」とし、新規仕様を書く前に既存挙動の再構築を行うフェーズを置く [Augment Code](https://www.augmentcode.com/guides/what-is-spec-driven-development) ことを推奨しています。実践報告でも仕様は実装全体を通じてエージェントを整合させる「一貫性のアンカー」として機能し、既存システムでは「Xファイルのパターンに従え」「既存のYサービスを使え」と明示することでエージェントが独自方式を発明する傾向が劇的に減り、既存コードベースに馴染むコードが出てくる [Alex Cloudstar](https://www.alexcloudstar.com/blog/spec-driven-development-2026/) とされます。標準的な流れはSpecify→Plan→Tasks→Implementの4フェーズで各段に人間のチェックポイントを置き、主要ツールはGitHub Spec Kit、AWS Kiro、Claude Code skills、Cursor Plan Mode、OpenSpec、BMAD-METHODなど [BCMS](https://thebcms.com/blog/spec-driven-development) です。仕様が退職とともに消える設計判断を捕捉し、レガシーへの危険な外科手術の代わりに「仕様を変更してエージェントに再生成させる」運用 [Augment Code](https://www.augmentcode.com/guides/spec-driven-development-ai-agents-explained) へ移行できるのが長期的な狙いです。

## 4. Skillsには何を入れるべきか

ご質問の「局所ツールの使い方 vs フレームワークのアイディア提供」でいうと、研究と実務の答えはかなり明確に**前者+手続き的知識**です。理由はコンテキスト経済にあります。スキルは3層構造で、起動時には名前と1行説明だけ(Anthropic公式17スキルの中央値で約80トークン)が読み込まれ、関連時に本文、さらに必要時のみ参照資料が読まれます [Swirlai](https://www.newsletter.swirlai.com/p/agent-skills-progressive-disclosure) 。逆に大量のインライン参照資料を含む下手なスキルは、毎タスク冒頭で不要トークンを読み込ませ、コンテキスト劣化をむしろ加速させます [MindStudio](https://www.mindstudio.ai/blog/context-rot-ai-coding-agents-how-to-prevent) 。フレームワークの一般知識はモデルが既にパラメータとして持っているので、Skillsに書く価値が薄い(むしろ害)のです。位置づけとしてはMCPが「配管」(接続の定義)、Skillsが「マニュアル」(その接続をどう使い目的を達成するかの手順書) [financialcontent](https://markets.financialcontent.com/workboat/article/tokenring-2025-12-25-anthropic-unveils-agent-skills-open-standard-a-blueprint-for-modular-ai-autonomy) という整理が分かりやすいです。

保守改修用なら、具体的には次を入れると効きます。

- **トリガー語を含む description**:「リファクタリング」「○○モジュールの改修」「テスト追加」など、発火条件になる語彙を1024文字以内で具体的に。スキルが読まれるかどうかはここで決まります。
- **手順+チェックポイント**:複雑な操作は明確な逐次ステップに分解し、特に複雑なワークフローでは、進捗に応じてチェックを付けられるチェックリストを提供する [GitHub](https://github.com/obra/superpowers/blob/main/skills/writing-skills/anthropic-best-practices.md?plain=1) のが公式ベストプラクティスです。例:「①影響範囲をgrep→②特性試験生成→③テストGREEN確認→④変更→⑤テスト再実行→⑥diff要約」。
- **検証コマンドと合格基準**:ビルド・テスト・lintの正確なコマンド、何がGREENなら次に進めるか。エージェントの自己検証ループの質はここで決まります。
- **scripts/ に決定的処理を同梱**:ast-grepルール、codemodスクリプト、移行用のシェル等。「LLMに毎回考えさせず、書いたスクリプトを実行させる」のが第2層の思想のSkills版です。
- **プロジェクト固有の規約と禁止事項**:命名規則、レイヤ構造、「触ってはいけないファイル」、過去にエージェントがやらかしたアンチパターン。これは汎用知識でなく御社固有なので、Skillsに書く価値が最大の情報です。

なお常時必要な短い制約(コーディング規約の核心など)はCLAUDE.md/AGENTS.md、コンテキスト隔離が必要な探索や検証はサブエージェント、強制したいゲート(コミット前テスト必須など)はhooksと役割分担します。Skillsはオープン標準(agentskills.io)としてClaude以外にCodex、Gemini CLI、Copilot、Cursorなど26以上のプラットフォームに採用されている [Strapi](https://strapi.io/blog/what-are-agent-skills-and-how-to-use-them) ので、ここに知見を蓄積すればツールを乗り換えても資産が持ち運べます。

## 5. リードタイム短縮の実践パイプライン(提案)

以上を御社の状況(ドキュメント◎、品質改善△)に当てはめると、こういう順序が最短距離だと思います。

**Phase 1 — 資産化**:エージェントに既存コードからドキュメント+仕様(振る舞い記述)を抽出させる。すでに得意な領域なので低リスク。同時にコードベースの健全性スコアリング(複雑度・スメル計測)を入れ、「AIが安全に触れる場所/触れない場所」の地図を作る。

**Phase 2 — オラクル構築**:カバレッジの低いホットスポットに特性試験・ゴールデンマスターテストをエージェントで量産。ここもAIの得意領域です。

**Phase 3 — 改修の二刀流**:機械的変更(API移行、リネーム、依存更新)はOpenRewrite/codemod/ast-grep+LSPで一括実行し、LLMはルール作成とレビューに回す。判断を要する変更は、Phase 1の仕様をアンカーに、1セッション=1サブタスク(DAGの1ノード)単位で実行し、各ノードでテストGREENを通過させてから次へ。リファクタリングは機能変更と必ず別コミットに分離(もつれコミット対策)。

**Phase 4 — Skills化**:Phase 3で安定したワークフローをSkillに固めて再現性を確保。レビュー観点もSkill化すれば、レビュー側エージェントにも使えます。

## 6. 全体の議論として

生成とエージェント全般に視野を広げると、業界の重心は「生成能力」から「検証能力」へ移っています。コードを書く速度はもうボトルネックではなく、**正しさを機械的に確認できる範囲がエージェントの活動限界を決める**、というのが2026年の共通認識です。だからこそテスト・型・静的解析・仕様という「検証資産」への投資が、そのままAI活用度に変換されます。また、健全なコードはAIにとっても扱いやすくトークン消費も半減する [Codescene](https://codescene.com/blog/making-legacy-code-ai-ready-benchmarks-on-agentic-refactoring) という知見は、「品質改善はAI活用の前提投資でもある」という二重の意味を持ちます。御社の場合、ドキュメント生成の精度が高いという事実自体が、コードベースがAIに「読める」ことの証拠なので、検証ハーネスと決定的ツールを足せば改修側も伸びる素地は十分あると見ます。

もし次の一歩を具体化するなら、お使いの言語・スタック(Java系ならOpenRewrite、JS/TS/Pythonならast-grep+codemodが第一候補)と現在のテストカバレッジ状況を教えていただければ、Phase 1〜3に対応するSkillテンプレートとワークフロー定義のたたき台を作れます。
