# 技術的決定の記録

`rules/60-delivery-ops.md` OPS-04（SHOULD）に基づく。判断とその理由を残す。

## 2026-08-12: ブランチ保護をローカルフックで代替する

**背景**: DEV-01 (MUST)「`main` に直接コミットしない」をサーバー側（GitHub branch
ruleset）で強制しようとしたが、非公開リポジトリでは GitHub Pro 以上が必要
（Free プランは 403 で拒否される）。

**決定**: `.githooks/pre-commit` でローカルに同等のゲートを実装する。
`git config core.hooksPath .githooks` で有効化し、`main` ブランチでの
コミットを拒否する。緊急時は `ALLOW_COMMIT_ON_MAIN=1` で解除できる。

**理由**: `00-principles.md` P6「人の注意力に依存しない。機械が落とせる形に
変換する」に従う。サーバー側で強制できないなら、次善としてローカルで
機械的に落とす。

**既知の限界**: `core.hooksPath` はクローンごとのローカル設定であり、
リポジトリに含めても自動では有効化されない。新しく clone した環境では
`git config core.hooksPath .githooks` を手動で実行する必要がある
（`README.md` のセットアップ手順に明記すること）。また `git commit --no-verify`
で回避できる — これは Git 標準機能であり、フック自体では防げない。
真の強制が必要になったら、リポジトリを Public にするか GitHub Pro に
アップグレードして ruleset を使う。
