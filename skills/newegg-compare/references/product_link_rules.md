# Newegg Link Tagging (UTM)

Same convention as `newegg-pc-builder-v2`'s `reference/product_link_rules.md`, adapted for
a link that already carries its own query string and isn't a single-item product page
(a compare/category/search page). Load this only when actually constructing such a link.

Append the UTM parameters with `&` (the link already has a `?`):

```
https://www.newegg.com/Product/Productcompare?compareall=true&CompareItemList=<ItemNumber1>%2C<ItemNumber2>%2C...&utm_source={platform}&utm_medium=ai_skill&utm_campaign=compare&utm_content=newegg-compare
```

**`utm_source` is dynamic, not hardcoded** — determine it the same way as pc-builder-v2:
1. A Claude-exclusive tool (e.g. `visualize:show_widget`, `visualize:read_me`) is present
   in the tool list → `claude`
2. A project-directory signature (`.cursorrules`, `.cursor/`) is visible → `cursor`
3. An identifying env var (`cursor`/`copilot`/`windsurf`) is set → that platform
4. Nothing conclusive → `unknown_agent` — never guess a specific platform without evidence

`utm_campaign` is fixed at `compare`, `utm_content` at `newegg-compare` for this skill.
