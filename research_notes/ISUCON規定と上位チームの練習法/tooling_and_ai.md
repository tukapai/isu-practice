# ISUCON上位チームのツール群と生成AI/AIエージェント活用（調査日: 2026-10-31）

> 前提の注意: 依頼文では「ISUCON13〜15」とされていたが、公式Blogのアーカイブは ISUCON14（2024年12月）の次が **ISUCON2026**（2026-10-31開催、本日）になっており、ISUCON15の結果・講評・まとめは公式サイト上に見つからなかった（[isucon.net トップ](https://isucon.net/)）。2025年に開催されなかったのか、名称が変わったのかは公式には明記されていない（**未確認**）。運営は2026年にLINEヤフーからさくらインターネットへ移管された（[ISUCON2026 開催決定](https://isucon.net/archives/59917350.html)）。以下、「直近の本番」は ISUCON13 / 14 / 2026 を指す。

## 計測・プロファイリングツールと典型的な配線

### Takeaway
定番は「nginxアクセスログ(LTSV/JSON)→alp」「MySQL slow log(long_query_time=0)→pt-query-digest / slp」「Go pprof(flame graph)」の3点セットで、これらを1画面で収集・比較する **kaz/pprotein** が近年の標準的ダッシュボードになっている。ベンチ開始(initialize)をトリガに自動収集する配線が要点。

### Cited Findings
- pprotein は「Performance monitoring utilities for API server written in Go」で、pprof / fgprof / alp / slp と連携する。MIT、Go製、UI(`view`)とCLI・integration パッケージを含む — [kaz/pprotein](https://github.com/kaz/pprotein)
- pprotein の典型セットアップ手順（ISUCON11予選題材）:
  - `go install github.com/tkuchiki/alp/cmd/alp@latest` と `github.com/tkuchiki/slp/cmd/slp@latest` を導入
  - nginx に LTSV の `log_format`（`reqtime`, `apptime`=`$upstream_response_time`, `uri`, `method`, `status` 等）を定義し `access_log ... ltsv`
  - MySQL: `slow_query_log_file=/var/log/mysql/mysql-slow.log`, `long_query_time=0`, `slow_query_log=ON`（SET GLOBAL は再起動で消えるので my.cnf に永続化）
  - Goアプリに `github.com/kaz/pprotein/integration/echov4` を入れ `echov4.EnableDebugHandler(e)`、`POST /initialize` ハンドラ内で goroutine から `http://localhost:9000/api/group/collect` を叩いてベンチ開始と同時に収集
  - pprotein本体はポート9000、SSHポートフォワードでUI閲覧。収集対象は `/debug/pprof/profile`、`/debug/log/httplog`、`/debug/log/slowlog`（各60秒）
  - httplog 設定に alp の `matching_groups`（例 `^/api/isu/.+[0-9a-f\-]$`）を入れて動的パスを集約
  — [pprotein getting started (HackMD, to-hutohu)](https://hackmd.io/@to-hutohu/pprotein-getting-started)
- alp は LTSV / JSON / 正規表現 / pcap を入力にでき、`-m/--matching-groups` でURI集約、`--sort`（count/max/avg/p90 等）、`-r`、`--dump`/`--load` でプロファイル保存、`alp diff` で2回分の差分比較ができる — [tkuchiki/alp](https://github.com/tkuchiki/alp)
- ISUCON13参加チームの実例: pprotein を「すんごい便利」と評価。slp のスロークエリ出力を見てインデックス追加（約1万点）、pprof flame graph から `getReactionsHandler` や `sha256.Sum256` を特定し N+1解消・`sync.Map` キャッシュへ — [SODA Engineering Blog (Zenn)](https://zenn.dev/team_soda/articles/20231126195358)
- 「ISUCONチートシートv2022」など、計測設定をまとめたチートシート文化がある — [HackMD](https://hackmd.io/@to-hutohu/SJuUqzdsc)
- APM/オブザーバビリティSaaSの試行例:
  - Splunk: ISUCON14 を OpenTelemetry Collector（host/MySQL/nginx メトリクス+ログ）+ Go手動スパン（HTTP, SQL, コネクションプール）で計測し Cursor が計装と改修を担当。903→59,657点。決済API呼び出し中にDBトランザクションを保持していた問題を発見し修正（53,192→56,925）。一方で SQL/ランタイム/決済スパンを常時ONにすると 50,412 に低下＝**計装のオーバーヘッドがスコアを削る**ので、最終計測時は計装を外しCollectorを止めるよう推奨（2026-10-21）— [Splunk Blog](https://www.splunk.com/ja_jp/blog/observability/isucon-rethink-observability-in-the-age-of-ai.html)
  - Mackerel: ISUCON9予選を mackerel-agent（Docker plugin）+ OTel or OBI(eBPF, コード変更不要)のAPMトレース + Mackerel MCP Server で計測。約405万スパンで無料枠（月500万）内（2026-01-22）— [Mackerel Blog](https://mackerel.io/ja/blog/entry/tech/ai-isucon)
  - New Relic はISUCON12にスポンサー枠で出場しレポートを公開している — [New Relic Blog](https://newrelic.com/jp/blog/news/isucon12)
- 公式レギュレーションはモニタリング用途の外部サーバー利用を許可（「モニタリングやテスト、開発などにおいては、PCや外部のサーバーを利用しても構わない」）。ベンチ処理の外部委譲は禁止 — [ISUCON2026 レギュレーション](https://isucon.net/archives/59966826.html)、[ISUCON13 レギュレーション](https://isucon.net/archives/57768216.html)

### Inferences
- pprotein の「initialize で collect を叩く」配線は、自律エージェントにとっても「1ベンチ=1計測グループ」という単位を自然に作るため、ハーネスの観測層としてそのまま流用しやすい。
- APM（OTel/Splunk/Mackerel/Datadog/New Relic）は「チームで同じ画面を見る」「履歴比較」に強いが、計装コストでスコアが落ちるため、本番計測時は外す運用が必須。
- alp の `--dump` / `alp diff` は改善前後比較をテキストで出せるので、LLMに渡す要約として相性が良い。

### Gaps
- pt-query-digest / mysqldumpslow / fgprof / Pyroscope / kataribe / netdata について、今回の調査で一次資料（ISUCONでの使用記事）を取得・確認できなかった。一般知識として pt-query-digest（Percona Toolkit）と kataribe（nginxログ集計）は旧来の定番、netdata/Pyroscope はリアルタイム可視化・継続プロファイリング用途として知られるが**本ノートでは未検証**。
- pprotein README 本体（agent構成・memo機能など）は取得失敗。詳細は HackMD 手順のみに依拠。
- Datadog のISUCON試用記事は未確認。

## デプロイ/自動化テンプレート

### Takeaway
各チームは「Makefile or シェルスクリプトで build→rsync/配置→systemctl restart→ログローテート→ベンチ」を1コマンド化したテンプレートを用意し、GitHubで公開している（ansible 版、docker化ツール群版もある）。AI時代の新潮流は「状態を完全にリポジトリ化＋スナップショットから復元可能」にし、エージェント同士の deploy/bench 衝突を防ぐロックを入れること。

### Cited Findings
- 公開テンプレート例: [itohdak/isucon_ansible](https://github.com/itohdak/isucon_ansible)（ansible）、[EkeMinusYou/isucon-template](https://github.com/EkeMinusYou/isucon-template)、[kitakou0313/isucon-tools](https://github.com/kitakou0313/isucon-tools)（ISUCONで使うツール群をdockerで使えるようにまとめたtemplate）、[isucon11q向けMakefile (gist)](https://gist.github.com/Hiroya-W/b5f84ac8938f53742f4d0f84f8958fbc)、[tmokmss/isucon12q-infra](https://github.com/tmokmss/isucon12q-infra)、優勝チームの [narusejun/isucon12-final](https://github.com/narusejun/isucon12-final) — 各GitHub（検索結果より。中身の詳細は未精査）
- [tatsumack/isucon-tools PR#3](https://github.com/tatsumack/isucon-tools/pull/3) では「ISUCON2026当日の方針・作業宣言ルール・タイムライン」を todo.md に追記しており、ツールだけでなく**当日の運用ルール**もテンプレ化されている（PRタイトルより）
- ISUNARABE合同演習2026（AI無制限）での ichyo 氏の基盤: 開始時に点数に影響するすべての設定とコードをリポジトリに入れ、DB snapshot から毎回同じ状態に戻せるようにした。AIはサーバーを直接触らず「tar.gz artifact 作成→アップロード→専用agentでdeploy→bench→PR作成」のループ。複数AIの同時deploy衝突を防ぐ管理サービス「isudeck」+CLIを自作し、運用ルールは AGENTS.md に記述 — [ichyo::blog](https://blog.ichyo.jp/posts/isunarabe2026)
- rail44 氏: sccache / mold / rsync によるリモート差分ビルド、MySQL・nginx設定の事前準備と適用スクリプト、sftpマウント、ベンチ実行→レポート・slow log収集→分析エージェントへ渡す自動化を準備 — [Zenn rail44](https://zenn.dev/rail44/articles/932a22e70ef98d)
- overcached! チーム: 人間がベンチ・アクセスログ・スロークエリログ・deploy&check の基盤を作り、その上で Claude Code に実装させた。複数人のエージェントが互いのベンチ中にデプロイする事故があり、deploy/bench ロックと運用ルールが必要と総括 — [edge2992](https://tech.edge2992.dev/posts/20260510_isucon_nrb2026/)

### Inferences
- 人間チーム用テンプレ（Makefile）は「1人が順番に叩く」前提。エージェント並列時代は **排他ロック・成果物（artifact）単位のデプロイ・スナップショット復元・PR単位の履歴** がテンプレの必須要素になりつつある。

### Gaps
- 「isucon-setup」等の特定名スクリプトの一次情報は未確認。上記テンプレの具体的中身（Makefileターゲット等）は精査していない。

## AI利用に関する公式ルールと実際の使われ方

### Takeaway
公式レギュレーションは ISUCON13 以降「開発において、AIを活用したコードの分析、生成等を利用しても構わない」と明記しており AI 利用は合法。ISUCON14 では Copilot/Cursor の利用が多数あったと公式講評が言及。2026年の非公式演習（ISUNARABE、AI無制限）では Claude Code / Codex 主体のチームが1000万〜2000万点台に到達し、AI前提の戦い方が実証された。

### Cited Findings
- ISUCON13 レギュレーション禁止事項内: 「開発において、AIを活用したコードの分析、生成等を利用しても構わない。」 — [ISUCON13 レギュレーション](https://isucon.net/archives/57768216.html)
- ISUCON2026 レギュレーション: 「変更してよいもの」に「開発において、AIを活用したコードの分析、生成等を利用すること」、禁止事項にも同文の例外。チームは1〜3名、合計1000チーム上限、競技 2026-10-31 10:00-18:00。ベンチ中の外部リソース利用（他インスタンスへの処理委譲）は禁止。競技中にチーム外の人と問題について情報交換することは禁止（外部AIサービスが該当するかは明記なし）。当日マニュアルがレギュレーションに優先 — [ISUCON2026 レギュレーション](https://isucon.net/archives/59966826.html)
- ISUCON2026 は 856チーム（学生83）、1,596名参加。結果・講評は本日時点で「準備中」 — [isucon.net](https://isucon.net/)
- ISUCON14 公式講評: 「昨年よりも格段に強力になった生成AIを利用しているチームも数多くありました」「GitHub CopilotやCursorなどを利用している方も多くいたのではないかと思います」、主に「SQLクエリやアプリケーションコードの改善」に使われた。優勝は1人チーム takonomura（2回目）。講評は2024-12-20付 — [ISUCON14 問題の解説と講評](https://isucon.net/archives/58869617.html)
- ISUCON14 での具体例（nakario / 百万円ドリブン）: GitHub Copilot（VS Code）の Copilot Edit を SQL・Goのエラーハンドリング・N+1解消・キャッシュ導入に使用。長いファイルではEditが全書き換えで約54秒→関数単位にファイル分割するツール **fsplit** で約5秒に短縮。分割で型定義が見えなくなる問題を依存ファイル列挙ツール **depf** で補完。ただし本選結果は振るわず「ツールを十分活用できなかった」— [nakario Blog](https://nakario.hateblo.jp/entry/2024/12/10/025218)
- スポンサーLayerXのISUCON2026応援メッセージ: 「AIの変更を最後に責任もって出すのは今のところ人間」 — [isucon.net](https://isucon.net/)（2026-10-29掲載）
- ISUNARABE合同演習2026（非公式、2026-05-09 10:00-18:00、1〜3名でAIは人数に含めない、AI無制限、ベンチ中の外部リソースは禁止、参照実装はRustのみ、賞金なし）。趣旨は「全員が同じコードでAIをフル活用したら何が起こるか」の共通ベンチマーク作り — [Write and Run](https://diary.hatenablog.jp/entry/2026/04/13/110000)
- 同演習の問題作成者（KOBA789）は5日間でほぼ1人＋AIで作問: Claude Code（Opus 4.7）を主軸に53メインセッション＋10 worktreeセッション、Codex CLI をセカンドオピニオンレビューに24回、subagent並列で過去ベンチマーカーを調査。「中核の設計判断は人間、形式化・細部はAI」「記憶はバージョン管理されたdocsに」「軽量プリミティブ（worktree, subagent, cross-model review）が重い構造化ワークフローより試行錯誤向き」と総括 — [nrb2026 POSTMORTEM](https://github.com/KOBA789/nrb2026/blob/main/POSTMORTEM.md)
- 同演習の参加者結果:
  - ichyo 氏: Codex(GPT-5.5)のみ（重要局面は reasoning xhigh、それ以外 medium）。39,100 → メモリ常駐Rust APIへ作り直し 17,464,000 → 通知バックグラウンドキュー化 20,310,000 → 通知ペーシング 21,609,000（最終記載 21,396,000）— [ichyo::blog](https://blog.ichyo.jp/posts/isunarabe2026)
  - rail44 氏: 多くのチームが1000万点超、上位は単一サーバーのインメモリ化で約1500万〜2000万点。自チームは3台シャーディングでロック競合・タイムアウト→最終0点 — [Zenn rail44](https://zenn.dev/rail44/articles/932a22e70ef98d)
  - overcached!（3名、Claude Codeのみ、Max 5x プランの5時間枠1つ分程度消費）: 14時頃に100万点、17時頃まで200万点前後で停滞、人間がログ分析・方針決定し Claude Code が実装する分担へ切替え、最終 5,176,800点・9位 — [edge2992](https://tech.edge2992.dev/posts/20260510_isucon_nrb2026/)

### Inferences
- 公式ルール上「開発でのAI利用」は明示的に許可。ただし「チーム外との情報交換禁止」と外部AIの関係は文言上グレーで、当日マニュアルの確認が必要（ISUCON2026当日マニュアルは未確認）。
- ISUCON14（2024年）時点は Copilot/Cursor による「部分的なコード生成・SQL改善」が中心、2026年には Claude Code / Codex による「エージェント主導の実装・デプロイ・ベンチ」に移行している。
- AI無制限環境でのスコア分布は「インメモリ化などの大胆な設計判断ができたか」で桁が分かれ（数百万点 vs 2000万点）、AIの実装速度より**人間（またはハーネス）の戦略判断**が差を生んでいる。

### Gaps
- ISUCON2026 本番（本日）の結果・上位チームのAI使用状況は未公開（講評「準備中」）。
- ISUCON13・14 上位入賞チーム自身による「AIをどれだけ使ったか」の定量的記述は見つからなかった。
- 「AI vs 人間」の公式イベントや LINEヤフーによるAI実験は見つからなかった（LINEヤフー関連の検索は社内AIエージェント施策の記事のみ）。connpass「帰ってきたISUCON'26 AI時代の楽しみ方・戦い方」（レバテックラボ）は存在するが内容は取得不可（robots拒否）— [connpass](https://levtechlab.connpass.com/event/400761/)

## 自律型性能改善エージェント/ハーネス構築への示唆

### Takeaway
成功例に共通するループは「計測→仮説→小さな変更→再現可能なデプロイ→ベンチ→keep/discard→履歴記録」。過去問（ISUCON9予選）なら完全自律で1,810→50,280点まで伸びた一方、本番級の問題では「保守的すぎる改善」「ISUCON特有のトレードオフ判断」「並列エージェントの衝突」で頭打ちになり、人間の戦略判断が依然カギ。

### Cited Findings
- 完全自律の事例（Mackerel, 2026-01-22）: ISUCON9予選（約2,500行・2ファイル）を Claude Code + Mackerel MCP Server で、プロンプトに「目標（スコア最大化）・使えるツール・発見/根拠/変更/スコア影響の記録」を与え人手介入なしで反復。P95レイテンシとエラー率から「外部API呼び出し中のロック保持」を特定しトランザクション外へ移動。**1,810 → 50,280点**。教訓: 変更前に計測、小さいコードベース＋整合性チェック＋定量スコアがあればAI主導最適化は実用的 — [Mackerel Blog](https://mackerel.io/ja/blog/entry/tech/ai-isucon)
- 自律PDCAハーネス設計例（ISUCONではなく自社Railsエンドポイント, 2026-03-30）: Karpathy の autoresearch に倣い `/loop 10m` で1サイクル9分予算、opus の tuning-agent と sonnet の data-setup-agent に分離、`tuning_results.tsv` に commit/avg_ms/p95/specs/status を蓄積し次サイクルが全履歴を読む。改善時のみkeep、spec失敗は3回まで修正→revert。Read hook でベンチ・シードを見せない（ベンチへの過学習防止）、Bash はホワイトリスト、編集可能範囲を限定、専用ブランチのみ。約25.7%改善 — [Zenn dely](https://zenn.dev/dely_jp/articles/3117e590465e38)
- 失敗/限界の報告:
  - 「Claudeの提案はこの種のコンテストには慎重・漸進的すぎる」「後方互換性や保守性は不要と何度も言う必要があった」「並列エージェント間の通信が慎重さを強化した」「思考は外注できても理解は外注できない」 — [Zenn rail44](https://zenn.dev/rail44/articles/932a22e70ef98d)
  - 100万点以降、ログだけからAIが高インパクト改善を出し続けられない。DBロック→アプリ側mutex置換は単一サーバーでは有効だが複数台で壊れる、といったISUCON特有の判断に弱い。CLAUDE.md に文脈と判断基準を書くべき — [edge2992](https://tech.edge2992.dev/posts/20260510_isucon_nrb2026/)
  - エージェントが未検証の前提を積み上げる（調査レポートに推測のISUCON慣習が混入）。悪い出力を直させるより問題を再定義した方が効く — [nrb2026 POSTMORTEM](https://github.com/KOBA789/nrb2026/blob/main/POSTMORTEM.md)
  - 通知並列数を8に上げるとTCP active open/retransmit増でスコア低下、profileで初期化コストをhot pathと誤認、PR bodyのバッククォートがshellに解釈される等の事故 — [ichyo::blog](https://blog.ichyo.jp/posts/isunarabe2026)
  - ベンチ開始が2時間遅延したことにエージェントが気づかなかった — [Zenn rail44](https://zenn.dev/rail44/articles/932a22e70ef98d)
- 効いたプロンプト運用: 設計を先に出させる、deploy/benchを止めて考察だけのターンを作る、役割と段階を明確化。ログで仮説を検証（join 73,849件中64,819件が409→残り1枠への殺到を発見；「既参加者への無駄な通知」仮説は0人で棄却）— [ichyo::blog](https://blog.ichyo.jp/posts/isunarabe2026)
- 計装はスコアを下げる（常時スパンで 56,925→50,412 相当の低下）ため、調査時のみ計装・最終計測は計装OFF — [Splunk Blog](https://www.splunk.com/ja_jp/blog/observability/isucon-rethink-observability-in-the-age-of-ai.html)

### Inferences
- ハーネス設計の要件（上記からの推論）:
  1. **再現性**: 全設定をリポジトリ化、DBスナップショット復元、artifact単位デプロイ。
  2. **排他制御**: deploy/bench のロック（isudeck 型）。並列エージェントはworktreeで隔離し、ベンチ実行権は1つに。
  3. **観測の標準化**: pprotein（alp+slp+pprof）のテキスト要約、`alp diff` で前後比較をLLMに渡す。最終計測は計装OFF。
  4. **履歴メモリ**: TSV/docs に「仮説・根拠・変更・スコア」を記録し毎サイクル読み込む。
  5. **戦略層の分離**: 「インメモリ化」「サーバー分割」「仕様上許容される遅延の活用」など大胆な設計変更は、ISUCON特化の判断基準（CLAUDE.md/AGENTS.md）か人間/上位エージェントが決める。保守性・後方互換は不要と明記。
  6. **ガードレール**: ベンチマーカーやシードを見せない／編集範囲を限定／コマンドホワイトリスト。
  7. **時間・状態の外部確認**: ポータルの開始時刻・ベンチ状態を定期確認する仕組み。
- 過去問自律（〜5万点）と、AI無制限演習の上位（2000万点級）の差は、「局所最適の積み上げ」vs「アーキテクチャ刷新」の差と解釈できる。自律エージェントの次の課題は後者の判断の自動化。

### Gaps
- Devin や Codex 単独の完全自律で過去ISUCON本番問題（ISUCON12〜14等）を解き、人間上位と比較した公開ベンチマークは見つからなかった。
- 同一問題での「AIあり/なし」や「自律/人間協調」の比較データは存在しない（Splunkも定性比較のみ）。
