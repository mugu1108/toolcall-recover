# run-toolcall-improve — 改善履歴台帳

本体（recover SKILL.md / hooks/detect-toolcall-leak.sh / このスキル）への反映を1件ずつ記録する。
手順1で読み、反映済みの教訓を二重反映しないための台帳。インシデント自体の記録は
`skills/run-toolcall-recover/notes.md`（純インシデントログ）にある。

記入例は `examples/history.md` を参照。

---

<!-- ここに追記していく。1件 = 1見出し（## YYYY-MM-DD — 一言サマリ） -->

## 2026-07-01 — 複数ツール同時発行ガードを SKILL.md に昇格＋初インシデントを notes.md に記録
- 反映元: notes.md 2026-07-01（複数ツール同時＋長文脈でタグ漏れ多発）。
- 変更: run-toolcall-recover/SKILL.md の「最重要ルール」に「複数ツールを1メッセージに詰めない（崩壊の予防）」を追加。notes.md に当該インシデントを記録。
- hook: 変更なし（既存署名で捕捉・block 済みのため署名追加不要）。回帰テストも変更不要。
