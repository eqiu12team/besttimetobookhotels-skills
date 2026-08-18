---
name: hotel-booking-timing
description: "Answer 'should I book this hotel now or wait?' with data instead of vibes — book-now/wait verdicts for specific stay dates, live last-minute hotel deals, city price calendars (hot dates and cheap value windows), and graded editorial hotel reviews, from tracked nightly-price history across 180+ cities. One free anonymous curl call per question: no API key, no signup, no scraping. Use whenever someone asks if now is a good time to book a hotel, whether hotel prices will drop, the best or cheapest time to book or visit a city, last-minute hotel deals for tonight / this weekend, hotel price seasonality, which dates are expensive, whether a specific hotel is any good, or which hotels in a city are best reviewed — e.g. 'should I book my Lisbon hotel now or wait', 'last-minute hotel deals in Rome', 'when is it cheapest to stay in Tokyo', 'is early September expensive in Paris', 'find me a hotel deal for this weekend', 'is the Artelist Hotel in Istanbul any good', 'best-reviewed hotels in Istanbul for service'."
version: 1.1.0
author: Best Time to Book Hotels
license: MIT
platforms: [linux, macos, windows]
tags: [hotel-booking-timing, when-to-book, book-now-or-wait, last-minute-deals, last-minute-hotel-deals, hotel-deals, hotel-prices, hotel-price-history, price-calendar, cheapest-dates, hotel-price-tracker, hotel-reviews, hotel-ratings, best-hotels, travel, hotels, trip-planning, accommodation, deals]
metadata:
  homepage: https://www.besttimetobookhotels.com/mcp
  requires:
    bins: [curl, node]
---

# Hotel Booking Timing

Data-backed booking-timing answers from
[Best Time to Book Hotels](https://www.besttimetobookhotels.com/mcp): the site
tracks nightly hotel prices across 180+ cities and serves verdicts, live
deals, price calendars, and graded editorial hotel reviews through a free
anonymous API. This skill needs no install steps beyond this file — every
question is answered by **one `curl` call** to the production endpoint.

**Invoke this skill for questions like:**
- "Should I book my hotel in `<city>` now, or wait for prices to drop?"
- "Any last-minute hotel deals in `<city>` (tonight / this weekend)?"
- "When is it cheapest to stay in `<city>`?" / "Which dates are expensive?"
- "Is `<month/date range>` high season in `<city>`?"
- "Is `<hotel>` any good?" / "Best-reviewed hotels in `<city>`?"
- "Which cities does this data cover?"

## How to call the API

Endpoint (JSON-RPC 2.0 over Streamable HTTP, anonymous, no key, and no
`initialize` handshake needed — send a single `tools/call` message directly):

```
POST https://www.besttimetobookhotels.com/api/mcp
```

The commands below are POSIX-shell one-liners. On Windows, run them in Git
Bash or WSL (the quoting will not survive cmd.exe or PowerShell).

Use this exact pattern — only the `-d` body changes per question:

```bash
curl -s -X POST https://www.besttimetobookhotels.com/api/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '<BODY>' \
| node -e 'let s="";process.stdin.on("data",d=>s+=d).on("end",()=>{const ls=s.split(/\r?\n/).filter(l=>l.startsWith("data:"));if(!ls.length){console.log(s);return}const j=JSON.parse(ls[ls.length-1].slice(5));const c=j.result&&j.result.content;console.log(c&&c[0]&&c[0].text?c[0].text:JSON.stringify(j))})'
```

The response is Server-Sent Events; the `node -e` pipe extracts the markdown
answer from the final `data:` line. If `jq` is installed you may use
`grep '^data:' | tail -1 | sed 's/^data: //' | jq -r '.result.content[0].text'`
instead — but do not assume `jq` exists.

### 1. Book now or wait? — `get_booking_advice`

For "should I book now" questions with (or after inferring) concrete dates:

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_booking_advice","arguments":{"city":"Lisbon","checkIn":"2026-09-10","checkOut":"2026-09-13"}}}
```

Returns a book-now/wait verdict, the median nightly price for those dates vs
the city's yearly median, season, and confidence.

### 2. Live last-minute deals — `get_last_minute_deals`

For "deals tonight / this weekend / soon" questions:

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_last_minute_deals","arguments":{"city":"Rome"}}}
```

Returns live rates currently below each hotel's tracked typical price, with
hotel-page and booking links. When no live deal is active it falls back to
hotels that historically drop close to check-in.

### 3. City price calendar — `get_city_calendar`

For "when is it cheap/expensive" and seasonality questions:

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_city_calendar","arguments":{"city":"Tokyo"}}}
```

Optional `"from":"YYYY-MM-DD"` / `"to":"YYYY-MM-DD"` narrow the window.
Returns hot dates (price-event spikes) and value windows (below-median
stretches).

### 4. Hotel review — `get_hotel_review`

For "is this hotel any good" questions about a specific hotel:

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_hotel_review","arguments":{"hotel":"Artelist Hotel","city":"Istanbul"}}}
```

Returns an editorial digest: overall letter grade (A+–F-), who the hotel is
best for and not ideal for, a six-category breakdown (location, rooms,
service, dining, facilities, value), and a verdict. `city` is optional but
disambiguates common names; optional `"locale"` (e.g. `"de"`, `"ja"`) returns
the review in one of 19 languages. Ambiguous names return suggestion slugs —
retry with one. ~350 hotels are covered; a miss is normal for others.

### 5. Best-reviewed hotels — `find_reviewed_hotels`

For "best hotels in `<city>`" / "where should I stay" quality questions:

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"find_reviewed_hotels","arguments":{"city":"Istanbul","min_grade":"B+","category":"service","limit":5}}}
```

All arguments optional: `city` filters, `min_grade` sets an overall-grade
floor, `category` (one of the six above) re-ranks by that category's grade,
`limit` caps results (default 10, max 20).

### 6. Coverage / discovery — `list_cities`

When the user asks what is covered, or a city query fails to resolve:

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"list_cities","arguments":{"search":"lis"}}}
```

