---
name: image-to-ui-skill
description: Use when the user asks for image2, image generation, image-to-UI, UI screenshot to code, design to code, clickable app demo, mobile prototype, iOS preview, high-fidelity UI recreation, production-ready UI architecture, multi-agent UI implementation, or generating bitmap assets for a UI. Split the reference into code-rendered UI and image2-generated assets, orchestrate specialist agents when available, build a clickable preview, and verify the output.
---

# Image To UI Skill

把 UI 参考图做成可点击 demo。少解释，多落地。

Turn UI references into clickable demos. Keep the explanation lean and make the output real.

## 必守原则 / Non-Negotiables

- 需要生图时，调用项目指定的 `image2`。如果当前环境没有可用入口，要说明缺口，不能把 CSS/SVG/占位图说成已生图。
- When image generation is needed, call the project-designated `image2`. If no channel is available, state the gap instead of presenting CSS/SVG/placeholders as generated images.
- `image2` 只负责复杂位图：照片、产品图、人物、插画、纹理、背景、地图、卡片缩略图、物体抠图。
- `image2` is for complex bitmap work: photos, products, people, illustrations, textures, backgrounds, maps, thumbnails, and object cutouts.
- 代码负责 UI：文字、按钮、状态栏、导航、表单、开关、价格、标签、普通 icon、播放器控件。
- Code owns UI: text, buttons, status bars, navigation, forms, toggles, prices, labels, common icons, and player controls.
- 生成图片里不要包含可读 UI 文案、logo、水印、状态栏、按钮或小图标。
- Generated images must not contain readable UI text, logos, watermarks, status bars, buttons, or small UI icons.
- 最终 demo 必须可打开、可点击、可继续修改。
- The final demo must be openable, clickable, and editable.
- 开始工作前先询问用户交付形态：单文件 HTML 总览页还是交互式原型。默认交付单文件 HTML 总览页：一个 HTML 文件内含所有屏幕的实现，文件内结构为「总览网格（全部屏幕缩略图 + 名称）→ 点击缩略图切换到对应屏幕的完整视图」，屏幕间用轻量 JS/锚点切换；不要把各屏幕纵向堆叠成一个长页面，也不要每屏拆成多个文件。除非用户选择交互原型，否则不启动 dev server、不打开浏览器、不强行运行交互式原型。
- Before starting, ask the user which deliverable they want: a single-file HTML overview or an interactive prototype. Default is the single-file HTML overview: one HTML file contains the implementation of all screens, structured as an overview grid (thumbnails + names) that switches to the full view of the selected screen via lightweight JS/anchors. Do not stack screens vertically into one long page, and do not split into multiple files. Do not start a dev server, open a browser, or force an interactive prototype unless the user chose it.
- 以用户提供的参考图为准做 1:1 还原（布局、间距、字号、颜色、图标、比例、留白），不是风格近似；不确定的细节先按参考图原样做，不自由发挥。
- Match the user-provided reference 1:1 (layout, spacing, typography, colors, icons, ratios, whitespace) - not approximate. When in doubt, copy the reference exactly; do not improvise.

## Image2 通道 / Image2 Channels

优先按当前项目或会话的 `AGENTS.md` 执行；本仓库默认把系统 `imagegen` 视为原生 image2 入口。若系统入口不可用，再使用本仓库脚本 fallback。

Follow the current project or session `AGENTS.md` first. In this repo, system `imagegen` counts as the native image2 path. If it is unavailable, use the local fallback wrapper.

通道适配规则（Channel adaptation rule）：
- 有默认生图通道（系统 imagegen / 项目指定入口 / fallback 脚本且 API 可用）→ 直接走默认通道。
- 没有可用通道 → 默认不使用生图：所有视觉用代码/CSS/SVG 表达，明确标注「未生成位图」，不调用未指定的 API，不报错卡住流程，不强行安装或配置。
- Use the default generation channel when one is available. When none is available, skip image generation by default: express all visuals in code/CSS/SVG, state clearly that no bitmap was generated, and never block the delivery on image generation.

- Native image2 sources:
  - `source=system-imagegen`
  - `source=openai-imagegen-cli`
  - `image_gen` built-in tool surface, when available
