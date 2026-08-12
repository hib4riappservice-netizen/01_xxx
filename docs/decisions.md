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

## 2026-08-12: CSP の `script-src` に `unsafe-inline` を許可する

**背景**: `checklists/release.md` SEC-92「`unsafe-inline` を避けている」を満たすため、
`next.config.ts` で `script-src 'self'` を試した。

**決定**: `script-src 'self' 'unsafe-inline'` を採用する。

**理由**: `next build && next start` の状態で Playwright を使って実測したところ、
`unsafe-inline` を外すと Next.js が RSC のペイロードを渡すために注入するインラインscriptが
CSP違反でブロックされ、ハイドレーションが壊れた（`React error #412`、コンソールにCSP違反2件）。
現時点ではペイロードに機微情報を含む機能（認証・決済等）が無いため、この妥協を許容する。

**見直し条件**: 認証や決済など、XSS経由のインラインscript実行が実害に直結する機能を追加する
タイミングで、`middleware.ts` によるnonceベースのCSPへ移行し、`unsafe-inline` を外す。
