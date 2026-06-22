# toolcall-recover

> ツール呼び出しの出力崩壊を検知し、クリーンになるまで粘って復旧させる Claude Code プラグイン。

## これは何の問題を解くのか

ツール結果が文字化けする（環境側の障害）と文脈が汚染され、次トークン生成が脱線して
**生のツール呼び出しマークアップ（`court` という壊れた前置トークン、`invoke` タグ、`parameter` タグ、`<function_calls>` など）が返信の地の文にそのまま漏れる**ことがあります。
こうなると「Your tool call was malformed and could not be parsed」が連発し、編集が適用されないまま同じ呼び出しを繰り返す——という崩壊ループに陥ります。

このプラグインは、その崩壊を **検知 → 粘り強い復旧 → 記録 → 自己改善** の4段で抑え込みます。

- **検知**: `Stop`/`SubagentStop` hook が最終メッセージ内の生マークアップを検出し、クリーンになるまで `block` して再生成を強制する（撤退・回数制限なし）。
- **復旧**: `run-toolcall-recover` スキルが「正規の function-call チャネルで出し直す／化けた結果を根拠に推論しない／単一・最小の呼び出しで ground truth を取り直す」という行動規律を与える。
- **記録**: 復旧のたびに症状・原因・効いた手・予防策を `notes.md` に1件追記する。
- **自己改善**: `run-toolcall-improve` スキルが、溜まった記録から繰り返しパターンや新しい漏れ署名を抽出し、検知 hook や復旧規律へ反映する（編集はサブエージェントへ隔離委譲）。

設計の白眉は **復旧と自己改善の分離** です。復旧直後の文脈は汚染されている可能性が高いため、その場で本体（SKILL.md / hook）を編集しません。改善は必ずクリーンな文脈で別途 `run-toolcall-improve` を起動し、実ファイル編集はサブエージェントに委ねてメイン文脈から隔離します。

## 構成

```
toolcall-recover/
├── .claude-plugin/
│   ├── plugin.json          # プラグイン定義
│   └── marketplace.json     # このリポジトリ自身をマーケットプレイス化
├── hooks/
│   ├── hooks.json           # Stop / SubagentStop へ検知 hook を登録
│   ├── detect-toolcall-leak.sh   # 漏れ検知 → block(再生成強制)。判定不能時のみ fail open
│   └── test-detect-toolcall-leak.sh  # 6パターンの回帰テスト
├── skills/
│   ├── run-toolcall-recover/     # その場の復旧 + notes.md 記録
│   └── run-toolcall-improve/     # 記録からのパターン抽出 → 本体へ反映
├── examples/                # 作者の実インシデント記録（書き方の見本）
├── LICENSE
└── README.md
```

## インストール

### マーケットプレイス経由（推奨）

```
/plugin marketplace add nam-tech-studio/toolcall-recover
/plugin install toolcall-recover@nam-tech-studio
```

インストールすると検知 hook が `Stop`/`SubagentStop` に自動登録されます（`settings.json` の手編集は不要）。

### ローカルから（開発・記録を残したい場合）

```
git clone https://github.com/nam-tech-studio/toolcall-recover
claude --plugin-dir ./toolcall-recover
```

> **記録→自己改善のループを回すなら、こちらのローカルクローンを推奨します。**
> マーケットプレイス経由のインストール先は更新で上書きされうるため、`notes.md` への追記や `run-toolcall-improve` による本体更新は、書き込み可能なローカルクローン上で行うのが安全です。

## 動作確認

検知 hook の回帰テスト（6パターン: leak / clean / fenced / 単独court / 文中court / 単独タグ）:

```
bash hooks/test-detect-toolcall-leak.sh
# => pass=6 fail=0
```

`jq` が必要です（未インストールなら hook は fail open し、通常作業を止めません）。

## 設計上の限界（既知）

- **「ユーザー表示前」には止められません。** 漏れは malformed tool call であり、パース失敗ゆえ `PreToolUse` は発火せず、テキストは既にパーサ層でストリーム表示済みです。`Stop`/`SubagentStop` がターン末の最速の捕捉点（事後）になります。
- **根本原因（ツール結果の文字化け）は環境側の事象**で、スキル/hook では防げません。本プラグインは検知と粘り強い復旧による被害抑制です。
- 検知 hook は誤検知回避のため**コードフェンス内の言及を除外**します。タグを文書中で例示するときは必ずコードフェンスに入れてください。

## ライセンス

MIT License — 詳細は [LICENSE](./LICENSE) を参照。
