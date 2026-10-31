# 過去ISUCON（ISUCON9〜ISUCON14、およびISUCON2026）のお題・ボトルネック・優勝チームと勝ち筋

調査日: 2026-10-31。一次情報は isucon.net の公式「まとめ」「結果」「解説と講評」「言語比率」記事を優先した。不確かな点は【要確認】と記した。

## 各回のお題・主なボトルネック・上位チームの決定打

### Takeaway
ISUCON9〜14では毎回、序盤はインデックス不足とN+1の解消で数倍伸ばし、中盤以降は「その問題固有の重い処理」を潰した上位チームが勝っている。固有の処理とは、外部API・ロック、空間検索、大量INSERT、SQLite分割、DNS、マッチングアルゴリズムなど。複数台構成への分割（DB分離・機能別振り分け・シャーディング）も、ほぼ毎回上位の必須要素になっている。

### Cited Findings

#### ISUCON9（2019）
- 開催概要: 予選は2019/9/7・9/8のオンライン2日制で、598チーム。本選は2019/10/5にLINE本社で行われ、32チーム。予選の作問はメルカリ、本選の作問はさくらインターネット、インフラはAlibaba Cloud — [ISUCON9 まとめ](https://isucon.net/archives/53570241.html)
- **予選「ISUCARI」**: 椅子のフリマアプリで、決済APIと配送APIという2つの外部サービスと連携する。サーバは3台まで。スコアは売上（取引完了した商品価格の合計）から減点を引いて計算される — [ISUCON9予選 解説](https://isucon.net/archives/53789931.html)
  - ボトルネック1: 商品一覧系でインデックス不足とN+1が起きていた。カテゴリ情報は更新されないのでコードに直接持たせられる。取引一覧の `buyer_id=? OR seller_id=?` はUNIONに書き換える — 同上
  - ボトルネック2: 配送APIの遅延。必要な呼び出しは非同期・並列化し、不要な呼び出しは削る（ドキュメントを読むと不要とわかる） — 同上
  - ボトルネック3: 購入時、トランザクション内で外部APIを呼ぶためロック待ちが起きる。商品単位のmutexやflockで排他する — 同上
  - ボトルネック4: bcryptが1回約0.1秒かかる。ログイン専用サーバを置く案が示されていた — 同上
  - 隠し要素: ユーザーの購入カテゴリに合わせて新着一覧を返すと、数%スコアが上がる — 同上
  - 予選は2日とも学生の1人チームがトップだった — 同上
  - 本選進出ラインは最終スコア9,880点以上（参加者ブログの記述で、公式値ではない）【要確認】 — [VELTRA参加記](https://medium.com/veltra-engineering/isucon9-cf382a76cf99)
- **本選「ISUXPRESS予約」**: 新幹線の座席予約サイト。区間ごとに座席の空き状況を管理する必要がある — [ISUCON9本選 解説](https://isucon.net/archives/53916974.html)
  - 初期実装は列車検索がタイムアウトしてスコア0点。列車検索・座席検索でN+1、予約処理で広範囲ロック、キャンセルでは外部決済APIが約1秒かかる — 同上
  - 想定解法は次のとおり — 同上
    - JOINやバッチ取得でクエリ数を減らす
    - `train_timetable_master(date, train_class, train_name, station)` にインデックスを張る（これだけで約1,500点）
    - 区間の重なり判定をSQLに寄せる
    - ロックを車両単位に絞る
    - 決済キャンセルを約5秒ごとにバックグラウンドで一括処理する
    - マスタデータをメモリに載せる
  - 優勝は「白金動物園」で、言語はRuby。スコアは解説記事・言語比率記事のどちらにも記載がない【要確認】 — [ISUCON9本選 解説](https://isucon.net/archives/53916974.html), [ISUCON9本選 言語比率](https://isucon.net/archives/53878163.html)
  - 本選の言語比率はGo 23チーム（71.9%）、Ruby 5チーム、PHP 2チーム、Python 1チーム、Perl 1チーム — [ISUCON9本選 言語比率](https://isucon.net/archives/53878163.html)

#### ISUCON10（2020）
- 開催概要: 予選は2020/9/12で490チーム、本選は2020/10/3で33チーム。予選の作問はリクルート、本選の作問はクックパッドと宇宙海賊合同会社 — [ISUCON10 まとめ](https://isucon.net/archives/54704557.html)
- **予選「ISUUMO」**: 椅子と物件の検索サイト — [ISUCON10予選 解説](https://isucon.net/archives/55025156.html)
  - ボトルネック1: ORDER BY 用のインデックスがない。`popularity DESC, id ASC` という混在ソートはMySQL 5.7ではインデックスが効かないため、符号を反転した生成列を作ってそこにインデックスを張る — 同上
  - ボトルネック2: なぞって検索（ポリゴン検索）のN+1。POINT型、`ST_Contains`、SPATIAL INDEXで1クエリにまとめる（アプリ側で内外判定したチームもあった） — 同上
  - ボトルネック3: bot（クローラ）のアクセス。User-Agentで判定して503を返す — 同上
  - chairとestateはJOINしないので、DBを2台に分ける「App×1/DB×2」構成が推奨されていた。運営の試走は8,000点台 — 同上
  - 予選通過チームの多くは、なぞって検索の早期解決とApp/DBの分割を正しく行っていた — 同上
- **本選「XSUCON」**: ISUCON自体を模したコンテストポータル。protobuf、gRPC、Envoyを使う — [ISUCON10本選 解説](https://isucon.net/archives/55140539.html)
  - 主なボトルネックは次のとおり — 同上
    - 静的ファイルをアプリが配信していた
    - 不変データを毎回クエリしていた
    - 69行のリーダーボードSQL
    - 通知ポーリング
    - ダッシュボードのキャッシュは1秒しか許容されない
    - 3台のスペックが異なり、1GBメモリの機種がある
  - 想定解法は次のとおり — 同上
    - 静的ファイルをnginxに移し、brotli/gzipとCache-Controlを付ける
    - リーダーボードをジョブ完了時に事前計算する
    - singleflightやVarnishで同時リクエストをまとめる（キャッシュスタンピード対策）
    - 個別通知をWeb Pushに移す
  - 作問者のRuby実装は36,768点だった。巨大リーダーボードSQLは「罠」で、ほかのボトルネックの方が重要だった — 同上
  - **結果**: 1位takonomura（学生・1人チーム、49,545点、Go）、2位Azeit（学生、43,366点）、3位がんもどき（学生、34,767点）で、学生が上位3位を独占した — [ISUCON10本選 結果](https://isucon.net/archives/55076348.html), [ISUCON10本選 言語比率](https://isucon.net/archives/55081269.html)
  - 本選の言語比率はGo 28/33チーム（84.8%） — [ISUCON10本選 言語比率](https://isucon.net/archives/55081269.html)

#### ISUCON11（2021）
- 開催概要: 予選は2021/8/21で598チーム、本選は2021/9/18で30チーム。作問はNTTコミュニケーションズ、ヤフー、traP、チームtakonomura。インフラはAWS — [ISUCON11 まとめ](https://isucon.net/archives/55821036.html)
- **予選「ISUCONDITION」**: 椅子のIoTコンディション監視サービス。CPUがボトルネックになる — [ISUCON11予選 解説](https://isucon.net/archives/56044867.html)
  - 想定解法は次のとおり — 同上
    - `isu_condition(jia_isu_uuid, timestamp)` に複合インデックスを張る（複合主キーならさらに良い）
    - POSTはbulk insertやキューでまとめて書き込む
    - condition_levelを事前に計算して保存し、LIMITを効かせる
    - trendは最新1件だけ取得してキャッシュする
    - ログ出力を削減する
    - `target_base_url` でPOST専用サーバに分ける
    - dropProbabilityを下げて加点を増やす
  - Go実装の目安は、前半の改善で15万点以上、trend改善まで含めて65万点以上。上位チームは「コンディション処理に集中」し、「複数台分散」を行っていた — 同上
  - 予選の総合順位は1位shallowverse、2位都営三田線東急目黒線直通急行日吉行、3位NaruseJun、4位fujiwara組（スコアは別記事）。HTTP/2 GOAWAYに関するベンチマーカーのバグがあり、最終スコアは2回計測した高い方を採用した — [ISUCON11予選 結果](https://isucon.net/archives/56021237.html)
- **本選「ISUCHOLAR」**: 大学の履修・成績管理システム。外部サービスはない — [ISUCON11本選 解説](https://isucon.net/archives/56163308.html)
  - ボトルネックは次のとおり — 同上
    - 成績確認でN+1、多重ループ、重いJOINが起きる（タイムアウトすると10秒sleep）
    - GPA一覧で同じクエリが繰り返される
    - 課題ダウンロードでshell経由のzip生成とdeflate圧縮が重い
    - お知らせ一覧のOFFSETページング
  - 10万点を超えるには3点が必須とされた: 複数台化、GPAクエリの共有（coalescing）、無圧縮zip。ほかに、終了講義の集計を永続キャッシュする、seekページングにする、外部キー制約を外す、binlogを無効化する、といった手がある — 同上
  - **結果**: 1位fujiwara組（135,523点、Go、通算4度目の優勝）、2位NaruseJun（119,984点）、3位FetchDecodeExecWrite（100,258点）。学生1位は雑談係（75,751点） — [ISUCON11本選 結果](https://isucon.net/archives/56098785.html), [ISUCON11本選 言語比率](https://isucon.net/archives/56100672.html), [ISUCON11 まとめ](https://isucon.net/archives/55821036.html)
  - 講評は、fujiwara組が確実な小改善と思い切った変更を組み合わせて作業を止めなかった点を評価している。本選の言語比率はGo 27/30チーム（90%） — [解説](https://isucon.net/archives/56163308.html), [言語比率](https://isucon.net/archives/56100672.html)

#### ISUCON12（2022）
- 開催概要: 予選は2022/7/23で698チーム、本選は2022/8/27で30チーム。運営はLINE、作問はカヤックとサイバーエージェント — [ISUCON12 まとめ](https://isucon.net/archives/56571716.html)
- **予選「ISUPORTS」**: マルチテナントSaaSのランキングアプリ。テナントごとにSQLiteファイルがあり、テナント管理用にMySQLが1つある。作問のテーマは「いろいろな解き方ができる問題」 — [ISUCON12予選 解説](https://isucon.net/archives/56850281.html)
  - ボトルネックは次のとおり — 同上
    - player_scoreが約370万行あるが、大半は冗長
    - visit_historyが約320万行
    - アプリでflockを取っている
    - auto_incrementテーブルでID採番している
    - 請求額を毎回再計算している
    - prefork系言語では初期の同時接続数が問題になる
  - 想定解法は3通り — 同上
    1. データを圧縮してMySQLへ移行する（/initializeは30秒以内という制約がある）
    2. SQLiteのまま、テナントを複数ホストに振り分ける（Redisとnginx Luaなどを使う）
    3. 主な共通施策: flockをトランザクションに置き換える、IDをUUID/ULIDにする、請求額を大会終了時に1回だけ計算する
  - 作問者のRuby・1台・SQLite構成で約24,000点。これは予選通過ラインと同程度だった。本選進出チームの言語はGo、Ruby、Node.js、Rust、Perlの5種類 — 同上
- **本選「ISU CONQUEST」**: 放置ゲーム（フロントはUnity WebGL）。ログイン処理（ログインボーナス・プレゼント配布）が主なボトルネックになるよう設計されていた — [ISUCON12本選 解説](https://isucon.net/archives/56959385.html)
  - 想定解法は次のとおり — 同上
    - 5台をApp×1、DB×4で構成し、DBを水平シャーディングする
    - プレゼントを未受取と履歴に分ける
    - マスタデータをオンメモリにキャッシュする（キャッシュキーにバージョンを含める）
    - Snowflake風にアプリ側でIDを採番する
  - 運営の想定優勝スコアは25〜30万点だったが、実際は約34万点だった — 同上
  - **結果**: 1位NaruseJun（341,258点、Go）、2位織時屋（学生、242,653点）、3位「と」（231,428点）、学生2位takonomura（219,830点） — [ISUCON12本選 結果](https://isucon.net/archives/56923294.html), [ISUCON12本選 言語比率](https://isucon.net/archives/56923304.html)
  - NaruseJunのメンバーは、とーふとふ（大橋）、sekai（澤田）、takashi（岸本）。担当はそれぞれ次のとおり — [gihyo NaruseJunインタビュー](https://gihyo.jp/article/2022/10/isucon12-champinterview-narusejun)
    - sekai: ユーザーごとのDB水平分割
    - takashi: N+1の解消とクエリ削減
    - とーふとふ: マスタデータのキャッシュ
  - NaruseJunの準備として、自作ツールpprotein（pprofと連携）による計測基盤、毎回更新するチートシート、事前練習1回が挙げられている — 同上
  - 本選の言語比率はGo 26/30チーム（86.7%） — [ISUCON12本選 言語比率](https://isucon.net/archives/56923304.html)

#### ISUCON13（2023、予選・本選の区別なし）
- 開催概要: 2023/11/25に開催し、694チーム（学生78チーム）が参加。運営はLINEヤフー、作問はさくらインターネット — [ISUCON13 まとめ](https://isucon.net/archives/57801192.html)
- **お題「ISUPipe」**: ライブ配信サイト。配信者ごとにサブドメインが割り当てられる。スコアはリクエスト数ではなく投げ銭（tip）の合計 — [ISUCON13 解説・講評](https://isucon.net/archives/58001272.html)
  - ボトルネックは次のとおり — 同上
    - インデックスがほぼない（追加で約3,000点→約10,000点）
    - アイコン画像のSHA-256をリクエストごとに計算している（icon_hashの事前計算と If-None-Match による304応答で解決）
    - fill*Response関数群のN+1
    - 統計処理
    - NGワード追加時の過去コメント遡及削除
  - **DNS（PowerDNS + MySQL）が初めて題材になった**。ランダムなサブドメインを問い合わせる「DNS水責め攻撃」でDBに負荷がかかる。対策はTTLを上げる、PowerDNSのキャッシュ、インデックスの復元、DNS用DBの分離、dnsdistによるフィルタ — 同上
  - 運営はAIによる分析・コード生成を許可していた。LLMを全面活用したチームのブログも紹介されている — 同上
  - **結果**: 1位NaruseJun（468,006点）、2位・学生1位は織時屋（243,852点、東工大traP）、3位ultra_fast_gopher（214,941点）、学生2位takonomura（167,438点） — [ISUCON13 結果](https://isucon.net/archives/57993937.html), [東工大ニュース](https://www.titech.ac.jp/news/2024/068310)
  - NaruseJunは2位にほぼダブルスコア（20万点以上の差）をつけた。メンバーのコメントに「普通の人が10時間かかる変更は、3時間で終えられる自信があるときだけやる」という趣旨がある — [Pocketsign Wantedly](https://www.wantedly.com/companies/Pocketsign/post_articles/928118)
  - 上位30チームの言語はGo 29、Ruby 1。NaruseJunの使用言語は公式記事に記載がない（前年はGo）【要確認】 — [ISUCON13 言語比率](https://isucon.net/archives/57995340.html)

#### ISUCON14（2024）
- 開催概要: 2024/12/8に開催し、834チーム（学生99チーム）、1,920人が参加。運営はLINEヤフー、作問はポケットサイン（NaruseJunのメンバーが担当） — [ISUCON14 まとめ](https://isucon.net/archives/58818382.html), [ISUCON14 解説・講評](https://isucon.net/archives/58869617.html)
- **お題「ISURIDE」**: 自動運転の椅子を使うライドシェアサービス。ベンチマーカーは30msを1tickとして動く。スコアは次の合計 — [ISUCON14 解説・講評](https://isucon.net/archives/58869617.html)
  - 迎車距離×0.1
  - 乗車距離
  - 完了ライド数×5
- ボトルネックと解法の流れ — 同上
  - 序盤
    - インデックスがほぼない（Go実装でインデックス中心に約4,000点）
    - nearby-chairsで2N+1回のクエリが発生する
    - owner/chairsの総走行距離集計（椅子ごとの最新位置と総走行距離を持つテーブルをUPSERTで管理する）
  - 中盤
    - マッチング: 初期実装は500msごとに1件、ランダムに割り当てる。これを一括マッチング、近い椅子の優先、遠すぎるペアの見送りに変える
    - 通知ポーリング（30ms間隔）を retry_after_ms で調整する。SSEへの移行は必須ではなく、JSON APIのままでも約10万点出る
    - coordinate を非同期化・バルク化する
  - その他
    - Idempotency-Key は作問者自身が「ブラフ」と認めている
    - 二部マッチングなどヒューリスティック的な発展も想定されていた
- **結果**: 1位takonomura（1人チーム・学生、58,153点、Go、ISUCON10に続き2度目の優勝）、2位第9西東京市（55,003点、Go）、3位「// TODO: あとでちゃんとしたチーム名になおす」（52,150点、Go）、学生2位team k（36,218点） — [ISUCON14 結果](https://isucon.net/archives/58837992.html), [ISUCON14 言語比率](https://isucon.net/archives/58847106.html)
- 講評の要点 — [ISUCON14 解説・講評](https://isucon.net/archives/58869617.html)
  - takonomuraは1人で着実に改善を積み重ねた
  - マッチングのアルゴリズムを変えず、頻度を上げるだけで上位に入ったチームもいた
  - ISUCON10のWeb Pushの経験から、SSE化を最初から捨てたチームもあった（作問者は妥当な判断と評価）
  - 多くのチームが生成AIでSQLやN+1を修正した
- 各言語の最高スコアは次のとおり（上位30チームはGo 26チーム） — [ISUCON14 言語比率](https://isucon.net/archives/58847106.html)
  - Go: takonomura（58,153点）
  - Rust: 焼肉ジャンボチキン（36,810点）
  - Ruby: 白金動物園（36,300点）
  - PHP: (´・ω・｀)（31,002点）
  - Node.js: 即興パフォーマンス2024（仮）（25,411点）
  - Python: 紙の保険証（13,033点）
- 4位チーム（都営三田線…、Go）の施策は次のとおり — [LayerX 参加記](https://zenn.dev/layerx/articles/3bd55b77e047c4)
  - インデックスの追加
  - ownerGetChairsの集計を総走行距離のDBキャッシュに変更
  - マッチングを短い間隔で複数件処理し、最近傍の椅子を選ぶ
  - 通知間隔を30msから1000msに調整
  - DBを別インスタンスに分離
  - interpolateParams=true
  - アクセストークンをオンメモリでキャッシュ
  - SQLに呼び出し元をコメントとして付け、スロークエリから追跡しやすくする自作ツール
- 失格例: チーム最上川は本番で37,127点（参考8位）だったが、データ保持の違反で失格になった。施策は、インデックス、DB分離、評価・椅子状態のオンメモリキャッシュ、ライド状態のN+1をまとめる、通知頻度の調整、マッチングの全件処理 — [kawaemon 参加記](https://zenn.dev/kawaemon/articles/f40529e0c447f5)

#### ISUCON15（2025）／ISUCON2026
- 公式の「まとめ」一覧は、ISUCON14の次がISUCON2026になっている。「ISUCON15」という回は見つからなかった。2025年に大会が開催された形跡は確認できなかった【要確認】 — [isucon.net トップ](https://isucon.net/), [ISUCON2026 まとめ](https://isucon.net/archives/60124520.html)
- ISUCON2026の概要 — [ISUCON2026 まとめ](https://isucon.net/archives/60124520.html), [スポンサー資料PDF](https://isucon.net/pdf/isucon2026_sponsors_information.pdf)
  - 2026/10/31（本日）に開催され、856チーム（学生83チーム）、1,596人が参加
  - 運営はさくらインターネット（LINEヤフーから運営主体が交代）
  - 本選はなく1日開催。大阪にオフライン会場もある
  - 結果・解説はまだ「準備中」
  - 問題リポジトリは github.com/isucon/isucon2026
- お題は「ISUReal」という写真共有サービス（参加者ブログ1件に基づく）【要確認】 — [いもす研 参加記](https://imoz.jp/scraps/202610_isucon.html)
  - アジア・欧州・米州の各リージョンでアプリが動き、初期構成ではアジアにあるPostgreSQLを共有しているため、リージョン間の遅延が主なボトルネック
  - チームAnago（imos、okuta、iwiwi）の取り組み
    - Claude/CodexなどAIコーディングエージェントを並列に動かした
    - Rustでオンメモリに再設計し、リージョンごとにデータを複製した
    - ブランチごとに約42万〜47万点を記録した
  - 最終順位は未公表

### Inferences
- ISUCON13以降は予選・本選の区別がない1日開催になり、ISUCON2026もこれを引き継いでいる。
- 優勝者の顔ぶれは、fujiwara組（4回）、NaruseJun（ISUCON12・13の連覇）、takonomura（ISUCON10・14）と固定化が進んでいる。NaruseJunはISUCON14の作問側に回った。
- 言語はGoが圧倒的で、本選・上位30チームの85〜97%を占める。ISUCON10以降の優勝チームは、言語が確認できた回はすべてGoだった。

### Gaps
- ISUCON9本選の優勝スコアと予選トップのスコアは、公式の全チームスコア記事を取得できず未確認。
- ISUCON9予選・ISUCON10予選・ISUCON11予選・ISUCON12予選のトップスコアの数値は未取得（各回の「全チームのスコア（参考値）」記事に掲載されているはず）。
- ISUCON13でNaruseJunが使った言語、およびISUCON13・14の優勝チームによる詳細な解法の一次情報（本人の参加記）は見つからなかった。
- ISUCON2026のお題・結果の公式情報は本日時点で未公開。「ISUReal」は参加者ブログ1件のみに基づく。

## 年を跨いで繰り返す勝ち筋（パターン）と近年の特殊要素

### Takeaway
ほぼ毎年出る定番は次のとおり。
- 初期実装のインデックス欠如
- N+1
- 不変データやマスタのオンメモリキャッシュ
- 静的ファイルのnginx配信やHTTPキャッシュ（304）
- 複数台構成への分割
- ログ削減・ミドルウェア設定

勝敗を分けるのは、その回固有のドメインロジックを正しく読み替えられるかどうか。例としては、外部API呼び出しの削減・非同期化、事前計算や非正規化、ロック粒度の調整、マッチングアルゴリズム、DNS、シャーディング、リージョン配置がある。これに加えて、短時間で着実に改善を積み重ねる計測駆動の進め方が効く。

### Cited Findings
- インデックスの欠如は毎回の初手として明記されている — [ISUCON14](https://isucon.net/archives/58869617.html), [ISUCON13](https://isucon.net/archives/58001272.html), [ISUCON11予選](https://isucon.net/archives/56044867.html), [ISUCON9本選](https://isucon.net/archives/53916974.html)
  - ISUCON9本選: 1つで約1,500点
  - ISUCON11予選: 複合インデックス・複合主キー
  - ISUCON13: 約3,000点→約10,000点
  - ISUCON14: Goで約4,000点
- N+1の解消（JOIN、IN句、プリロード）は全回に登場する — 各回の解説記事（上記）
- 「変わらないデータ／確定したデータはキャッシュ・事前計算する」も繰り返し出る — [ISUCON9予選](https://isucon.net/archives/53789931.html), [ISUCON10本選](https://isucon.net/archives/55140539.html), [ISUCON11本選](https://isucon.net/archives/56163308.html), [ISUCON12予選](https://isucon.net/archives/56850281.html), [ISUCON12本選](https://isucon.net/archives/56959385.html), [ISUCON14](https://isucon.net/archives/58869617.html)
  - ISUCON9: カテゴリをコードに持たせる
  - ISUCON10本選: リーダーボードの事前計算
  - ISUCON11本選: 終了講義の集計
  - ISUCON12予選: 請求額を終了時に1回だけ計算
  - ISUCON12本選: マスタデータ
  - ISUCON14: 総走行距離テーブル
- 書き込みのバルク化・非同期化・キュー化も繰り返し出る — [ISUCON11予選](https://isucon.net/archives/56044867.html), [ISUCON9本選](https://isucon.net/archives/53916974.html), [ISUCON14](https://isucon.net/archives/58869617.html)
  - ISUCON11予選: condition POST
  - ISUCON9本選: 決済キャンセルのバッチ処理
  - ISUCON14: coordinate
- 複数台構成の設計がスコアを決める回が多い — [ISUCON10予選](https://isucon.net/archives/55025156.html), [ISUCON11本選](https://isucon.net/archives/56163308.html), [ISUCON12本選](https://isucon.net/archives/56959385.html), [ISUCON13](https://isucon.net/archives/58001272.html)
  - ISUCON10予選: App×1/DB×2
  - ISUCON11本選: 複数台化が10万点超えの必須条件
  - ISUCON12本選: DBの水平シャーディング
  - ISUCON13: DNSとDNS用DBの分離
- ポーリングと通知の扱いも何度も題材になっている — [ISUCON10本選](https://isucon.net/archives/55140539.html), [ISUCON14](https://isucon.net/archives/58869617.html)
  - ISUCON10本選: Web Push
  - ISUCON14: retry_after_ms の調整、SSEは任意
  - ISUCON14では、ISUCON10の経験からSSE化を捨てる判断が妥当とされた
- HTTPキャッシュ（304 / Cache-Control / 静的ファイルのnginx配信）も繰り返し出る — [ISUCON10本選](https://isucon.net/archives/55140539.html), [ISUCON11予選](https://isucon.net/archives/56044867.html), [ISUCON13](https://isucon.net/archives/58001272.html)
- 「推測するな、計測せよ」と、計測→最大ボトルネックへの集中が上位の共通点とされている — [ISUCON11予選](https://isucon.net/archives/56044867.html), [ISUCON10本選](https://isucon.net/archives/55140539.html)
  - ISUCON10本選では、巨大SQLは罠でほかのボトルネックの方が重要だった
- スコアがリクエスト数ではなくビジネス指標で決まる回が増えている。ドメイン理解やマニュアルの読解で得点が伸びる仕掛けがある — [ISUCON9予選](https://isucon.net/archives/53789931.html), [ISUCON13](https://isucon.net/archives/58001272.html), [ISUCON14](https://isucon.net/archives/58869617.html)
  - ISUCON9: 売上、購入カテゴリの偏り
  - ISUCON13: 投げ銭合計
  - ISUCON14: 移動距離とライド数、「地域内でしか移動しない」という仕様の活用
- 近年の特殊要素 — [ISUCON9予選](https://isucon.net/archives/53789931.html), [ISUCON10本選](https://isucon.net/archives/55140539.html), [ISUCON12予選](https://isucon.net/archives/56850281.html), [ISUCON13](https://isucon.net/archives/58001272.html), [ISUCON14](https://isucon.net/archives/58869617.html), [いもす研 参加記](https://imoz.jp/scraps/202610_isucon.html)
  - ISUCON9: 外部API
  - ISUCON10本選: gRPC/protobuf/Envoy
  - ISUCON12予選: SQLiteマルチテナント
  - ISUCON13: DNS（PowerDNS）
  - ISUCON14: マッチングアルゴリズム（ヒューリスティック的な要素）
  - ISUCON2026: マルチリージョンとPostgreSQL【要確認】
- 生成AIの活用が進んでいる — [ISUCON13](https://isucon.net/archives/58001272.html), [ISUCON14](https://isucon.net/archives/58869617.html), [いもす研 参加記](https://imoz.jp/scraps/202610_isucon.html)
  - ISUCON13: 運営がAIの利用を許可した
  - ISUCON14: 多くのチームがSQLやN+1の修正に使った
  - ISUCON2026: 上位候補チームがAIエージェントを並列稼働させ、40万点台を記録した
- 上位チームは計測・デプロイの基盤を事前に整えている（pprotein、チートシート、SQLに呼び出し元を付けるツールなど） — [gihyo NaruseJunインタビュー](https://gihyo.jp/article/2022/10/isucon12-champinterview-narusejun), [LayerX 参加記](https://zenn.dev/layerx/articles/3bd55b77e047c4)
- 1人チームの優勝や学生上位独占が起きている — [ISUCON10本選](https://isucon.net/archives/55140539.html), [ISUCON14](https://isucon.net/archives/58869617.html)
  - ISUCON10: takonomuraが1人で優勝し、学生が上位3位を独占
  - ISUCON14: takonomuraが1人で優勝

### Inferences
- 練習で優先すべき順番は次のとおりと考えられる。
  1. 計測基盤とデプロイの自動化を事前に用意し、序盤30分〜1時間で完了させる
  2. インデックスとN+1の解消を素早くこなす
  3. 複数台構成への移行を手早く行う
  4. マニュアルを精読してスコア計算式とドメインの仕様の抜け道を探す
- 優勝チームに共通するのは「着実な積み上げ」と「時間内に終わる変更だけを選ぶ判断」。派手な再設計は、確実に終えられる場合に限られる。
- 近年はAIエージェントによる並列開発が上位の戦い方を変えつつある。来年に向けてはAI活用の練習が重要度を増すと推測される（ISUCON2026の結果公開後に再検証が必要）。
- 本番での失格はスコアを失う最大のリスク。データ保持の違反や再検証での画像不表示などが例で、整合性を確認しながら進める習慣が必要になる。

### Gaps
- 優勝チーム本人による詳細な時系列の解法記録（ISUCON13 NaruseJun、ISUCON14 takonomura）は今回の調査では見つからなかった。
- ISUCON2026の公式の解説と結果は未公開。公開後に追記が必要。
