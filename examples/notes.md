# run-toolcall-recover — インシデント記録

復旧のたびに1件追記する。フォーマット: 日付 / 症状 / 推定原因 / 効いた手 / 予防策。
本体（SKILL.md / hook）への反映は run-toolcall-improve が行う（改善履歴は同スキルの history.md）。

---

## 2026-06-09 — 初回（このスキルを作る契機になった崩壊）

- **症状**: 返信の地の文に `court` という語＋invoke タグ・parameter タグがそのまま出力された。「Your tool call was malformed and could not be parsed」が複数回連続。Edit/Write が適用されず（grep で未反映を確認）、同じ編集を何度も出し直す事態に。
- **推定原因**: 直前のツール結果が文字化け（file-state メッセージの混線・コマンドのエコーバック・無関係タグの混入）し、文脈が汚染された。その結果、次トークン生成が正規の function-call 前置（本来の invoke 形式）ではなく、壊れた前置トークン `court` を吐く方向に脱線した。malformed なのでパーサが弾き、残骸が地の文に表示された。
- **効いた手**:
  1. ツール呼び出しを**正規 function-call 形式で出し直す**（`court` や生タグを付けない）。これが通れば即復旧。
  2. 推論前提が化けた結果に依存していた箇所は、`grep`/`ls`/単一 `Read` など**最小のクリーンな呼び出し**で ground truth を取り直す（例: SKILL.md の編集が効いたか `grep` で確認、bd の close 状態を `bd show ... | grep status` で確認）。
  3. 大量出力（`cat` 全文等）や連続ツール呼び出しを避け、1コマンドずつ確認しながら進める。
- **予防策（判明分）**:
  - 地の文に invoke/parameter タグを手書きしない（常に正規チャネル）。
  - 化けた結果を根拠に推論を続けない。まず最小再確認。
  - `Stop`/`SubagentStop` hook で最終メッセージの生マークアップを検出し block→再生成（クリーンになるまで撤退しない）。
- **未解決**: 根本原因の「ツール結果文字化け」は環境側の事象でスキル/hook では防げない。検知・粘り強い復旧で被害を抑えるのみ。

## 2026-06-10 — hook 検証と「表示前検知」の限界

- **検証結果**: detect-toolcall-leak.sh を合成 transcript で確認。leak→block / clean→NO-BLOCK / fenced→NO-BLOCK の 3/3 で正しく動作。
- **テスト時の罠**: (1) `jq -n` は既定で複数行 pretty 出力 → 実 transcript の JSONL(1行=1JSON) と食い違い解析されない。テストは `jq -nc`。(2) フェンスをバックスラッシュ・エスケープすると実際の素の三連バックティックと変わり sed が一致せず誤検知。テストデータは素のバックティックで作る。
- **設計上の確定事項（重要）**: 「ユーザー表示前」に hook で止めることはできない。理由: 漏れは malformed tool call で、パース失敗ゆえ PreToolUse は発火せず、テキストは既にパーサ層でストリーム表示済み、トークン単位/表示前に走る hook も存在しない。Stop/SubagentStop が最速の捕捉点(事後)。表示前抑制は CLI を包む出力フィルタ(非hook・重い・正規出力破壊リスク)しかない。現実解は「生成側の規律＋Stop hook の安全網」。
