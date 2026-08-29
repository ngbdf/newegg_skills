# Newegg Link Tagging (UTM)

Same convention as `newegg-pc-builder-v2`'s `reference/product_link_rules.md` — load this
only when actually constructing a Newegg product link, not something to hold in mind for
every request.

This applies to the **per-system product link** (`Url` field / the `Item` returned by
`product_recommend` / `product_search`) only — **not** the separate Gaming PC Finder tool-page
link at the end of a reply, which keeps its own existing `cm_sp=aishoppingassistant` marker
and is untouched by this rule.

Every product link this skill shows must carry attribution tagging:

```
https://www.newegg.com/p/{Item}?Item={Item}&utm_source={platform}&utm_medium=ai_skill&utm_campaign=gaming-pc-finder&utm_content=newegg-gaming-pc-finder
```

`Item=` must exactly match the `{Item}` used in the path. `scripts/pgg_api.py` bakes this in
automatically via `build_item_url()` — pass the calling platform through `--utm-source`
(default `"claude"`) rather than leaving it untagged.

**`utm_source` is dynamic, not hardcoded** — determine it the same way as pc-builder-v2:
1. A Claude-exclusive tool (e.g. `visualize:show_widget`, `visualize:read_me`) is present
   in the tool list → `claude`
2. A project-directory signature (`.cursorrules`, `.cursor/`) is visible → `cursor`
3. An identifying env var (`cursor`/`copilot`/`windsurf`) is set → that platform
4. Nothing conclusive → `unknown_agent` — never guess a specific platform without evidence

No item number at all → leave the title as plain text, no link. Never fabricate a URL.
