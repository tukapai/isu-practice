# isu-practice

ISUCON での優勝を目標にした練習・試行錯誤用リポジトリ。

- コード・ハーネス・計測スクリプト → このリポジトリ
- 読書メモ・調査メモ・振り返りなどのドキュメント → Notion「ISUCON 学習」
  - https://app.notion.com/p/40a34acb098c818382adc076145b44a9

## ディレクトリ構成

| パス | 用途 |
| --- | --- |
| `book/` | ISUCON本（改訂新版 達人が教えるWebパフォーマンスチューニング）の写経・ハンズオン |
| `harness/` | AIハーネス／自律型性能改善エージェントの試作 |
| `tools/` | 計測・解析・デプロイ用のスクリプト（alp, pt-query-digest, pprof など） |
| `practice/` | 過去問の練習記録。`practice/<年>-<予選|本選>/` ごとに作業ログとスコア推移を残す |
| `docs/` | コードと一緒に管理したい短いメモ（長文は Notion へ） |

## 進め方のルール

- 1回の試行 = 1ブランチ or 1コミットを目安にし、コミットメッセージに「何を変えたか」と「スコアの前後」を書く
  - 例: `perf(db): add index on posts(user_id, created_at) score 12000 -> 18500`
- 計測結果（alp / slow log の集計など）は `practice/<回>/logs/` に置く。生ログは `.gitignore` 済み
- うまくいかなかった試行も消さずに残す（なぜダメだったかを一行添える）
