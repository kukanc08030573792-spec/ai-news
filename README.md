# AI・SNSマーケティング ニュース自動収集

AI業界とSNSマーケティングのニュースを**毎朝6時台（JST）に自動収集**し、Webサイトとして公開するシステムです。追加料金なし（GitHub無料枠のみ）で動作します。

既存の [robot-news](https://github.com/kukanc08030573792-spec/robot-news) と同じ仕組みですが、**ソースの分類方法**が違います。

## 設計 — 分類はAIに推測させず宣言する

このシステムの目的は「**公式発表と重要人物の発言を取りこぼさずキャッチアップする**」ことです。そのため次の方針で作っています。

- **tier（公式／重要人物／メディア）は `sources.yml` での宣言をそのまま使います。** 「公式かどうか」はソースの属性であって、記事本文から推測させると誤ります。OpenAIのブログは常に公式、Simon Willisonの記事は常に本人発信、として確定的に扱います。
- **公式・重要人物の記事は、話題判定にかかわらず必ず残します。** 捨てるのはメディア記事が話題外と判定されたときだけです。
- **取り込み枠はtierごとに確保します。** 公式50%・重要人物30%・メディア40%（使い切らなかったぶんは融通）。単純に公式を優先して上から切ると、公式の本数が多い日に重要人物の発信が1件も入らなくなるためです。
- **要約欄は配信元の説明文を整形して出します。** AIによる要約・翻訳は行いません。「お知らせ｜」や末尾の媒体名、`（画像）（4/6枚目）` といった定型を落とし、文字化けしている説明文は捨ててタイトルだけ見せます。
- **話題（AI／SNSマーケ）の判定はキーワードで行います。** 公式・重要人物はソースの宣言を基本にし、本文が明確に反対を示すときだけ覆します。メディアはASCII.jpのような総合ITフィードも含むため宣言を鵜呑みにせず、本文に根拠がある記事だけを採用します。

> **補足：なぜAI要約を使わないか**
> robot-news が使っていた **GitHub Models は 2026-07-30 に完全廃止されました**（[GitHub Changelog](https://github.blog/changelog/2026-07-01-github-models-is-being-fully-retired-on-july-30-2026/)）。robot-news は 2026-08-01 以降、AI要約が効かないまま動いています（収集自体は正常なので気づきにくい状態です）。
> 本システムは外部AIに依存せず、APIキーの登録も不要で動きます。そのぶん**英語ソースの要約は英語のまま**です。

## 収集元（全URL 2026-09-16 に取得して生存確認済み）

| 区分 | 主なソース |
|---|---|
| AI公式 | OpenAI／Google DeepMind／Google Research／Microsoft／Mistral／NVIDIA／AWS ML／Hugging Face／Google Developers／GitHub Blog |
| SNS・広告公式 | Meta Newsroom／YouTube公式ブログ／Google 広告・コマース公式 |
| AI重要人物 | Import AI（Jack Clark）／Simon Willison／Sam Altman／Andrej Karpathy／Ethan Mollick／Lilian Weng／Nathan Lambert／Latent Space／Ben Thompson／深津貴之／梶谷健人 |
| SNS重要人物 | Jon Loomer（Meta広告） |
| AIメディア | ITmedia AI+／ASCII.jp／TechCrunch AI／The Verge AI／MIT Tech Review／Ars Technica |
| SNSマーケメディア | Social Media Today／Social Media Examiner／Buffer／Hootsuite／Search Engine Journal／Adweek／ソーシャルメディアラボ／MarkeZine／AdverTimes／ITmediaマーケティング／Web担 |
| YouTube公式 | Anthropic／OpenAI／Google DeepMind／Creator Insider（YouTube公式）／Meta |

### 公式RSSが存在しないサービスの扱い

**Anthropic・TikTok・Instagram・X・LinkedIn はRSSを提供していません。**（2026-09-16 に確認。X Blogは外部アクセス自体が403）

これらは次の2段構えで拾います。

1. **Google Newsキーワード検索**（`sources.yml` の `proxy_for` 付きクエリ）— サイトでは「公式」タブに〈報道経由〉バッジ付きで表示されます
2. **Social Media Today** — TikTok・Instagram・Xの公式発表を当日〜翌日に記事化しており、実質的な一次情報の受け皿になります

それでも本家の告知を直接見たいときのために、サイトの「🔗 公式リンク」タブに手動チェック用リンクを置いています。

---

## セットアップ手順

### 1. Actionsの書き込み権限を有効化

リポジトリの **Settings → Actions → General → Workflow permissions** で
**「Read and write permissions」を選択 → Save**（botがコミットするために必要）

### 2. GitHub Pagesを有効化

**Settings → Pages → Build and deployment** で
- Source：**Deploy from a branch**
- Branch：**main** ／ フォルダ：**/docs** → Save

数分後、`https://<ユーザー名>.github.io/ai-news/` でサイトが見られます。

### 3. 初回収集を実行

**Actionsタブ → collect-news → Run workflow** で手動実行。
3〜5分で完了し、サイトに記事が並びます。以降は毎朝自動実行されます。

**APIキーやシークレットの登録は不要です。**

---

## 日常の使い方

| したいこと | 方法 |
|---|---|
| 公式発表だけ見る | サイトの「🏛 公式」タブ（初期表示） |
| 重要人物の発信だけ見る | 「🎙 重要人物」タブ |
| AIだけ／SNSだけに絞る | 検索窓の右の「AI」「SNSマーケ」チップを押す（両方オフ＝絞り込みなし） |
| RSSが無い公式を確認 | 「🔗 公式リンク」タブから各社のニュースページへ |
| ブックマーク | 記事の「☆ 保存」→ Issue画面が開く → そのまま「Submit new issue」→ 1〜2分で★タブに反映 |
| 収集対象を増減 | `sources.yml` をGitHub上で編集（鉛筆アイコン）→ Commit |
| 重要人物を追加 | `sources.yml` の `sources:` に `tier: voice` で追記 |
| 今すぐ収集 | Actionsタブ → collect-news → Run workflow |
| 通知が欲しい | サイトの「RSS」リンクをRSSリーダー（Feedly等）に登録 |

## 仕組み

```
毎朝6時台 GitHub Actions（collect.yml）
  → scripts/collect.py が全ソースを巡回
      RSS 42本 ＋ Google News 9クエリ ＋ YouTube 5チャンネル
  → tier は sources.yml の宣言をそのまま採用
  → tierごとに取り込み枠を配分（公式50%／人物30%／メディア40%）
  → Google NewsのリダイレクトURLを実URLへ復号し、OGP画像を取得
  → キーワードで話題分類（ai/sns/other）、説明文を整形して要約欄へ
  → 公式・人物は全件保持／メディアは other を除外
  → docs/data/articles.json（最新600件）＋ archive/YYYY-MM.json（全量）
    ＋ feed.xml を更新
  → GitHub Pagesが自動配信

☆保存ボタン → 入力済みIssue → bookmark.yml → scripts/bookmark.py
  → 所有者本人か確認 → bookmarks.json 更新 → Issue自動クローズ
```

## 補足・注意

- **費用**：公開リポジトリはActions実行が無料・無制限、Pagesも無料。外部APIを使わないので**完全無料**です。
- **実行時刻**：robot-news（06:07〜）と重ならないよう、**06:17/06:47/07:47/08:17 JST**に仕掛けてあります。
- **公開範囲**：サイト・データ・設定はすべて公開されます（URLを知らなければ実質見つかりませんが、非公開ではありません）。
- **要約の性質**：要約欄は配信元の説明文を整形したものです。英語ソースは英語のまま出ます。説明文が空・文字化けの場合はタイトルだけ表示します。
- **サムネイル**：記事のOGP画像・YouTube公式サムネを直接参照しています（画像はこのリポジトリに保存しません）。
- **収集しないことにしたソース**：MarkeTRUNK（profuture.co.jp）は収集スクリプトのUser-Agentに403を返すため、bot拒否の意思表示とみなして対象から外しました。ferret・Ledge.aiはRSS提供を終了していました。
- **Web担当者フォーラム**：RSSは返りますが2026-07-21以降の更新が確認できませんでした。しばらく記事が流れてこないようなら `sources.yml` から削除してください。
- **Substackの `*.substack.com` は使えません**：GitHub Actionsの実行元IPから403が返ります（ローカルからは取得できるので気づきにくい）。Substack発でも独自ドメインを持つもの（oneusefulthing.org・interconnects.ai・latent.space・jack-clark.net）は問題なく取得できています。新しくSubstackの書き手を足すときは、独自ドメイン側のURLを探してください。
- **YouTubeチャンネルの追加**：`channel_id` 直書きを推奨します。ハンドルからの解決はページ内の `"channelId"` を使うと**別チャンネルを拾います**（@Meta が Facebook チャンネルに化ける例を確認済み）。`collect.py` は canonical / externalId から解決しています。
- **メンテナンス**：60日間コミットがないとGitHubがスケジュール実行を自動停止します。毎日コミットが発生する本システムでは通常起きませんが、長期停止後はActionsタブで再有効化してください。
