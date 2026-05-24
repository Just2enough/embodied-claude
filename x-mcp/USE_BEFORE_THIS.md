# ⚠ x-mcp 使用前注意（Just2enough fork ローカル）

## 結論: このフォークでは x-mcp を使用しない

`Just2enough/embodied-claude` では、セキュリティ監査の結果に基づき **x-mcp 全機能を使用しない方針**としている。
このファイルはフォーク独自の注意書きで、upstream（`lifemate-ai/embodied-claude`）には存在しない。

## 理由

`post_tweet` / `delete_tweet` ツールに **投稿前の human-in-the-loop ゲートが存在しない**。
- `boundary-mcp.evaluate_action` も `sociality-mcp.review_social_post` も呼んでいない
- tweepy `client.create_tweet(**kwargs)` / `client.delete_tweet(tweet_id)` を直接実行
- LLM が誤ってツールを選んだ瞬間に投稿/削除される

詳細: [`../docs/local/SECURITY_AUDIT_2026-05-24.md`](../docs/local/SECURITY_AUDIT_2026-05-24.md) §3.1

## 使う場合に必要な改修

このフォークで x-mcp を使いたくなったら、**先に以下の改修を施すこと**:

### 必須

1. **`src/x_mcp/server.py:126-170` の `post_tweet` / `delete_tweet`**
   - `require_review: bool = True` パラメータを追加
   - `require_review=True` のときは sociality-mcp の `review_social_post()` を呼んで承認を取る
   - 承認が取れなかった場合は投稿/削除しない
2. **`.mcp.json` / `.mcp.local.json` での起動経路を追加**
   - 起動時に `--require-review` フラグを必須化
3. **`docs/local/MCP_USAGE_POLICY.md` の x-mcp 行を更新**
   - ❌ → ⚠（条件付き使用可）に変える

### 推奨

- `dry_run: bool = False` パラメータも追加し、テスト時は実投稿せず JSON で結果プレビュー
- 投稿ログを `~/.claude/x-mcp/posts.jsonl` 等に append-only 保存（事後監査用）

## 改修を行わない場合の運用

- **`.mcp.json` から x-mcp を除外する** — そもそも起動しない
- 環境変数 `X_CONSUMER_KEY` / `X_CONSUMER_SECRET` / `X_ACCESS_TOKEN` / `X_ACCESS_TOKEN_SECRET` / `XAI_API_KEY` / `GROK_API_KEY` を **設定しない** — 設定があっても起動時 `RuntimeError` で落ちる安全側設計になっている

## 上流の状況

upstream（`lifemate-ai/embodied-claude`）には PR していない。本フォーク独自の判断。
upstream の状況が変わったら（投稿前ゲートが入ったら）本ファイルを更新すること。
