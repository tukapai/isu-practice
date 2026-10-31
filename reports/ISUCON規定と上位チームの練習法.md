# 計測を自動化し、反復で優勝をつかむ

**結論**: 過去のISUCONで優勝したチームは、練習量の多さで勝っていたわけではありません。勝因は次の3つにまとめられます。

- 個人で過去問を深く解き込んでいた（1人あたり約10問、1問に数十時間かけることもある）
- チームで練習するのは1〜2回で、内容は「初動 → 改善 → 再起動試験」の手順確認に絞っていた
- 問題公開から数分で計測できる自作の資産（テンプレートリポジトリ、Ansible、pprotein）を、練習のたびに直していた

ルールについては、変化の大きい点を先に押さえてください。

- **形式**: 2022年のISUCON12を最後に予選・本選制は終わりました。ISUCON13（2023）からは「1日8時間・1〜3名・上限700〜1000チーム」の一発勝負です。
- **環境**: ISUCON11以降は、参加者が自分のAWSアカウントを用意し、東京リージョンにCloudFormationで構築します。ISUCON13・14ではc5.large×3台でした。
- **採点**: 最終スコアはベストではなく「最後に実行したベンチ」のスコアです。競技後には再起動を伴う追試があり、ISUCON14では834チーム中**約270チーム**がFAIL扱いになりました。
- **生成AI**: ISUCON13から3大会続けて、利用を明文で許可しています。

**時期に関する前提**: 今日（2026-10-31）は**ISUCON2026の本番当日で、結果と講評はまだ公開されていません**。また、公式アーカイブではISUCON14（2024年12月）の次がISUCON2026です。**「ISUCON15」という回は存在せず、2025年には大会が開催されていません**。なぜ番号を飛ばして西暦の名称にしたのかは、公式に説明されていません。

AIハーネスに関心がある読者向けの要点も先に書きます。2026年に非公式で開かれた「AI無制限」の演習では、上位チームが1000万〜2000万点台に達しました。差を生んだのはAIの実装速度ではありません。「インメモリ化に踏み切れるか」という**戦略判断**と、デプロイとベンチの**排他制御**でした。来年、優勝を狙う準備は、人間の型稽古とエージェントの基盤づくりを並行して進める形になります。

## 予選制は2022年で終わり、2023年からは1日勝負

### 形式の変遷