- Project fallback:
  - `scripts/image2_asset.py`
  - `source=project-image2`
- Other supported fallback labels:
  - `youtoken-gpt-image-2`
  - `openrouter-icu-gpt-image-2`

不要在回复或日志中输出完整密钥。若使用 `OPENAI_API_KEY` 或其它凭证，只说明变量名和通道，不泄露值。

Never print full credentials in replies or logs. If `OPENAI_API_KEY` or another credential is used, report the variable/channel only, not the value.

Diagnostic command:

```bash
image2-ui doctor
```

## 工作流 / Workflow

0. 交互式开头：先询问用户交付形态——单文件 HTML 总览页（默认）还是交互式原型，确认后再开始。
1. 拆出两张清单：`code-ui` 和 `image2-assets`。
2. 为每个 `image2-assets` 写清楚用途、尺寸、风格、裁切方式和负面约束。
3. 调用 `image2` 生成真实图片文件，并放进项目资源目录。
4. 用项目现有技术实现页面，把生成资产接回界面。
5. 如果是 App 或手机参考图，做手机外框、状态栏安全区、Dynamic Island 或等价设备 chrome；单文件总览页内用锚点/JS 做轻量页面切换，只有用户选择交互原型才做多页面路由和完整交互。
6. 交付物是单文件 HTML 总览页：所有屏幕的实现代码在一个 HTML 文件里，文件内做「总览网格（缩略图 + 名称）→ 点击切换到单屏完整视图」，同一时刻只显示一个屏幕（或总览网格），不纵向堆叠成长页面；预览检查按需进行，不强制打开浏览器。

English version:

0. Interactive kickoff: first ask the user for the deliverable form - single-file HTML overview (default) or interactive prototype. Confirm before starting.
1. Produce two inventories: `code-ui` and `image2-assets`.
2. For each `image2-assets` item, record purpose, size, style, crop strategy, and negative constraints.
3. Call `image2` to generate real image files and place them in the project asset directory.
4. Build the page with the existing project stack and wire the generated assets into the UI.
5. For app or mobile references, include the device frame, safe area, Dynamic Island or equivalent device chrome. Use lightweight in-file anchors/JS for screen switching inside the single overview file; build a routed interactive prototype only when the user chose that form.
6. Deliverable is a single-file HTML overview: the implementation of all screens lives in one HTML file, structured as an overview grid (thumbnails + names) that switches to the full view of the selected screen. Only one screen (or the grid) is visible at a time; do not stack screens vertically into a long page. Preview checks are on demand, not mandatory.

## Multi-Agent Orchestration

When subagents or multi-agent tools are available, use the orchestration contract in `references/multi-agent-orchestration.md`.

The lead agent owns the user request, repository architecture, task decomposition, merge decisions, and final report. Specialist agents must return structured artifacts and must not silently redefine the product scope.

Recommended roles:

- `visual-analyst`: inspect references, identify visual hierarchy, and split `code-ui` from `image2-assets`.
- `asset-engineer`: create or verify the asset manifest, prompt records, formats, paths, alt text, and provenance.
- `ui-architect`: define routes, feature boundaries, component APIs, design tokens, state models, and i18n structure.
- `backend-contract`: define API contracts, request/response schemas, error envelopes, permissions, and mock data boundaries.
- `state-machine`: define async, device, form, retry, offline, optimistic-update, and rollback states before implementation.
- `ui-implementer`: implement the UI in the existing project conventions.
- `accessibility`: audit keyboard flow, focus management, accessible names, ARIA, contrast, reduced motion, and screen-reader semantics.
- `qa-auditor`: run build, typecheck, lint, browser, visual regression, and production-readiness checks.
- `release`: run the final checks, summarize changes, confirm artifacts, record execution mode, and prepare the commit or PR handoff.

Run `visual-analyst` and `asset-engineer` in parallel when their outputs are independent. Run `ui-architect`, `backend-contract`, and `state-machine` after repository discovery and before implementation. Run `accessibility` and `qa-auditor` after implementation. Run `release` last. Keep implementation and integration under the lead agent unless a specialist has an explicit, non-overlapping write scope.

