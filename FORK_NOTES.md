# Fork notes: lkha/daily_stock_analysis

**Монгол товч:** Энэ fork нь upstream `ZhuLinsen/daily_stock_analysis` @ `db0570e` (MIT) дээр суурилсан. Ажлын өдөр бүр 18:00-д (Улаанбаатар) АНУ-ын хувьцааны шинжилгээ болон зах зээлийн тоймыг Telegram руу **монгол хэлээр** илгээдэг. Монгол хэл нь `REPORT_LANGUAGE=en` слотыг орлосон тул энэ fork-д `en` гэдэг нь монгол гэсэн үг.

## What differs from upstream

The `en` report language renders Mongolian (Cyrillic):

- **LLM output**: the output-language directives in `src/analyzer.py` (system prompt and prompt tail) and the market-review shell language in `src/market_analyzer.py` ask for Mongolian. `operation_advice`, `trend_prediction`, `confidence_level` and `stock_name` stay English canonical values, because the app localises them itself and uses them for icons and decision inference.
- **Static text**: the `en` label table, translation maps and placeholders in `src/report_language.py` are Mongolian, and Mongolian surface forms are added to the canonical maps. Dashboard section labels (`src/notification.py`), market-review titles (`src/core/market_review.py`) and two fallbacks (`src/core/pipeline.py`) are Mongolian too.
- **Robustness**:
  - Mongolian text needs far more tokens than English and was cut off at `max_output_tokens=8192`, so the directives ask for compact output.
  - `_reattach_members_after_early_close` repairs a root JSON object that the model closed one brace early.
  - Decode failures log an `[LLM JSON debug]` line with the error and the response tail.

`tests/test_report_language.py` is updated. Other upstream tests that assert English text for `en` (for example `tests/test_market_review.py`, `tests/test_notification*.py`, `tests/test_share_image.py`, `tests/test_market_analyzer_generate_text.py`) fail by design.

## Configuration

GitHub → Settings → Secrets and variables → Actions:

- Secrets: `DEEPSEEK_API_KEY`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`
- Variables: `STOCK_LIST=NVDA,AAPL,MSFT,SPY,QQQ`, `LITELLM_MODEL=deepseek/deepseek-v4-flash`, `LITELLM_FALLBACK_MODELS=deepseek/deepseek-v4-pro`, `MARKET_REVIEW_REGION=us`, `REPORT_LANGUAGE=en`

Workflow `.github/workflows/00-daily-analysis.yml` runs on cron `0 10 * * 1-5` (10:00 UTC = 18:00 Asia/Ulaanbaatar, Mon-Fri). Manual run (`force_run` skips the trading-day check):

```bash
gh workflow run 00-daily-analysis.yml --repo lkha/daily_stock_analysis -f mode=full -f force_run=true
```

## Operating notes

- A green run is not proof of delivery: Telegram errors such as `chat not found` are only logged as warnings. Check the log for `通知发送完成：成功 N 个`.
- GitHub pauses scheduled workflows in public repositories after 60 days without repository activity.
- Never put `#patch`, `#minor` or `#major` in a commit message on `main`: `auto-tag.yml` would push a tag, which triggers the release and docker workflows.

## Updating from upstream or going back to English

- **Update**: fetch `upstream`, rebase the fork commits onto the new upstream commit, run a manual `mode=full` test, then move `main`.
- **Back to English**: `git revert` the Mongolian commits on top of `db0570e`. Do not force-push `main`.