ISUCON12までは2段階でした。オンライン予選に約700チームが参加し、そこから**30チーム（一般25・学生5）**が本選に進みます。ISUCON12の予選は2022-07-23、本選は2022-08-27で、どちらも10:00〜18:00でした（[ISUCON12 予選レギュレーション](https://isucon.net/archives/56671734.html)、[ISUCON12 まとめ](https://isucon.net/archives/56571716.html)）。

ISUCON13は「今年は本選のみ開催！」と告知し、2023-11-25の1日だけで694チームが競いました（[ISUCON13 まとめ](https://isucon.net/archives/57801192.html)）。この形式はISUCON14（2024-12-08、834チーム、上限1000）とISUCON2026（2026-10-31、856チーム・学生83チーム、上限1000）にも引き継がれています（[ISUCON14 まとめ](https://isucon.net/archives/58818382.html)、[ISUCON2026 レギュレーション](https://isucon.net/archives/59966826.html)、[ISUCON2026 まとめ](https://isucon.net/archives/60124520.html)）。

チームは1〜3名で、一般枠と学生枠があります。学生枠の条件は「全員が学生であること」で、社会人学生や週5日フルタイムで働いている人は一般枠です（[ISUCON2026 レギュレーション](https://isucon.net/archives/59966826.html)）。

賞金はISUCON13・14とも同じ構成でした。総合優勝**100万円**、2位30万円、3位10万円、学生1位30万円、学生2位10万円です。これに「最初に○点に到達」した特別賞5万円が付きます（[ISUCON14 受賞チーム](https://isucon.net/archives/58837992.html)、[ISUCON13 受賞チーム](https://isucon.net/archives/57993937.html)）。ISUCON2026の賞金額は公式記事で確認できませんでした。ただしスポンサー資料に「優勝賞金」の項目があるので、賞金自体はあります（[スポンサー資料PDF](https://isucon.net/pdf/isucon2026_sponsors_information.pdf)）。

### 2026年の運営体制

ISUCON2026では、運営窓口がLINEヤフーからさくらインターネットに替わりました。オンラインとオフライン（大阪の「Blooming Camp」など）のハイブリッドで開催され、作問は「第n西東京市」が担当しました（[ISUCON2026 開催決定](https://isucon.net/archives/59917350.html)）。「第9西東京市」はISUCON14で総合2位になったチーム名です。

運営は替わりましたが、レギュレーション本文はISUCON14とほぼ同じです。ルールが大きく変わったわけではありません。

### 競技環境の変遷

インフラはISUCON5〜10まで毎年変わっていました。GCP・テコラス、Azure、さくら、GMO、Alibaba Cloud、サイバーエージェントの順です（[ISUCON9 まとめ](https://isucon.net/archives/53570241.html)、[ISUCON10 まとめ](https://isucon.net/archives/54704557.html)）。

ISUCON11でAWSになり、以降は参加者が自分のAWSアカウントで構築する方式が定着しました。ISUCON14の構成は、チーム専用のCloudFormationテンプレートで**c5.large×3台・gp3 20GB・EIP**をap-northeast-1に立てるものです。テンプレートを他チームと共有することは禁止されていました（[ISUCON14 マニュアル](https://github.com/isucon/isucon14/blob/main/docs/manual.md)）。

ISUCON13では、次の行為が失格条件として明記されていました（[ISUCON13 当日マニュアル](https://github.com/isucon/isucon13/blob/main/docs/cautionary_note.md)）。

- インスタンスタイプを変更すること
- 競技終了時に、複数スタックのサーバーが混在していること
- セキュリティグループの特定ポートを変更すること

ISUCON2026もCloudFormationと東京リージョンで構築する点は同じです。事前準備は必須で、10/27 19:00までにスタックを作成してSSH接続を確認する必要がありました。間に合わないチームは「当日サポート不可」です。確認後はスタックを削除しないと「高額な請求がかかる可能性があります」と注意されていました（[ISUCON2026 事前準備](https://isucon.net/archives/60111629.html)）。

ISUCON2026当日の台数とスペックは、当日マニュアルを取得できなかったため未確認です。クラウド費用は参加者の負担です（[ISUCON2026 参加規約](https://isucon.net/archives/59967038.html)）。普段のAWS運用と同じように、練習用スタックを放置しない習慣をつけておくと安心です。

### 参加登録

登録は先着順で、数日おきに3回に分けて受け付けます。

| 大会 | 受付日 |
|---|---|
| ISUCON2026 | 7/28 20:00に300枠、8/1 12:00に300枠、8/5 10:00に335枠 |
| ISUCON14 | 9/24、9/28、10/2 |

出典: [ISUCON2026 開催決定](https://isucon.net/archives/59917350.html)、[ISUCON14 まとめ](https://isucon.net/archives/58818382.html)

元運営者は「ISUCON9頃から約200枠が1分以内に埋まる」と話しています（[カケハシ 登壇資料](https://speakerdeck.com/kakehashi/why-do-people-love-isucon)）。スポンサー企業の「参加確定枠」を持つ人は優先して登録できます。

2027年の開催告知は、まだ出ていません。過去の実績からは、夏ごろに告知、本番の2〜3か月前に受付という流れが予想されます（推測）。

### 主な変化のまとめ

| 年・回 | 形式 | 環境 | 転換点 |
|---|---|---|---|
| 2020 ISUCON10 | オンライン予選・本選 | サイバーエージェント | 完全オンライン化 |
| 2021 ISUCON11 | 予選→本選30チーム | AWS協賛 | AWS方式へ移行 |
| 2022 ISUCON12 | 予選→本選30チーム | AWS（自前アカウント） | 予選制の最後、Javaを追加 |
| 2023 ISUCON13 | 本選のみ・1日 | AWS c5.large×3 | 生成AIの利用を明文で許可、DNSが題材に |
| 2024 ISUCON14 | 本選のみ・1日 | AWS c5.large×3 | 上限1000チーム、追試で全台を2回再起動 |
| 2025 | 開催なし（ISUCON15は存在しない） | — | — |
| 2026 ISUCON2026 | 本選のみ・ハイブリッド | AWS CloudFormation | さくらへ運営交代、名称を西暦化（結果は未公開） |

## 最後のベンチと追試で3割強が脱落する採点

### 最終スコアの決まり方

ISUCONで最も多い「事故」はスコアの付け方から生まれます。競技中にポータルからベンチを何度も実行しますが、**最終スコアは最後に実行したベンチのスコア**です。それがfailなら記録なしで、表彰の対象外になります（[ISUCON14 マニュアル](https://github.com/isucon/isucon14/blob/main/docs/manual.md)、[ISUCON13 レギュレーション](https://isucon.net/archives/57768216.html)）。

ISUCON14のベンチは、次の順に進みました。

1. 初期化（30秒）
2. 整合性チェック
3. 60秒の負荷走行
4. 最終チェック

クリティカルエラーが出るか、warningが200件を超えると中断されます。スコアは「迎車距離×0.1＋乗車距離＋完了ライド数×5」という、ビジネス指標に近い式でした（[ISUCON14 マニュアル](https://github.com/isucon/isucon14/blob/main/docs/manual.md)）。

### 追試と失格

競技が終わると運営が追試を行います。ISUCON14では3台すべてを**2回**再起動したうえで、次のような条件を確認しました（[ISUCON14 マニュアル](https://github.com/isucon/isucon14/blob/main/docs/manual.md)）。

- envcheckに合格すること
- 3台が同一VPCにあること
- 再起動後もデータを保持していること
- フロントエンドが参照実装と同じ構造で表示できること

ベンチがFAILになるか、最終スコアの**75%以下**だった場合は再実行され、3回ともNGなら失格です。

結果を見ると、この仕組みの厳しさがわかります。ISUCON14では最終スコア0のチームが117、追試NGのチームが153で、合計**約270チーム**がFAIL扱いでした。全834チームの3割強にあたります。追試NGの主な理由は「負荷走行NG」と「データ保持NG」で、ほかに画面が真っ白になる目視確認NGもありました（[ISUCON14 全チームスコア](https://isucon.net/archives/58837992.html)）。

上位チームでも例外ではありません。チーム最上川はISUCON14で37,127点を出し、参考順位は8位でしたが、データ保持違反で失格になりました（[kawaemon 参加記](https://zenn.dev/kawaemon/articles/f40529e0c447f5)）。

追試に関する共通ルールは、ISUCON12からISUCON2026まで変わっていません（[ISUCON2026 レギュレーション](https://isucon.net/archives/59966826.html)）。

- 再起動の順序は保証されない
- 競技中に近いスコアが再現されない場合や、捏造があると判断された場合は失格
- ベンチ中に書き込んだデータを、再起動後も取得できなければならない

ISUCON2026のスコア式とfail条件は当日マニュアルが未入手のため未確認です。ただ、75%基準はISUCON13・14で共通なので、2026年も踏襲されている可能性が高いと考えます。

### 実務上の含意

ここから導かれる教訓ははっきりしています。

- オンメモリキャッシュを使うなら、再起動後に復元できることを保証する
- 全台の再起動試験を、競技中に必ず行う
- 最後のベンチは、成功が確実な状態で実行する

これはスコアを上げる作業と同じくらい重要です。オンメモリキャッシュの扱いは、ふだんのインフラ設計で言えば、ステートフルなキャッシュをどう永続化するかの問題に相当します。

## 変更はほぼ自由。ただし外部委譲は禁止で、生成AIは明文で許可

### 変えてよいもの・変えてはいけないもの

変更できる範囲は非常に広いです（[ISUCON2026 レギュレーション](https://isucon.net/archives/59966826.html)）。

- **変更してはいけないもの**: アクセス先のURI、レスポンス構造、JS/CSSの内容、メディアファイルの内容
- **変更してよいもの**: サーバーの役割分担、DBのスキーマとインデックス、DBミドルウェアの入れ替え、キャッシュやジョブキューによる遅延書き込み、他言語での再実装

当日マニュアルはレギュレーションより優先されます。

### 言語の選択

参照実装の言語は、Go、Perl、PHP、Python、Ruby、Rust、Node.jsです（ISUCON12はJavaもありました）。運営は、言語ごとに性能が同等であることは保証していません（[ISUCON2026 レギュレーション](https://isucon.net/archives/59966826.html)）。

実際の分布はGoに大きく偏っています。ISUCON14では787チーム中549チームがGoで、上位30チームのうち**26チーム**がGoでした（[ISUCON14 言語比率](https://isucon.net/archives/58847106.html)）。言語が確認できた範囲では、ISUCON10以降の優勝チームはすべてGoを使っています。

### 禁止事項と連絡手段

禁止事項は大きく3つです（[ISUCON2026 レギュレーション](https://isucon.net/archives/59966826.html)）。

1. ベンチ中に、認められたサーバー以外へ処理を委譲すること。ただし、モニタリング・テスト・開発に外部のPCやサーバーを使うのは許可されています。
2. 競技中にチーム外の人と問題について話すこと（SNSを含む）。
3. 他チームと結託すること。

連絡手段も決まっています。全体アナウンスはDiscord、競技中の質問はポータルで行います。全員がDiscordにログインし、通知を設定しておく義務があります。

### 生成AIの扱い

生成AIについては、ISUCON13から次の文言が入っています。

> 開発において、AIを活用したコードの分析、生成等を利用しても構わない。

ISUCON14とISUCON2026も同じ文言で、3大会連続です（[ISUCON13 レギュレーション](https://isucon.net/archives/57768216.html)、[ISUCON14 レギュレーション](https://isucon.net/archives/58657116.html)、[ISUCON2026 レギュレーション](https://isucon.net/archives/59966826.html)）。ISUCON12の予選レギュレーションにはAIの記述がなく、ISUCON2026の参加規約にもAI固有の条項はありません（[ISUCON2026 参加規約](https://isucon.net/archives/59967038.html)）。

規定のない「グレーゾーン」も2つあります。

- **外部LLMへのコード送信**: 外部のLLMにコードやログを送ることが「チーム外の人と問題について話すこと」にあたるかは、明文化されていません。条文は人間を想定していると読むのが自然です。
- **実行時のLLM呼び出し**: アプリの実行時処理として外部のLLM APIを呼ぶことは、スコア向上に寄与する外部委譲にあたり、禁止と解釈すべきです。

AIエージェントを本番で使う予定なら、その年のレギュレーションと当日マニュアルを確認し、迷ったらポータルで質問するのが確実です。

## 優勝チームは「個人で10問、チーム練習1〜2回」だった

### 歴代優勝チーム

ここ数年の優勝者はほぼ決まった顔ぶれです。

| 大会 | 優勝チーム | スコア | 備考 |
|---|---|---|---|
| ISUCON10 | takonomura | 49,545点 | 学生・1人チーム |
| ISUCON11 | fujiwara組 | 135,523点 | 通算4度目の優勝 |
| ISUCON12 | NaruseJun | 341,258点 | |
| ISUCON13 | NaruseJun | 468,006点 | 2位にほぼダブルスコア |
| ISUCON14 | takonomura | 58,153点 | 1人チームで2度目の優勝 |

出典: [ISUCON10本選 結果](https://isucon.net/archives/55076348.html)、[ISUCON11本選 結果](https://isucon.net/archives/56098785.html)、[ISUCON12本選 結果](https://isucon.net/archives/56923294.html)、[ISUCON13 結果](https://isucon.net/archives/57993937.html)、[ISUCON14 結果](https://isucon.net/archives/58837992.html)

このうち練習の中身まで一次情報が残っているのは、主にNaruseJunです。

### 練習量: チーム練習は少なく、個人の過去問は深く

ISUCON12のときのNaruseJunは、次のように進めていました。

- 7/9: 約1時間の方針会議
- 7/17: ISUCON11予選で約5時間の通し練習（100万点到達）
- 8/21: 本選前のチーム練習はこの1回だけ（ISUCON11本選を約4.5時間）

一方で、メンバーのとーふとふ氏は個人で過去問を「1問数十時間」かけてチューニングしていました（[NaruseJun ISUCON12参加記](https://zenn.dev/tohutohu/articles/8c34d1187e1b21)）。本番前の約2週間は毎晩予習し、Goを書く「Go筋力」を鍛えたと語っています（[@IT](https://atmarkit.itmedia.co.jp/ait/articles/2209/29/news014.html)）。

ISUCON13ではさらに絞り込んでいます。11/14に約30分のキックオフ、11/18に通し練習を1回やっただけです。その代わり、各メンバーが過去問を**約10問ずつ**（重複あり）解いていました。private-isuは毎年50万点を目標に約10時間かけて解いており、「何回解いても学びがあるので本当にいい問題」と書いています（[NaruseJun ISUCON13参加記](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)）。

takonomuraは、ISUCON8に出た年は「毎週のように過去問の練習」をしていたと書いています。ISUCON9直前の3日間は、スコアを伸ばすことよりツールの使い方の確認を優先しました（[takono.io](https://www.takono.io/posts/2019/09/isucon/)）。

この構造をまとめると、次のようになります。

- **地力**: 個人の反復で身につける。同じ問題を何度も解くことも推奨されています。
- **チーム練習**: 手順の検証とツールのデバッグに使う。

対照的に、下位・失格チームの振り返りには「1か月前には始めるべき」「前日だけでは圧倒的準備不足」という言葉が繰り返し出てきます（[kawaemon](https://zenn.dev/kawaemon/articles/f40529e0c447f5)、[irsc](https://zenn.dev/irsc/articles/isucon14-diary)）。

### 練習環境: 3台構成を再現する

練習環境の定番は2つあります。

- **matsuu/aws-isucon**: ISUCON5〜14の過去問とベンチマーカーを同梱した公開AMIです（[GitHub](https://github.com/matsuu/aws-isucon)）。
- **catatsuy/private-isu**: 入門の定番問題です。

ただし、1台にベンチを当てるだけの練習では、複数台への分割やDB分離の手順は身につきません。過去回のCloudFormationテンプレートのAMIをmatsuu版に差し替えれば、本番と同じ3台構成を再現できます（[sadnessOjisan LT](https://speakerdeck.com/player/298ef05ffaf84a20ac31da0bea2e3018)）。NaruseJunもこの方法を取り、private-isu・ISUCON11予選・ISUCON9予選のCloudFormationテンプレートを用意しました。ベンチを起動するための簡易ポータルまで自作しています（[NaruseJun ISUCON13参加記](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)）。

コミュニティの練習の場もあります。

- **traP**（NaruseJunの母体サークル）: 過去問を使った社内ISUCON「PISCON」を開いています。ポータルを約3日開放し、44人・12チームが参加しました（[ikura-hamu LT](https://speakerdeck.com/ikura_hamu/trap-nobu-nei-isucon-tosorewozhi-erupotaru)）。
- **ISUCON精進鯖**: 非公式のDiscordで、毎週火曜に定例会を開いています（[HackMD](https://hackmd.io/@to-hutohu/isucon-shojin-server)）。

### 役割分担: 固定するのは初動の30〜60分だけ

ISUCON12のNaruseJunは、初動を次の3人で分けていました。

- 1人: ドキュメントとコードを読み、エンドポイントとテーブルの読み書きをスプレッドシートに整理する
- 1人: Ansibleとpproteinの設定
- 1人: コードをリポジトリに入れ、デプロイスクリプトを整える

それ以降は担当を固定しません。各自がブランチで作業し、デプロイとベンチは口頭で「予約」してから使っていました（[NaruseJun ISUCON12参加記](https://zenn.dev/tohutohu/articles/8c34d1187e1b21)）。ISUCON13でも同じやり方で、本番中は各自が「計測→分析→改善内容を宣言」し、他の人と被ったら次点の改善に移るというルールでした（[ISUCON13参加記](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)）。

fujiwara組も、役割はあるが固定はしない「十種競技」型です。手が空いている時間を無駄とみなし、誰が何をしているかを可視化していました（[カヤック](https://www.kayac.com/news/2021/12/interview_ISUCON)）。ISUCON14で4位だった「都営三田線…」は、インフラ担当が鍵の配置とログ設定を「一瞬で」終え、その間に残り2人がマニュアルを読む分担でした（[LayerX 参加記](https://zenn.dev/layerx/articles/3bd55b77e047c4)）。

### 事前準備資産と振り返り

NaruseJunのチートシートは、「初動」「改善」「ブラウザチェック確認」の3つに分かれていました。毎回見直して更新し、問題公開からほぼすぐに作業に入れる状態にしていました（[gihyo.jp インタビュー](https://gihyo.jp/article/2022/10/isucon12-champinterview-narusejun)）。

テンプレートリポジトリには次のものが入っていました。

- アプリのコード、env.sh、ミドルウェアの設定（設定の変更もリポジトリ経由で行う）
- デプロイ用のMakefile: `make deploy BRANCH=<名前>` で任意のブランチをデプロイする
- pproteinの計測開始: `/initialize` の末尾で自動的に始める

さらに、pproteinで取ったプロファイルを使ってGoのPGOビルドまで行っていました（[ISUCON13参加記](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)）。

1人で参加したtakonomuraも、ISUCON9のときに次の作業をスクリプト化していました（[takono.io](https://www.takono.io/posts/2019/09/isucon/)）。

- ソースのGitHub化
- ツールの一括導入
- ビルド・再起動・ログローテート
- alpの結果をGitHubに投稿する

振り返りの方法にも特徴があります。KPTのような形式的なふりかえりではなく、練習で見つかった問題を**テンプレートリポジトリとツールにすぐ戻す**、実務的なやり方です（[ISUCON13参加記](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)）。上位チームが定型の振り返りフォーマットを使っているという記述は見つかりませんでした。

## 計測を自動化したチームが、後半の「固有ボトルネック」に時間を使える

### 毎年出る典型と、勝負を決める固有の問題

過去問の解説を並べると、毎年ほぼ同じ形が見えます。序盤に毎回出るのは次のような改善です。

- **インデックスの欠如**: ISUCON13では追加だけで約3,000点から約10,000点に、ISUCON14ではGoで約4,000点に伸びました。
- **N+1の解消**
- **マスタデータのオンメモリ化**
- **静的ファイルのnginx配信と304応答**

出典: [ISUCON13 解説](https://isucon.net/archives/58001272.html)、[ISUCON14 解説](https://isucon.net/archives/58869617.html)

勝負を決めるのは、その回にしかない固有の処理です。

| 回 | 固有の処理と対策 |
|---|---|
| ISUCON9予選 | 外部APIを呼ぶ間のロック保持 |
| ISUCON10予選 | 空間検索 |
| ISUCON11本選 | 複数台化・GPAクエリの共有・無圧縮zipが10万点超えの必須条件 |
| ISUCON12本選 | DBの水平シャーディング |
| ISUCON13 | PowerDNSへの「水責め」攻撃 |
| ISUCON14 | マッチングアルゴリズムと通知間隔 |

出典: [ISUCON9予選 解説](https://isucon.net/archives/53789931.html)、[ISUCON11本選 解説](https://isucon.net/archives/56163308.html)、[ISUCON12本選 解説](https://isucon.net/archives/56959385.html)

ISUCON2026のお題は、参加者のブログ1件にしか情報がありません。それによると、写真共有サービス「ISUReal」で、マルチリージョンとPostgreSQLを使う構成だったとされています（[いもす研 参加記](https://imoz.jp/scraps/202610_isucon.html)）。公式には未確認です。

### 計測ツールと配線

計測の定番は3つの組み合わせです。

- nginxのLTSVログ → alp
- MySQLのslow log（`long_query_time=0`）→ pt-query-digest / slp
- Goのpprof

近年はこれらを1つの画面にまとめる**kaz/pprotein**が標準になっています（[pprotein](https://github.com/kaz/pprotein)）。要点は配線です。`POST /initialize` のハンドラからpproteinのcollect APIを叩けば、ベンチの開始と同時に計測が始まります（[pprotein getting started](https://hackmd.io/@to-hutohu/pprotein-getting-started)）。alpの `--dump` と `alp diff` を使うと、改善の前後をテキストで比較できます（[alp](https://github.com/tkuchiki/alp)）。

ISUCON13でのNaruseJunの分析は、次の順で進みました（[ISUCON13参加記](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)）。

1. alpをsumでソートして、重いエンドポイントを見る
2. slpでSQLを見る。Rows Examined / Rows Sentが100倍程度ならインデックス不足を疑う
3. DBの負荷が下がってから、pprofのフレームグラフを見る

この結果、11:00の時点で5,261点に到達しています。

### 最後の1時間の使い方

最後の1時間の使い方は、優勝チームでほぼ決まっています。ISUCON13のNaruseJunは17:00から改善を止め、次の順に確認しました（[ISUCON13参加記](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)）。

1. ベンチ
2. フロントエンドの確認
3. 再起動
4. systemctl statusで全サービスの状態確認
5. フロントエンドの確認
6. ベンチ（以降、ベンチとフロントエンド確認を繰り返す）

最後にアクセスログとpprofを止め、効果のなかった設定を元に戻しています。ISUCON8で優勝した「最大の敵は時差」も、1変更ごとにベンチを3〜4回回し、最後の1時間は何も変更しませんでした（[ThinkIT](https://thinkit.co.jp/article/15737)）。

ISUCON13でNaruseJunのメンバーは「普通の人が10時間かかる変更は、3時間で終えられる自信があるときだけやる」という趣旨のことを話しています（[Pocketsign Wantedly](https://www.wantedly.com/companies/Pocketsign/post_articles/928118)）。大胆な再設計は、時間内に確実に終えられるときだけ選ぶ、という判断基準です。

## AIエージェントの時代も、差を生むのは戦略判断と排他制御

### AI活用の広がり

AIの使い方は、2年でまったく別のものになりました。

- **ISUCON14（2024）**: 公式講評が「生成AIを利用しているチームも数多く」あったと書き、Copilot・Cursorで「SQLクエリやアプリケーションコードの改善」をしていたと指摘しています（[ISUCON14 解説・講評](https://isucon.net/archives/58869617.html)）。百万円ドリブンのnakario氏は、Copilot Editが長いファイルを全部書き換えて約54秒かかる問題に直面しました。関数単位にファイルを分割するツール「fsplit」を自作して約5秒に短縮しましたが、結果は振るいませんでした（[nakario Blog](https://nakario.hateblo.jp/entry/2024/12/10/025218)）。
- **ISUCON2026（本日）**: いもす研の参加記によると、チームAnago（imos、okuta、iwiwi）はClaudeやCodexを並列で動かし、Rustでオンメモリ構成に作り直して、ブランチごとに約42万〜47万点を記録しています（[いもす研 参加記](https://imoz.jp/scraps/202610_isucon.html)）。最終順位は未公表です。

### AI無制限の演習「ISUNARABE合同演習2026」

最も示唆に富むのは、2026-05-09に開かれた非公式の「ISUNARABE合同演習2026」です。ルールはAI無制限、参照実装はRustのみでした（[Write and Run](https://diary.hatenablog.jp/entry/2026/04/13/110000)）。

- **ichyo氏**: Codexだけを使いました。39,100点からメモリ常駐のRust APIに作り直して17,464,000点、通知のキュー化とペーシングで**2,100万点台**まで伸ばしています（[ichyo::blog](https://blog.ichyo.jp/posts/isunarabe2026)）。
- **rail44氏のチーム**: 3台にシャーディングした結果、ロック競合で最終0点になりました。rail44氏は「上位は単一サーバーのインメモリ化で約1500万〜2000万点」と報告しています（[rail44](https://zenn.dev/rail44/articles/932a22e70ef98d)）。
- **overcached!**: Claude Codeだけを使ったチームです。200万点前後で伸びが止まり、人間がログ分析と方針決定を担う分担に切り替えて最終5,176,800点でした（[edge2992](https://tech.edge2992.dev/posts/20260510_isucon_nrb2026/)）。

AIの実装速度はどのチームもほぼ同じでした。それでも点数が桁違いに分かれたのは、**アーキテクチャを刷新する判断ができたかどうか**の差です。

### AIエージェントの失敗例

各チームの報告には、AIエージェントの失敗例も具体的に書かれています。

- 「Claudeの提案はこの種のコンテストには慎重・漸進的すぎる」「後方互換性や保守性は不要と何度も言う必要があった」（[rail44](https://zenn.dev/rail44/articles/932a22e70ef98d)）
- DBロックをアプリ側のmutexに置き換える改修は、単一サーバーなら効くが、複数台では壊れる。こうしたISUCON特有のトレードオフ判断にAIは弱い（[edge2992](https://tech.edge2992.dev/posts/20260510_isucon_nrb2026/)）
- 複数のエージェントが、互いのベンチ中にデプロイしてしまう事故（同上）
- ベンチの開始が2時間遅れたことに、エージェントが気づかなかった（[rail44](https://zenn.dev/rail44/articles/932a22e70ef98d)）

### 基盤の作り方

これらへの対策として、ichyo氏は次のような基盤を作りました（[ichyo::blog](https://blog.ichyo.jp/posts/isunarabe2026)）。

- スコアに影響する設定をすべてリポジトリに入れる
- DBスナップショットから毎回同じ状態に戻せるようにする
- AIにはサーバーを直接触らせず、「tar.gzのartifactを作る → 専用agentでdeploy → bench → PRを作る」というループにする
- デプロイの衝突を防ぐ管理サービス「isudeck」を自作し、運用ルールはAGENTS.mdに書く

作問者のKOBA789氏も、自身の振り返りで次のように書いています（[nrb2026 POSTMORTEM](https://github.com/KOBA789/nrb2026/blob/main/POSTMORTEM.md)）。

- 中核の設計判断は人間が行い、形式化や細部はAIに任せる
- 記憶はバージョン管理されたdocsに置く
- worktree・subagent・クロスモデルレビューといった軽い仕組みのほうが、試行錯誤には向いている

### 自律エージェントの到達点と注意点

完全自律の事例もあります。Mackerelのブログでは、Claude CodeとMackerel MCP Serverを使い、ISUCON9予選を人手の介入なしで**1,810点から50,280点**まで伸ばしました。外部APIを呼ぶ間のロック保持を見つけ、トランザクションの外に移しています（[Mackerel Blog](https://mackerel.io/ja/blog/entry/tech/ai-isucon)）。

ISUCON以外のハーネス設計例も参考になります。KarpathyのautoresearchにならってISUCON外のRailsエンドポイントを改善した例で、要素は次のとおりです（[dely Zenn](https://zenn.dev/dely_jp/articles/3117e590465e38)）。

- 1サイクル9分の予算
- 結果をTSVに記録する
- 改善したときだけkeepし、それ以外はrevertする
- ベンチとシードデータをエージェントから見えないようにする
- 使えるコマンドをホワイトリストで制限する

計測を常に有効にしておくとスコアが落ちる点にも注意が必要です。Splunkの検証では、計装を常時ONにしたことで56,925点から50,412点に下がりました。最終計測の前には計装を外すべきです（[Splunk Blog](https://www.splunk.com/ja_jp/blog/observability/isucon-rethink-observability-in-the-age-of-ai.html)）。

## 結論

これまでの記録が示すのは、「AIで速く書けること」はもう差別化要因ではないということです。差を生むのは、上位チームが人間の時代から磨いてきた3つの要素です。

- **再現可能なデプロイと計測**
- **衝突しない作業の割り振り**
- **時間内に終わる大胆な変更だけを選ぶ判断**

NaruseJunがやっていたことは、口頭でのデプロイ予約、initializeに仕込んだ計測、練習のたびのテンプレート改修でした。これは、ichyo氏のisudeckやAGENTS.mdとほぼ同じ構造です。つまりAIハーネスは、優勝チームの運用プロトコルを機械に読める形に書き起こしたものと言えます。

過去問を自律エージェントで解くと5万点ほどで頭打ちになり、AI無制限の上位は2000万点に届いています。この差は、インメモリ化や単一サーバーへの集約、仕様で許される遅延の利用といった「戦略層」をまだ自動化できていないことを示しています。ここが、来年に向けた最大の研究テーマになります。

AWSインフラの経験は、2つの場面で直接の強みになります。1つは、CloudFormationで3台構成の練習環境を量産し、スナップショットから毎回復元できる基盤を作ること。もう1つは、追試での再起動やデータ保持を設計段階で保証することです。

学習を始める順序は次のとおりです。

1. 3台構成の過去問環境と、計測・デプロイのテンプレートを自作する
2. 自分の手で過去問を約10問解き、型を体に入れる。その中で見つけたISUCON特有の判断基準を、CLAUDE.md/AGENTS.mdに言語化していく
3. 排他制御と履歴記録を備えたエージェントのループを、過去問で回して比較する

ISUCON2026の結果と講評が公開されれば、AIを本格的に使った上位チームの実像が初めて公式に見えます。公開されしだい、この結論を検証し直すべきです。