If multi-agent execution is unavailable, execute the same roles sequentially in one context and preserve the same artifact names and handoff format.

## Reference Files

Read only the relevant reference files for the current task:

- `references/image2-entrypoint.md`: image2 entrypoint discovery and reporting.
- `references/asset-manifest-and-prompts.md`: asset inventory, prompt templates, and page output audit loop.
- `references/icon-system.md`: icon system, UI Glyph lock rule, and approved icon libraries.
- `references/loop-engineering.md`: iterative verification loop.
- `references/museum-app-case-study.md`: museum/mobile multi-screen case.
- `references/fashion-shopping-app-case-study.md`: fashion shopping visual asset case.
- `references/hicolor-case-study.md`: content graphic case.
- `references/multi-agent-orchestration.md`: reusable multi-agent roles, handoff contracts, and fallback behavior.

## Design, Icons, And Layout

- 图标统一用一套代码图标系统。
- Use one code-rendered icon system.
- 返回、关闭、菜单、搜索、设置、状态栏、电量/Wi-Fi/信号、播放、底部 tab、开关、加减号都用代码。
- Back, close, menu, search, settings, status bar, battery/Wi-Fi/signal, playback, bottom tabs, toggles, plus, and minus are code-rendered.
- 设备外观、产品抠图、商品图可以用 `image2`。
- Device appearances, product-cutout assets, and product images can use `image2`.
- 按角色而不是名称分类：`camera`、`lamp`、`speaker` 等在按钮里是 `code-icon`，在商品格或设备卡主视觉里可以是 `product-cutout`、`object-thumbnail` 或 `device-product-image`。
- Classify by role, not name: `camera`, `lamp`, and `speaker` are `code-icon` in controls, but can be `product-cutout`, `object-thumbnail`, or `device-product-image` in product/device visuals.
- 文字必须是真实文本，不能烘焙进图片里。
- Text must be real text, not baked into images.
- 明显控件必须能点击或有明确反馈。
- Visible controls must be clickable or provide clear feedback.
- 移动端不能横向滚动，文字不能溢出按钮或卡片。
- Mobile layouts must avoid horizontal scrolling, and text must not overflow buttons or cards.
- Avoid repeated `icon + heading + paragraph` card grids unless the reference clearly uses that pattern.
- Touch targets should be at least `44x44px`.
- Headings can use `text-wrap: balance` for better wrapping.

Approved icon libraries:

- `@phosphor-icons/react`
- `hugeicons-react`
- `@radix-ui/react-icons`
- `@tabler/icons-react`

Application code should use a single icon entry such as `UiIcon`, `IconRegistry`, or an SVG sprite. Keep an icon coverage table before delivery.

## UI Glyph Lock Rule

`image2` prompts must explicitly exclude UI glyphs:

```text
no icons, no UI symbols, no readable text, no logo, no watermark,
no status bar, no battery/Wi-Fi/signal glyphs, no arrows, no gear,
no menu dots, no plus/minus, no power symbol, no playback controls,
no tab icons, no toggles, no status dots
```

## Page Output Audit Loop

After generation and integration, prefer:

```bash
image2-ui validate <demo-dir> --reference <reference-image>
```

For iterative checks:

```bash
image2-ui loop <demo-dir> --reference <reference-image> --build "<build-command>"
```

This repo also includes `ui_output_audit.mjs` to catch broken assets, remote assets, low contrast, text overflow, mixed icon libraries, `generated-ui-glyph-asset`, `image-icon-in-control`, `cutout-asset-missing-alt`, and related issues.

## 最终汇报 / Final Report

- 预览入口或本地 URL（含所有页面总览单文件的路径）。
- Preview entry or local URL.
- `image2` 生成资产路径。
- Paths to generated `image2` assets.
- 实际通道，例如 `native-image2 source=system-imagegen`、`source=project-image2`、`youtoken-gpt-image-2` 或 `openrouter-icu-gpt-image-2`。
- Actual channel, such as `native-image2 source=system-imagegen`, `source=project-image2`, `youtoken-gpt-image-2`, or `openrouter-icu-gpt-image-2`.
- 哪些 UI 是代码实现。
- Which UI surfaces are code-rendered.
- 做过哪些检查。
- Which checks were run.
