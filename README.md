# Best Time to Book Hotels — Agent Skills

**Should you book that hotel now, or wait?** Agent skills that answer with
data, not vibes — book-now/wait verdicts, live last-minute deals, and city
price calendars from tracked nightly-price history across 180+ cities.

> ```bash
> npx skills add https://github.com/eqiu12team/besttimetobookhotels-skills --skill hotel-booking-timing
> ```

No API key, no signup, no bundled scripts — the skill makes one `curl` call
per question to the free public API at
[besttimetobookhotels.com](https://www.besttimetobookhotels.com/?utm_source=skills&utm_medium=readme).

**Sample answer:**

```
# Lisbon: Book now — prices for these dates look favorable.

- Stay: 2026-09-10 → 2026-09-13 (52 days until check-in)
- Median nightly price for these dates: 522 USD (+35% vs the yearly median of 386 USD)
- Season: high season · confidence: medium
```

## Skills

| Skill | What it answers |
|-------|-----------------|
| [`hotel-booking-timing`](skills/hotel-booking-timing/SKILL.md) | "Book now or wait?" verdicts for specific dates · live last-minute hotel deals · city price calendars (hot dates & value windows) · which cities are covered |

## Under the hood

The skill is instructions-only: it drives the public
[MCP endpoint](https://www.besttimetobookhotels.com/mcp) of Best Time to Book
Hotels (`https://www.besttimetobookhotels.com/api/mcp`, Streamable HTTP,
anonymous). If your agent speaks MCP natively you can also add the server
directly — it is listed in the official MCP registry as
`com.besttimetobookhotels/booking-timing`. The skill exists so agents without
MCP configuration (or users who prefer zero-config installs) get the same
data with plain `curl`.

Prices are USD. Data comes from the site's tracked nightly-price history and
live rate checks; every answer links back to the source city/hotel pages.

## License

[MIT](LICENSE)
