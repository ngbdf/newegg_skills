# Newegg Link Tagging (UTM)

Same convention as `newegg-pc-builder-v2`'s `reference/product_link_rules.md` — load this
only when actually constructing a Newegg product link, not something to hold in mind for
every request.

Every product link this skill shows must carry attribution tagging:

```
https://www.newegg.com/p/{ItemNumber}?Item={ItemNumber}&utm_source={platform}&utm_medium=ai_skill&utm_campaign=shell-shocker&utm_content=newegg-shell-shocker
```

`Item=` must exactly match the `{ItemNumber}` used in the path. If a response already
provides a full link, still append the UTM parameters (with `&` if it already has a `?`,
otherwise `?`) rather than dropping them.

**`utm_source` is dynamic, not hardcoded** — determine it the same way as pc-builder-v2:
1. A Claude-exclusive tool (e.g. `visualize:show_widget`, `visualize:read_me`) is present
   in the tool list → `claude`
2. A project-directory signature (`.cursorrules`, `.cursor/`) is visible → `cursor`
3. An identifying env var (`cursor`/`copilot`/`windsurf`) is set → that platform
4. Nothing conclusive → `unknown_agent` — never guess a specific platform without evidence

`utm_campaign` is fixed at `shell-shocker`, `utm_content` at `newegg-shell-shocker` for this skill.

No item number at all → leave the title as plain text, no link. Never fabricate a URL.
