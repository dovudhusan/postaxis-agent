---
name: postaxis
description: Schedule and publish social media posts with PostAxis (X, Threads, LinkedIn, Telegram, Facebook Pages, Instagram, TikTok, YouTube Shorts) through the PostAxis MCP tools, including image carousels and slides the agent designs itself. Use when the user asks to post, schedule, cross-post, plan a content calendar, or publish images or slides to their social accounts.
---

# PostAxis

PostAxis publishes and schedules posts to the user's connected social accounts. You work through the PostAxis MCP tools (`list_workspaces`, `list_accounts`, `get_platform_rules`, `render_images`, `request_media_upload`, `get_uploaded_media`, `upload_media_from_url`, `create_upload_link`, `create_post`, `create_posts_bulk`, `list_posts`, `get_post`, `update_post`, `delete_post`).

If those tools are not available, the PostAxis connector is not connected yet. Ask the user to add the MCP server `https://postaxis.io/api/mcp` (setup guides: https://postaxis.io/ai-agents) and sign in. Do not try to post any other way.

## Five rules

**1. Accounts first.** Call `list_workspaces`, then `list_accounts`. Posts go to account ids, never to platform names. If several accounts match ("my Telegram channel"), ask which one.

**2. Rules before copy.** Call `get_platform_rules` for every platform you will post to, before you write captions. Respect the limits it returns (characters, image counts, video requirements). Telegram captions drop to 1,024 characters when the post has media.

**3. Media goes through PostAxis, by one of these paths.** Every `media_ids` value must come back from a PostAxis tool:
- **Images you design yourself** (carousels, slides, quote cards, charts, announcements): `render_images`. Send each image as an SVG document; PostAxis renders the PNGs on its server. No upload is involved, so it works even when your code environment has no internet access.
- **A photo or video the user gave you** (an attachment in the chat, a file on their phone): `request_media_upload`. Show the user the returned link, ask them to drop the file there and tell you when it's done, then call `get_uploaded_media` for the media_ids. This works everywhere and needs no settings. Never paste a photo into an SVG as base64.
- **A public image or video URL:** `upload_media_from_url`.
- **A file your own code can reach** (local agents such as Claude Code or Cursor): `create_upload_link`, then run the exact `curl` command it returns. If that upload is blocked, switch to `request_media_upload`.

Never pass a local path or a third-party URL as a media id.

**4. Never report success you have not seen.** A post is scheduled only when `create_post` or `create_posts_bulk` returns it. If any upload or render failed, say so plainly and offer the next step; do not describe the post as published.

**5. Time is explicit.** Always pass a `timezone` (ask if unknown). `publish: "now"` posts immediately, so confirm with the user before using it. For more than a couple of posts, run `create_posts_bulk` with `validate_only: true` first and show the user the plan before creating anything.

## Designing images for render_images

Write one complete `<svg>` per image, with `width`, `height` and a matching `viewBox`.

Sizes:
- Instagram, Telegram and Facebook carousels: 1080×1350 (4:5)
- Square posts: 1080×1080
- X and LinkedIn: 1600×900 (16:9) or 1080×1350
- Keep each image under about 5.8 megapixels. The optional `width` argument rescales the whole batch.

Fonts (nothing else is installed):
- `Inter` 400/600/800: clean body text and UI-style labels
- `Plus Jakarta Sans` 500/700/800: bold geometric headlines
- `Playfair Display` 400, 400 italic, 700: editorial serif headlines
- `JetBrains Mono` 400/700: small caps-style labels, numbers, code
- `sans-serif`, `serif` and `monospace` map to Inter, Playfair Display and JetBrains Mono.

Emoji and symbols outside these fonts render as fallback or nothing, so draw icons with shapes instead. External `href` images are not loaded; embed a raster image as a `data:` URI if you must.

SVG does not wrap text. Break lines yourself with separate `<text>` elements or `<tspan x="…" dy="…">`, about 18–28 characters per line at headline sizes and 40–55 at body sizes.

For a carousel that looks designed rather than generated:
- One idea per slide, and one dominant element per slide.
- A consistent margin (80–100 px at 1080 wide) and a shared baseline grid across slides.
- Two typefaces at most, sizes on a clear scale (for example 32 / 44 / 64 / 96).
- One accent colour, used sparingly. Check contrast: light text on dark at least 4.5:1.
- A small, consistent label on every slide ("02 / 07", a series name) helps people swipe.
- Render all slides of a carousel in one `render_images` call and keep the returned order.

## Platform notes

- TikTok photo posts accept only JPEG or WebP. `render_images` produces PNG, so for TikTok use a JPEG through `create_upload_link`, or post a video.
- YouTube needs exactly one video (Shorts); images are not accepted.
- Instagram needs at least one image or a video; it cannot post text only.

## Example: a 7-slide carousel to a Telegram channel

1. `list_accounts` and pick the Telegram account the user means.
2. `get_platform_rules` with `telegram` (1–10 images, caption ≤ 1,024 with media).
3. Design 7 SVGs at 1080×1350 and call `render_images` once with all 7.
4. `create_post` with `account_ids`, `caption`, `media_ids` in slide order, and either `publish: "schedule"` with `scheduled_at` and `timezone`, or `publish: "now"` once the user has confirmed.
5. Report what was posted, with the post id. If anything failed, report that instead.
