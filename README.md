# Mudrex Trade Ideas — Plotline Widgets

Standalone HTML/CSS/JS cards for displaying Mudrex trade ideas inside Plotline (or any host page). Each file is self-contained and renders a single trade idea from [Project TIA](https://project-tia-production.up.railway.app/docs/api) JSON.

## Files

| File | Theme | Global API |
| --- | --- | --- |
| `trade-idea-widget-light.html` | Light mode | `TradeIdeaWidget` |
| `trade-idea-widget-dark.html` | Dark mode | `TradeIdeaWidgetDark` |

**Assets**

- `assets/placeholder-logo.png` — fallback coin logo when `logo_url` is missing or fails to load

## Preview locally

Open either HTML file in a browser. Each file includes a built-in demo grid showing all plotline states (Awaiting, Entry hit, TP1 hit, TP2 hit, SL hit, plus a Short example).

```bash
# optional: serve from repo root if you prefer http:// URLs
python3 -m http.server 8765
```

Then open:

- http://localhost:8765/trade-idea-widget-light.html
- http://localhost:8765/trade-idea-widget-dark.html

## Integration

The widget is **render-only**. The host app fetches trade ideas from TIA and calls `render()` whenever data changes.

### Light mode

```javascript
TradeIdeaWidget.render(containerElement, tradeIdeaObject);
```

### Dark mode

```javascript
TradeIdeaWidgetDark.render(containerElement, tradeIdeaObject);
```

**Update:** call `render()` again when TIA sends new data (poll or webhook).

**Remove:** clear the container when an idea leaves `GET /v1/trade-ideas/active`.

Each render replaces the container contents. The root card exposes:

- `data-idea-id` — trade idea id
- `data-trade-status` — derived widget status (`awaiting`, `active`, `tp1_hit`, `tp2_hit`, `sl_hit`)

### Exposed helpers

Both widgets also export formatting and status helpers on the global object, for example:

- `deriveTradeStatus`, `deriveMilestoneStates`, `deriveRiskLevel`
- `formatSymbol`, `formatPrice`, `formatProfitPct`, `formatRiskRewardRatio`, `formatValidity`
- `TRADE_STATUS`, `RISK_LEVELS`

## TIA fields used

| Field | Usage |
| --- | --- |
| `symbol` | Pair name (e.g. `BTCUSDT` → `BTC • USDT`) |
| `direction`, `leverage` | Long/Short badge |
| `entry_price`, `tp1_price`, `tp2_price`, `stop_loss` | Plotline milestones |
| `est_potential_pct` | Footer — Estd Profit |
| `risk_reward_ratio` | Footer — Risk/Reward |
| `expiry_at` | Validity line |
| `confidence` | Risk pill (HIGH → Low risk, MEDIUM → Medium risk, LOW → High risk) |
| `logo_url` | Coin logo |
| `mudrex_deeplink` | CTA button link |
| `entry_hit_at`, `tp1_hit_at`, `tp2_hit_at`, `sl_hit_at` | Plotline state |

## Status mapping

Status is derived from hit timestamps, not from `status` alone:

| Widget status | Condition |
| --- | --- |
| Awaiting | `entry_hit_at` is null |
| Entry hit | `entry_hit_at` set, TP1 not hit |
| TP1 Hit | `tp1_hit_at` set (wins over a later SL) |
| TP2 Hit | `tp2_hit_at` set or `status === TP2_HIT` |
| SL Hit | SL hit before TP1 only |

If TP1 was hit and SL hits later, the card stays on **TP1 Hit** — there is no separate “SL after TP1” state.

Expired and withdrawn ideas are not rendered; remove them from the feed when they leave the active list.

## Customisation

- **Colours / typography** — edit CSS in the `<style>` block of the relevant HTML file
- **Labels / number formatting** — edit `TRADE_STATUS`, `RISK_LEVELS`, and `format*` helpers in the script block
- **Placeholder logo** — replace `assets/placeholder-logo.png` or change `PLACEHOLDER_LOGO_SRC` in the script

## Plotline notes

- HTML + CSS + JS live in a single file per theme — suitable for Plotline embeds
- No built-in fetch, carousel, or expired/withdrawn UI
- Validity uses the browser locale for date/time formatting
