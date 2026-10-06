# PostAxis for AI agents

Schedule and publish social media posts from Claude, Cursor, Gemini CLI, Grok and other agents with [PostAxis](https://postaxis.io): X, Threads, LinkedIn, Telegram, Facebook Pages, Instagram, TikTok and YouTube Shorts.

This bundle contains:

- **The hosted PostAxis MCP server** (`https://postaxis.io/api/mcp`). You sign in to PostAxis on first use; no token or local install needed.
- **The `postaxis` skill**, which teaches the agent the safe workflow: accounts first, platform rules before copy, media through PostAxis, and no success claims without a confirmed post. It also includes a short design guide for carousels and slides.

## Images without uploads

Agents that design their own images (carousels, quote cards, slides) send them to the `render_images` tool as SVG. PostAxis renders the PNGs on its server, so it works even in sandboxes with no internet access, such as Claude's code execution or ChatGPT. Photos and videos the user already has go through `request_media_upload`: the agent gives the user a postaxis.io link, they drop the file there, and the agent picks it up with `get_uploaded_media`. No network settings needed. Local agents can also upload directly with `create_upload_link`.

## Install

- **Claude Code / Cowork:** run `/plugin marketplace add dovudhusan/postaxis-agent`, then `/plugin install postaxis@postaxis`.
- **Claude (web, desktop, mobile):** add the connector `https://postaxis.io/api/mcp` under Settings → Connectors. For the skill, upload `postaxis-skill.zip` under Settings → Capabilities → Skills.
- **Cursor:** use `.cursor-plugin`, or add the MCP URL in Cursor's MCP settings.
- **Gemini CLI:** `gemini-extension.json` declares the hosted server.
- **Grok:** `.grok-plugin` declares the skill and the hosted server.

Step-by-step guides for 30+ agents: https://postaxis.io/ai-agents
