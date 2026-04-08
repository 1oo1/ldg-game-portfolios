---
name: update-portfolio-from-screenshots
description: '根据持仓截图更新 ldg-game-portfolios 的 index.html 月度组合数据。用户提到 持仓截图、港股、A股、更新 index.html、补录月度持仓、根据截图改游戏仓数据、整理仓位明细 时务必使用。先从截图提取月份、标的、持股数量、市值、现金和市场类型，再核对 data 数组结构，判断是追加新月份还是覆盖已有月份，并以最小改动更新页面。'
argument-hint: '根据持仓截图更新 index.html 中的港股和A股数据'
---

# Update Portfolio From Screenshots

## Purpose

Use this skill to turn monthly holdings screenshots into verified entries in `index.html`.

The goal is not only to transcribe numbers. The workflow must:

1. Extract the holdings data from the screenshot.
2. Map that data to the existing `data` array structure in `index.html`.
3. Decide whether the target month should be appended or updated in place.
4. Make the smallest safe edit.
5. Check that the page can still render the updated month correctly.

## Use This Skill When

- The user asks to update `index.html` based on one or more holdings screenshots.
- The task mentions 港股, A股, 游戏仓, 月度持仓, 仓位截图, 组合截图, or 现金仓位.
- The user wants to append a new month of holdings or revise an existing month.
- The screenshot contains both Hong Kong and A-share positions and the page data needs to stay structured.

## Project Facts

- The page data lives in the `data` array inside `index.html`.
- Each month is an object with `time` and `portfolios`.
- Each portfolio row typically uses:
  - `name`
  - `count` for share count, except cash rows
  - `latestValue`
  - `hk: true` for Hong Kong holdings
- Cash rows use `name: "现金"` and `latestValue`, with no `count`.
- Treat screenshot market values as the direct source for `latestValue` unless the user explicitly says otherwise.
- The UI reverses display order by inserting each rendered month at the top, so new months should normally be appended to the end of the `data` array instead of inserted at the beginning.
- A-share rows may appear either without `hk` or with `hk: false`. Preserve the local style of the month block you are editing instead of reformatting unrelated entries.

## Required Inputs

Before editing, identify these inputs from the user request and attachments:

- The screenshot or screenshots to read
- The target month in `YYYY-MM` format, if available
- Whether the user wants to append a new month or correct an existing month
- Whether the screenshot values can be copied directly as `latestValue`

If any of these remain unclear after reviewing the provided context, ask before editing.

## Workflow

### 1. Inspect Current Data Shape

Read the relevant part of `index.html` before making assumptions.

Check:

- Where the latest month currently sits in the `data` array
- The object literal style used around the target month
- Whether the target month already exists
- Whether there are naming patterns that must stay unchanged

Do not reformat the whole array.

### 2. Extract Screenshot Data Into a Structured Draft

Convert the screenshot into a temporary structured list before editing code.

For each non-cash row, capture:

- `name`
- market type: Hong Kong or A-share
- `count`
- `latestValue`

For cash, capture:

- `name: "现金"`
- `latestValue`

Do not silently guess unreadable numbers. If the screenshot is ambiguous, stop and ask.

### 3. Normalize to Repository Rules

Normalize the extracted data to the page schema:

- Use numeric values for `count` and `latestValue`, with no currency symbols or thousands separators in code.
- Use `hk: true` for Hong Kong holdings.
- For A-share rows, mirror the style already used in the target month block.
- Prefer the screenshot's original holding names when writing the target month. If an existing repository label is clearly the same asset but spelled differently, keep the screenshot wording unless the user asks for normalization.
- If the same issuer appears in different markets, do not merge rows just because the names are similar.
- Keep `现金` as a dedicated row with no `count`.

### 4. Decide Whether To Append Or Replace

Use this decision logic:

- If the target month already exists and the user is correcting that month, update that month in place.
- If the target month is newer than the current latest month, append a new month object to the end of `data`.
- If the user provides a new screenshot but does not explicitly say whether to append or overwrite, default to appending a new latest month.
- If the target month is missing but not clearly newer, ask whether to insert a historical month or revise another month.

Do not duplicate the same month unless the user explicitly asks for parallel versions.

### 5. Edit With Minimal Surface Area

Only change the necessary month block in `index.html`.

Do not change:

- CSS
- rendering logic
- unrelated months
- sorting logic inside the renderer

When possible, preserve the surrounding punctuation, quote style, and line structure of the edited block.

### 6. Run A Sanity Check After Editing

Verify these points after the edit:

- The target month still has exactly one `time` key and one `portfolios` array.
- Each holding row has the expected fields.
- Hong Kong rows are marked with `hk: true`.
- Cash has no `count` field.
- The month can be rendered by the existing script without introducing syntax errors.
- The total implied by the rows is plausible relative to the screenshot.

### 7. Report What Changed

Summarize the result for the user with:

- which month was updated
- whether the change appended or replaced a block
- which holdings were added, removed, or materially changed
- any assumptions or unresolved ambiguities

## Guardrails

- Prefer asking one precise follow-up question over inventing a number.
- Do not convert screenshots into a different schema than the existing page uses.
- Do not drop rows because they seem small or repetitive.
- Do not merge holdings solely by issuer name.
- Do not reorder unrelated months.

## Completion Checklist

Consider the task complete only when all of the following are true:

- Screenshot data has been transcribed into the target month structure.
- The month placement in `data` is correct.
- Hong Kong and A-share rows are correctly distinguished.
- Cash is preserved as a separate row.
- The edit is limited to the intended month block.
- Any remaining ambiguity has been explicitly reported.

## Example Prompts

- 根据这张 2026-03 持仓截图，把港股和 A 股数据补到 index.html 里。
- 用附件里的组合截图更新最新一期游戏仓数据，先核对是否需要新增月份。
- 这次是修正 2026-02，不是新增月份，请按截图覆盖原有港股和 A 股持仓。