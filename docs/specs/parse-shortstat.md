# git diff --shortstat のパーサーを純粋関数として切り出す

## 1文の説明

`scripts/check-diff-size.mjs` 内の `git diff --shortstat` 出力の正規表現パースを、
テスト可能な純粋関数 `parseShortstat()` として `lib/` に切り出し、ユニットテストを書く。

## 受け入れ条件（Given / When / Then）

- Given `"2 files changed, 10 insertions(+), 3 deletions(-)"` / When `parseShortstat()` に渡す /
  Then `{ insertions: 10, deletions: 3 }` を返す
- Given `"1 file changed, 5 insertions(+)"`（deletions が無い） / When `parseShortstat()` に渡す /
  Then `{ insertions: 5, deletions: 0 }` を返す
- Given `""`（差分なし） / When `parseShortstat()` に渡す /
  Then `{ insertions: 0, deletions: 0 }` を返す

## 個人情報・課金情報の判定

含まない。

## 触るファイル

- `lib/parse-shortstat.ts`（新規）
- `tests/parse-shortstat.test.ts`（新規）
- `scripts/check-diff-size.mjs`（`lib/parse-shortstat.ts` を使うよう置き換え）

## 停止条件（機械判定可能な形で1行）

`pnpm test tests/parse-shortstat.test.ts` が緑になる

## ループで実行する場合

- turn上限: 5（既定。rules/70-loop-engineering.md LOOP-01）
- 作業場: git worktree（`../01_xxx-loop-parse-shortstat`）
- no-progress条件: 同一テストが2回連続で赤なら停止（既定）

## 満たせなかった条件

（実装後に記入）
