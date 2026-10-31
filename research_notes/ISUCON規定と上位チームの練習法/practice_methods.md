# ISUCON上位・優勝チームの事前準備と練習法（ISUCON10〜14中心）

調査日: 2026-10-31。一次情報（優勝チーム本人の参加記・公式ブログ・インタビュー記事）を優先。下位〜中位チームの記事は「対照例」として明示して扱う。

前提情報（歴代優勝）:
- ISUCON10 (2020): takonomura（1人）、ISUCON11 (2021): fujiwara組（fujiwara, acidlemon, macopy）、ISUCON12 (2022) / ISUCON13 (2023): NaruseJun（とーふとふ, sekai, takashi）、ISUCON14 (2024): takonomura（1人） — [『ISUCON』歴代優勝チーム一覧（note, 二次情報）](https://note.com/asusn_online/n/n1eddf234c9f7)
- ISUCON14（2024-12-08開催）上位: 1位 takonomura 58,153点 / 2位 第9西東京市 55,003 / 3位 「// TODO: あとでちゃんとしたチーム名になおす」52,150 / 4位 都営三田線東急目黒線直通急行日吉行 48,615 / 5位 ぱかぱかアルパカ 43,888 — [ISUCON公式Blog: ISUCON14 受賞チームおよび全チームスコア](https://isucon.net/archives/58837992.html)
- 2025年は「ISUCON15」名義の開催記事を公式ブログ上で確認できず。公式トップでは2026-10-31付で「ISUCON2026 まとめ」「ISUCON2026 関連エントリーまとめ」が掲載され、受賞チーム発表は「準備中」 — [isucon.net](https://isucon.net/)（**要確認**: 2025年開催の有無と、ISUCON2026が実質的な第15回かどうかは未確認）

## 練習の頻度・量（素振り、過去問をN問、模擬コンテスト）

### Takeaway
優勝チームNaruseJunは「チーム全体での通し練習は本番前に1〜2回」と少ないが、個人で過去問を約10問ずつ・1問数十時間かけて解き込み、本番2週間前は毎晩予習していた。チーム練習は「初動→改善→再起動試験」の手順確認に絞り、地力は個人の過去問精進で積む、という分業構造が見える。

### Cited Findings
- NaruseJun（ISUCON12優勝）: 7/9に約1時間の方針会議、7/17にISUCON11予選で約5時間の練習（「初動→改善→再起動試験」、100万点到達）。本選前のチーム練習は8/21の1回のみ（ISUCON11本選を約4.5時間、11.5万点） — [ISUCON12で優勝しました(チーム NaruseJun)](https://zenn.dev/tohutohu/articles/8c34d1187e1b21)
- 同記事: とーふとふ氏は個人で過去問を「1問数十時間」かけてチューニング。ISUCON12予選・ISUCON11予選を「典型的なムーブ」の練習用に推奨し、「すでに解いたことがある問題でも問題ない」と述べる — [同上](https://zenn.dev/tohutohu/articles/8c34d1187e1b21)
- NaruseJun（ISUCON12）: とーふとふ氏は本番前2週間ほど「毎晩予習」。実際の競技でGoを書く「Go筋力」を養った — [@IT: 悲願のISUCON初優勝を果たした「NaruseJun」](https://atmarkit.itmedia.co.jp/ait/articles/2209/29/news014.html)
- NaruseJun（ISUCON13優勝）: 11/14に約30分のキックオフで流れ確認と練習日決定、11/18にsekai氏のオフィスで14:00〜19:30の通し練習1回（初動→改善いくつか→再起動試験の手順確認）。各メンバーは被りありで過去問を約10問ずつ解いた。private-isuは今年も50万点を目指して約10時間かけ、「何回解いても学びがあるので本当にいい問題」 — [ISUCON13で優勝しました(チーム NaruseJun)](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)
- NaruseJun（ISUCON12）: 本選前の練習は対面で1回、チートシートを使った素振り — [gihyo.jp: ISUCON12優勝チームNaruseJun特別インタビュー](https://gihyo.jp/article/2022/10/isucon12-champinterview-narusejun)
- takonomura（ISUCON9予選1位、のちISUCON10・14優勝）: ISUCON8に出た年は「毎週のように過去問の練習をしていました」。ISUCON9直前は木〜土の約3日のみで、スコア最大化よりツールの使い方の確認を重視 — [ISUCON9 予選を全体1位で突破しました（takono.io）](https://www.takono.io/posts/2019/09/isucon/)
- fujiwara組（ISUCON11優勝）: 公式な練習計画の記述はなし。本番前に業務（Tonamelのライブ大会）でDB過負荷の障害を当日原因特定→翌朝修正→昼リリースした経験を「（ISUCONを）実践した感じ」と表現 — [カヤック: なぜカヤックは『ISUCON』で何度も優勝できるのか](https://www.kayac.com/news/2021/12/interview_ISUCON)
- ISUCON精進鯖（非公式Discord、とーふとふ氏関与）: 毎週火曜20:00の定例会、作業通話、#sodan、times チャンネル。過去問を「じっくり」解くことを推奨 — [ISUCON精進鯖（HackMD）](https://hackmd.io/@to-hutohu/isucon-shojin-server)
- traP（東京科学大学のサークル、NaruseJunの母体）: 過去問（ISUCONDITION=ISUCON11予選、ISUPORTS=ISUCON12予選）でサークル内ISUCON「PISCON」を開催。ポータルを約3日開放し、持ち時間は最大8時間、事後に解法共有会。44人12チームが参加 — [traP内ISUCONとそれを支えるポータル（ikura-hamu, 2025-01-10 ISUCON14振り返りLT）](https://speakerdeck.com/ikura_hamu/trap-nobu-nei-isucon-tosorewozhi-erupotaru)；サークル内ISUCONと勉強会の存在は [@IT](https://atmarkit.itmedia.co.jp/ait/articles/2209/29/news014.html) でも言及
- 対照例（中位）: 初参加3人チームは9月下旬から「2週に1回」集まり、ISUCON12過去問で練習。結果は約10,983点・約220位（順位は本人も「おそらく」） — [tockey（Zenn）](https://zenn.dev/tockey/articles/0d55e1b9109615)
- 対照例: ISUCON14で事前準備ゼロ（前日夜にvagrant-isuconで朝6時まで練習）のチーム最上川は、データ保持違反で失格（失格前8位）。「一ヶ月ぐらい前にはやっとくべき」 — [kawaemon（Zenn）](https://zenn.dev/kawaemon/articles/f40529e0c447f5)
- 対照例: ISUCON14を1人・準備不足で629/831位。前日のみISUNARABEを使い「圧倒的準備不足」と振り返り — [irsc（Zenn）](https://zenn.dev/irsc/articles/isucon14-diary)

### Inferences
- 優勝チームは「チームでの通し練習回数」より「個人の過去問解き込み量（1人10問前後、1問に長時間）」と「チーム手順の固定化（初動・デプロイ・再起動試験）」で差をつけている。チーム練習は手順の検証とツールのデバッグの場。
- 「同じ問題を何度も解く」ことが推奨されている（private-isuを毎年、ISUCON11/12予選を型の練習に）。新問を広く浅くより、典型問題の反復で初動速度と典型改善を体に入れる考え方。
- 開始時期は優勝チームでは予選の約2週間〜1か月前に集中練習（NaruseJunのISUCON12は7月上旬に始動し、予選は7/23）。ただし個人の素振りは年間を通じて行っている（takonomura「毎週」）。

### Gaps
- takonomura（ISUCON10/14優勝）のISUCON10・14準備記事を一次ソースで確認できなかった（ISUCON9予選の記事のみ）。
- ISUCON14の2位「第9西東京市」・3位チームの準備記事は見つからなかった。
- 2025年（ISUCON15相当）およびISUCON2026の優勝チームと準備法は未確認。

## 練習環境（private-isu、過去問AMI、ベンチマーカー、コミュニティツール）

### Takeaway
定番は matsuu/aws-isucon の公開AMI（ISUCON5〜14の過去問とベンチマーカー同梱）と catatsuy/private-isu。優勝チームNaruseJunはさらに本番同等構成（複数台・ネットワーク）を再現するCloudFormationテンプレートやベンチ起動用の簡易ポータルを自作していた。

### Cited Findings
- matsuu/aws-isucon: ISUCON5〜12予選、ISUCON9〜12本選、ISUCON13・14の環境を東京リージョンの公開AMI（webapp＋ベンチマーカー同梱）とPackerテンプレートで提供。CloudFormationは同梱せず — [matsuu/aws-isucon（GitHub）](https://github.com/matsuu/aws-isucon)
- 複数台練習は「過去回のCloudFormationテンプレートのAMIをmatsuu版に差し替え」、台数に応じてSecurityGroupのPrivateIpAddressを調整、同一VPCに配置。過去問環境構築を容易にするツールとしてISUNARABE（KOBA789氏開発）を推奨 — [sadnessOjisan「ISUCON入門以前」ISUNARABE LT #1](https://speakerdeck.com/player/298ef05ffaf84a20ac31da0bea2e3018)
- NaruseJun（ISUCON13）: 準備物としてprivate-isu・ISUCON11予選・ISUCON9予選のCloudFormationテンプレートを用意。ISUCON9のものにはベンチ起動用の簡易Webポータルも作成。練習は本番に近い構成（複数台、ミドルウェア、ネットワーク）を目指した — [ISUCON13で優勝しました](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)
- NaruseJun（ISUCON12）: 本番と同じ構成のインスタンスで練習し、開始時にIPとドキュメントを共有。pprotein・phpMyAdmin・Netdataは外部ホストし、Ansibleでエージェント導入とトンネル設定 — [ISUCON12で優勝しました](https://zenn.dev/tohutohu/articles/8c34d1187e1b21)
- traPのPISCONポータルは matsuu/aws-isucon のAMIを使い、参加者が自分のAWSアカウント不要でインスタンス作成・削除・チーム結成ができ、スコア推移グラフも表示 — [ikura-hamu LT](https://speakerdeck.com/ikura_hamu/trap-nobu-nei-isucon-tosorewozhi-erupotaru)
- ローカル練習には matsuu/vagrant-isucon も使われる（対照例チームが前日に使用） — [kawaemon（Zenn）](https://zenn.dev/kawaemon/articles/f40529e0c447f5)
- private-isuをAWS上に構築する手順記事も存在 — [DevelopersIO: private-isuをAWS上に構築してみた](https://dev.classmethod.jp/articles/private-isu-aws/)
- 対照例（中位）: private-isuをISUCON本の内容に沿って約30万点、ISUCON12予選を約5万点まで練習 — [asamuzak（Zenn, ISUCON13）](https://zenn.dev/pharmax/articles/123ada69ad7635)

### Inferences
- 「1台＋ベンチ1台」の練習だけでは本番の複数台構成・DB分離の手順が身につかない。上位を狙うなら過去回のCFnテンプレート＋matsuu AMIで3台構成を再現するのが実戦的。
- ベンチ実行を楽にする仕組み（ポータル/スクリプト）を作ると、反復回数が増えて練習効率が上がる（NaruseJun・traP共通の発想）。

### Gaps
- ISUNARABEの現在の提供状況（稼働中か、対応問題、料金）は一次ソースで確認できていない。
- 公式（isucon.net）が過去問AMI/CloudFormationをどこまで公開しているかは今回未検証。

## チームの役割分担とコミュニケーション

### Takeaway
上位チームは「初動（最初の30〜60分）だけ役割を固定」し、その後は計測結果に基づいて各自が独立に改善を拾う流動的な分担を取る傾向。典型的な初動分担は ①マニュアル/コード読解とエンドポイント・テーブルの整理 ②Ansible等で計測基盤構築 ③リポジトリ化とデプロイスクリプト整備。

### Cited Findings
- NaruseJun（ISUCON12）初動: 1人がドキュメントとコードを読み、エンドポイント特性とテーブルのR/Wをスプレッドシートに記録／1人がAnsible実行とpprotein設定／1人がリポジトリにコードを入れデプロイスクリプト整備。以降は改善作業の固定分担なし（得意分野は大規模変更・重いクエリ修正・オンメモリキャッシュ）。各自ブランチで作業し、PRなしでmasterにマージ — [ISUCON12で優勝しました](https://zenn.dev/tohutohu/articles/8c34d1187e1b21)
- 同: デプロイ・ベンチは「次デプロイいいですか」「じゃあその次予約していいですか」と口頭でロック/予約。再起動試験はリーダーが号令、複雑なマージは画面共有 — [同上](https://zenn.dev/tohutohu/articles/8c34d1187e1b21)
- NaruseJun（ISUCON13）初動: sekai=Ansible・ツール・pprotein、takashi=CloudFormation・リポジトリ、とーふとふ=マニュアル・ドキュメント・実装確認・UI。本番中は各自が計測→分析→改善内容を宣言し、被ったら次点の改善を選ぶ。ブランチをデプロイしてベンチ確認後にmasterへマージ — [ISUCON13で優勝しました](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)
- NaruseJun: 「誰がインフラで、誰がデータベースといった具合に特に役割分担を決めているわけではない」 — [@IT](https://atmarkit.itmedia.co.jp/ait/articles/2209/29/news014.html)
- fujiwara組（ISUCON11優勝）: 「プログラムを書く人」「サーバーを設定する人」「モニタリングする人」の役割はあるが固定せず、状況に応じて入れ替える。手が空く時間を無駄とみなし、誰が何をしているか可視化して重複作業を避ける。「XXさんに任せきり」を避け全員で相談する“十種競技”型 — [カヤック インタビュー](https://www.kayac.com/news/2021/12/interview_ISUCON)
- 都営三田線東急目黒線直通急行日吉行（ISUCON14 4位、ISUCON7から同メンバー）: 0gajun氏がインフラ担当で鍵配置・ログ設定を「一瞬で」終え、その間に他2人がマニュアルとコードを読む。アプリ改善は残り2人。alp・スロークエリログ・mysqldumpslowの結果を全員で議論 — [LayerX（Zenn）ISUCON14参加記](https://zenn.dev/layerx/articles/3bd55b77e047c4)
- 最大の敵は時差（ISUCON8優勝、学生）: 中西=インフラ（サーバー・ミドルウェア）、市川=アプリコードを読み怪しい箇所の特定、薮=インフラとアプリ両方の簡単な修正。CTFチームTokyoWesternsで日常的に協働していたことがチームワークの基盤 — [ThinkIT](https://thinkit.co.jp/article/15737)
- 対照例: 「最初にボトルネック候補を10個ほど洗い出してから得意分野で分担」する方針（2人がalp/pt-query-digest、1人がコード読み） — [asamuzak（Zenn）](https://zenn.dev/pharmax/articles/123ada69ad7635)

### Inferences
- 初動の役割は「読む人」「計測基盤を作る人」「リポジトリ/デプロイを作る人」の3分割がNaruseJun・LayerXで共通。インフラ担当が自動化スクリプトで初動を数分に圧縮し、残り時間をアプリ改善に振るのが勝ちパターン。
- 本番中は固定分担よりも「改善内容を宣言→被りを避ける→デプロイ/ベンチの排他は口頭予約」という軽量プロトコルが機能している。
- 同一メンバーで複数年継続出場していること（NaruseJun 3回目で優勝、LayerXチームはISUCON7から）が上位チームの共通点。

### Gaps
- 使用チャットツール（Discord/Slack等）や通話形態の詳細は優勝チーム記事で明示されていない（NaruseJunはリモート参加が機能したとの言及のみ）。

## 事前に用意していた資産（初動スクリプト、デプロイ、Makefile、チェックリスト、ダッシュボード）

### Takeaway
優勝チームは「チートシート（初動・改善・ブラウザチェック確認）」「テンプレートリポジトリ（Makefile/deploy.sh/env.sh/ミドルウェア設定）」「Ansible」「自作計測基盤pprotein」を毎年練習でアップデートし、問題公開から即作業開始できる状態にしていた。

### Cited Findings
- NaruseJunのチートシートは「初動」「改善」「ブラウザチェック確認」の3領域。毎回見直し・更新し、問題公開からチューニング開始まで「ほぼすぐ作業に入れる」状態に — [gihyo.jp インタビュー](https://gihyo.jp/article/2022/10/isucon12-champinterview-narusejun)
- NaruseJunは自作ツール pprotein（Goのpprof等と連携する計測・共有基盤）を使用。sekai氏がAnsible構成と計測ツールを準備 — [gihyo.jp](https://gihyo.jp/article/2022/10/isucon12-champinterview-narusejun)；[pprotein（GitHub）](https://github.com/kaz/pprotein)
- NaruseJun（ISUCON12）: 7/9の宿題が「チートシート作成」「Ansible整備」「Goの復習」。初期セットアップ・デプロイ用スクリプト、規約/ドキュメントを事前準備。`make deploy BRANCH=<ブランチ名>` で deploy.sh を呼ぶ。複数台デプロイはtmuxのsynchronize-panes。初回Ansibleでホスト名変更・SSH用DNS設定も行う — [ISUCON12で優勝しました](https://zenn.dev/tohutohu/articles/8c34d1187e1b21)
- NaruseJun（ISUCON13）: `make deploy`でmaster、`make deploy BRANCH=hoge`で任意ブランチをデプロイ。リポジトリにアプリコード・env.sh・ミドルウェア設定を置き、設定変更もリポジトリ経由。pprotein導入後Makefileを簡素化。pprotein取得プロファイルでGoのPGOビルド。練習で見つけた問題をテンプレートリポジトリとpproteinにフィードバック — [ISUCON13で優勝しました](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)
- NaruseJun（ISUCON13）監視: `/initialize`末尾のフックでpproteinの計測開始。各サーバーでNetDataを動かしチームモニターに表示。iTermのブロードキャスト入力で全サーバーへ同時コマンド — [同上](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)
- takonomura（ISUCON9予選1位、1人参加）の事前スクリプト: サーバーのソースをGitHubへ（git init〜deploy key登録）、ツール一括インストール、ビルド・再起動・ログローテート、alp結果をGitHubへ投稿、ログ設定スニペット。使用ツールは netdata, alp, pt-query-digest, Stackdriver Trace/Profiler。「頻繁に行う作業に時間を使って時間が足りなくなる」ことを見越して用意 — [takono.io](https://www.takono.io/posts/2019/09/isucon/)
- 都営三田線…（ISUCON14 4位）: SQLコメントに呼び出し元のファイル名・関数名を付与する自作パッケージ（行番号は全角数字にしてmysqldumpslowのプレースホルダ置換を回避） — [LayerX（Zenn）](https://zenn.dev/layerx/articles/3bd55b77e047c4)
- 初動ワークフロー例（入門LT）: .ssh/config、webappをgit init、alp・percona-toolkit導入とログ有効化、scp＋設定配置＋サービス再起動のデプロイスクリプト、htop/tmuxで監視、nginx upstreamで分散、DBユーザーのホストをlocalhostから変更してDB分離、手順をチートシート化 — [sadnessOjisan LT](https://speakerdeck.com/player/298ef05ffaf84a20ac31da0bea2e3018)
- 対照例（中位）のチェックリスト: GitHub管理（LFS含む）、nginx/MySQL初期設定、pt-query-digest、インデックス/INSERT/トリガー練習、alp、systemctl/journalctl、サーバー分散をNotionに整理 — [asamuzak（Zenn）](https://zenn.dev/pharmax/articles/123ada69ad7635)

### Inferences
- 準備資産は「一度作って終わり」ではなく、練習のたびに不具合をテンプレートへ還流させる運用が鍵（NaruseJunが明示）。
- 計測を「ベンチ開始と同時に自動で取れる」状態（/initializeフック）にしておくと、毎ベンチ後の分析コストがほぼゼロになる。
- 1人参加の優勝者（takonomura）はスクリプト化の比重がさらに大きい。

### Gaps
- NaruseJunのチートシート本体・テンプレートリポジトリの中身は公開範囲を確認できていない（「秘伝のタレ」ファイルの内容も不明）。

## 戦略・マインドセット（計測ファースト、書き換えを急がない、最後の1時間ルール）

### Takeaway
共通原則は「推測するな、計測せよ」と「分かるところから直す」。最後の約1時間は改善を止めて再起動試験・ブラウザ動作確認・ログ/計測停止に充てるのが優勝チームの定型。

### Cited Findings
- sekai氏「推測するな、計測せよ」、とーふとふ氏「取りあえず自分に分かるところ、直せるところから直していく」 — [@IT](https://atmarkit.itmedia.co.jp/ait/articles/2209/29/news014.html)
- NaruseJun（ISUCON13）の分析順: alp（sum）→slp。Rows Sent/Rows Examinedが5倍程度以下なら健全、100倍程度ならインデックス不足の疑い。DB負荷が下がってからpprofフレームグラフ。最初の1時間で interpolateParams=true、アイコンのハッシュ/304、slpに基づくインデックス、タグのオンメモリキャッシュで11:00時点5,261点 — [ISUCON13で優勝しました](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)
- NaruseJun（ISUCON13）最終1時間（17:00〜）の再起動試験: 再起動後もベンチが通る／アプリ再起動なしの連続ベンチでも同スコア／`sudo systemctl status`で全サービスactive／各段階でフロントエンド機能確認。手順は「ベンチ→フロント確認→再起動→status→フロント確認→ベンチ→フロント確認→ベンチ→フロント確認」。最後にecho・pproteinのアクセスログ停止、効果のないinnodb_buffer_pool_sizeを戻し、pprof無効化 — [同上](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)
- NaruseJun（ISUCON12）: 17:00頃から再起動試験と「ほぼすべての操作ができるか」ブラウザ確認（ベンチの整合性チェックが軽そうだったため）。17:30に不要ログ・サービス停止、最終コミット17:31 — [ISUCON12で優勝しました](https://zenn.dev/tohutohu/articles/8c34d1187e1b21)
- NaruseJun（ISUCON12）: Redisは本番で試さず事前に練習しておくべきだったと反省（時間切れ） — [gihyo.jp](https://gihyo.jp/article/2022/10/isucon12-champinterview-narusejun)
- 最大の敵は時差（ISUCON8優勝）: 1変更ずつ入れてベンチを3〜4回回して確認。最後の1時間は変更せずベンチを繰り返す。大きな問題から先に潰し、小さな問題は残してもよい — [ThinkIT](https://thinkit.co.jp/article/15737)
- takonomura（ISUCON9予選）: 最小構成で初回ベンチ→監視追加→MySQLを別インスタンスへ分離。ベンチ待ち時間に次のサーバーを準備。改善はテーマ単位で並行 — [takono.io](https://www.takono.io/posts/2019/09/isucon/)
- 都営三田線…（ISUCON14 4位）: 初回ベンチでMySQLがボトルネックと分かった時点で明らかなインデックスを即追加、DBを早期に別インスタンスへ。反省は「プロダクト仕様を理解したうえで改善する必要がある」、変更起因のバグ修正に時間を取られた — [LayerX（Zenn）](https://zenn.dev/layerx/articles/3bd55b77e047c4)
- fujiwara組: 「直し方が分からなくてもとにかく手を動かします」（谷谷氏）、スコアを桁で変える「秘孔をつく」改善 — [カヤック](https://www.kayac.com/news/2021/12/interview_ISUCON)
- 対照例（失格）: ISUCON14でデータ保持違反により失格（失格前8位）。速く再現可能な計測・テストループと、早い段階での仕様理解が重要と振り返り — [kawaemon（Zenn）](https://zenn.dev/kawaemon/articles/f40529e0c447f5)；LayerXチームも過去にISUCON11本選・ISUCON13本選で失格経験 — [LayerX（Zenn）](https://zenn.dev/layerx/articles/3bd55b77e047c4)

### Inferences
- 上位でも失格は珍しくない。再起動試験・データ保持・ブラウザでの整合性確認を「最後の1時間の必須手順」として練習時から毎回通すことが、スコア向上と同等以上に重要。
- 「仕様理解」が後半の伸びを決める（ISUCON14ではマッチングなどアルゴリズム的改善が鍵だった模様。LayerX記事の「椅子の特性や地域要因」への反省から推測）。

### Gaps
- ISUCON本（『達人が教えるWebパフォーマンスチューニング』）著者らの準備法トークは今回取得できていない。

## 振り返り（練習後・本番後のレビュー）と開始時期

### Takeaway
優勝チームの振り返りは形式的なKPTより「練習で見つけた問題をテンプレートリポジトリ・ツール・チートシートへ即反映」する実務型。開始時期はチームとしては予選の約2週間〜1か月半前、個人の素振りは通年。

### Cited Findings
- NaruseJun（ISUCON13）: 練習で見つけた課題をテンプレートリポジトリとpproteinにフィードバック — [ISUCON13で優勝しました](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)
- NaruseJun: チートシートを毎回見直し・更新 — [gihyo.jp](https://gihyo.jp/article/2022/10/isucon12-champinterview-narusejun)
- NaruseJun（ISUCON12）: 7/9に始動（予選7/23の約2週間前）、本選8/27までの約1か月で本選対策 — [ISUCON12で優勝しました](https://zenn.dev/tohutohu/articles/8c34d1187e1b21)；[gihyo.jp](https://gihyo.jp/article/2022/10/isucon12-champinterview-narusejun)
- NaruseJun（ISUCON13）: 11/14キックオフ、11/18通し練習、本番11/25（約11日前に始動、ただし個人で過去問約10問ずつ） — [ISUCON13で優勝しました](https://zenn.dev/tohutohu/articles/923bdf5dcd73af)；本番日付は [asamuzak](https://zenn.dev/pharmax/articles/123ada69ad7635)
- 最大の敵は時差: ベンチの回し方は過去の敗戦からの学びが大きい — [ThinkIT](https://thinkit.co.jp/article/15737)
- traP PISCON: 過去問コンテスト後に解法共有会を実施 — [ikura-hamu LT](https://speakerdeck.com/ikura_hamu/trap-nobu-nei-isucon-tosorewozhi-erupotaru)
- 対照例の振り返りでは「1か月前には始めるべき」「通年で過去問を定期的に解く」が繰り返し出る — [kawaemon](https://zenn.dev/kawaemon/articles/f40529e0c447f5)；[irsc](https://zenn.dev/irsc/articles/isucon14-diary)
- 上位チームは上位勢の公開GitHubリポジトリを読んで学ぶ（中位チームの事後学習例） — [tockey](https://zenn.dev/tockey/articles/0d55e1b9109615)

### Inferences
- 振り返りの成果物は「次回すぐ使えるもの（スクリプト・チートシート・ツール改修）」に落とすのが上位流。議事録より資産化。
- 練習後は上位チームの公開リポジトリ・講評（公式「問題の解説と講評」）と自分の解法を突き合わせる学び方が一般的。

### Gaps
- 優勝チームが練習後に定型の振り返りフォーマット（KPT等）を使っているかの記述は見つからなかった。
- 2025年以降（ISUCON15相当/ISUCON2026）の上位チーム準備記事は今回の検索範囲では見つからなかった。
