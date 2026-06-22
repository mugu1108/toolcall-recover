# run-toolcall-improve — 改善履歴台帳

本体（recover SKILL.md / detect-toolcall-leak.sh / このスキル）への反映を1件ずつ記録する。
手順1で読み、反映済みの教訓を二重反映しないための台帳。インシデント自体の記録は
`~/.claude/skills/run-toolcall-recover/notes.md`（純インシデントログ）にある。

---

## 2026-06-10 — スキル分割

recover（旧 recover-toolcall-hygiene）を「その場の復旧＋notes.md 記録」のみに責務を絞り、自己改善（notes.md からのパターン抽出 → SKILL.md/hook 更新のサブエージェント委譲 → 検証）を本スキル（旧 improve-toolcall-hygiene）に分離。理由: 復旧直後の汚染文脈で本体を編集する事故を防ぐため。あわせて hook の block reason を「notes.md 記録まで／本体更新は improve へ」に更新し、合成 transcript 6パターン（leak/clean/fenced/単独court/文中court/単独タグ）で回帰テスト 6/6 PASS を確認。回帰テストは `~/.claude/hooks/test-detect-toolcall-leak.sh` に永続化。

## 2026-06-10 — assign-agent-skill-evaluator 評価への全指摘対応

評価結果: recover 74点（不合格）/ improve 82点（合格）。主因はともに prefix が v3 taxonomy allowlist 違反。対応:

- **改名**: recover-toolcall-hygiene → `run-toolcall-recover`、improve-toolcall-hygiene → `run-toolcall-improve`（4軸 produce/user 系 → `run-*` 導出に準拠）
- **argument-hint 空欄を削除**（両スキル。引数を読まないため）
- **improve の description をトリガー条件に圧縮**（ワークフロー要約の混入を解消）
- **改善履歴を notes.md から本 history.md に移設**（notes.md を純インシデントログに戻し、recover/improve の境界を明確化）
- **hook の block reason・両 SKILL.md の相互参照・notes.md ヘッダを新名称に更新**し、回帰テストを再実行

## 2026-06-10 — 再評価(iter2: recover 93 / improve 93)の medium 2件対応

- **description の純化**: 末尾の why（「recover とは分離された自己改善スキル」）を削除し、発動ワード＋発動条件のみに。分離理由は本文「分離の理由」節が既に保持。
- **Axis C の一意化**: 本文冒頭に「形は orchestrated」段落を追加（メインが手順1〜5を指揮、編集のみ Task で dispatch、検証・記録はメイン、fork なし）。
