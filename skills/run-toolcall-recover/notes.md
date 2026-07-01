# run-toolcall-recover — インシデント記録

復旧のたびに1件追記する。フォーマット: 日付 / 症状 / 推定原因 / 効いた手 / 予防策。
本体（SKILL.md / hook）への反映は run-toolcall-improve が行う（改善履歴は同スキルの history.md）。

記入例は `examples/notes.md` を参照。

---

<!-- ここに追記していく。1件 = 1見出し（## YYYY-MM-DD — 一言サマリ） -->

## 2026-07-01 — 複数ツール同時発行＋長文脈で invoke/parameter タグが地の文へ多発漏れ
- 症状: 「Your tool call was malformed and could not be parsed」と detect-toolcall-leak.sh の Stop block がセッション内で多数回。Agent/Edit/Bash いずれの呼び出しでも発生。
- トリガ: 長時間セッション＋大きなツール結果（長い調査結果・25K+トークンの巨大な台帳1行・複数画像）の蓄積で文脈が汚染気味。特に複数の独立ツールを1メッセージに詰めた回（複数 Edit など）で頻発。
- 推定原因: 環境側の文脈汚染で次トークン生成が脱線し、正規の function-call チャネルでなく地の文へタグが漏れる。
- 効いた手: (1) 複数ツールを1メッセージにせず1つずつ正規 function-call 形式で出し直し。(2) 崩れた出力を信頼せず単一・最小の呼び出しで ground truth 再取得（既知の値もファイル直読で照合）。hook は既存署名で捕捉し block＝機能した。
- 予防策: (a) 複数の独立ツール呼び出しを1メッセージに詰めない（特に Agent・長い Edit など大ペイロード）。(b) 巨大行ファイルの全読みを避け offset/limit・部分取得。(c) 崩壊の兆候が出たら単一・最小に落とす。