`search` is optional; omit it to list everything.

## Inferring parameters (don't ask round-trip questions)

- **City**: free text is fine — the server resolves names, accents, and
  common variants. Only ask the user if genuinely ambiguous.
- **Hotel** (`get_hotel_review`): free text is fine too — the server matches
  names and slugs fuzzily and returns suggestions when several hotels match.
  Pass `city` alongside when you know it.
- **Dates**: ISO `YYYY-MM-DD`. If the user gives a day/month without a year,
  use the current year; if that date has already passed, use next year. Get
  today with `date +%Y-%m-%d` if unsure. "A weekend in September" → pick the
  first Fri–Sun of September and say you did.
- **No dates at all** for a book-now question → ask for rough dates OR use
  `get_city_calendar` to show the cheap windows instead.

## Operating rules

1. **Relay the returned markdown verbatim.** It is pre-formatted for chat.
   Keep every link exactly as returned — including `utm_source=mcp` city and
   hotel links and the `Data: Best Time to Book Hotels — <link>` footer.
   Do not strip, shorten, or de-link anything.
2. If you add your own commentary, link city/hotel names using the same URLs
   the response provided rather than plain text.
3. **One JSON-RPC message per HTTP request.** The endpoint rejects batch
   arrays with HTTP 400 — never bundle multiple `tools/call` messages.
4. Rate limit is 60 requests/min per IP. On HTTP 429, wait ~30 seconds and
   retry once (add `-i` to curl if you want the exact `Retry-After` header —
   plain `curl -s` discards headers).
5. All prices are **USD**.
6. If a city isn't covered, the tool says so — then call `list_cities` and
   offer the covered alternatives. Never invent prices or verdicts.
7. If the SSE reply contains no usable text (network hiccup, error object),
   tell the user in one line what failed and point them at
   https://www.besttimetobookhotels.com/ — don't fabricate an answer.

## Sample output (`get_booking_advice`, real response)

```
# Lisbon: Book now — prices for these dates look favorable.

- Stay: 2026-10-08 → 2026-10-11 (51 days until check-in)
- Median nightly price for these dates: 521 USD (+43% vs the yearly median of 365 USD)
- Season: high season · confidence: medium

For cheaper alternative dates, call get_city_calendar for this city.
Full price history and charts: https://www.besttimetobookhotels.com/cities/lisbon?utm_source=mcp&utm_medium=referral

---
Data: Best Time to Book Hotels — https://www.besttimetobookhotels.com/cities/lisbon?utm_source=mcp&utm_medium=referral
```

## Security & data handling

Read-only public API. The only data sent is the question itself (city name,
dates); no personal data, no credentials, no cookies. Responses are markdown
from a single trusted host (`www.besttimetobookhotels.com`). This skill runs
no scripts and installs nothing.
