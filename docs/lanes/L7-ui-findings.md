# L7 UI findings (2026-11-24/25)

Walked locally: Hub (built from origin/develop, local PostgreSQL, throwaway organizer, 6 seeded sticks, 2 recordings,
Keeper OS images faked through a local proxy because real images need Azure storage) and website (vite dev).
Widths 1280 / 768 / 375, Classic + Modern, light + dark (OS scheme). Hub is a single-page app: routes `/`
(login, create account, forgot password, sticks, stick detail, People, Platform), `/download/image`, `/forge/connect`.

## ck-hub (fixed in the Hub PR unless marked)
| Page | Problem | Status |
|---|---|---|
| Header, all pages | A long organization name makes the org `<select>` as wide as the name: at 1280 the account block wraps to a 2nd row; at 375 the page grows to ~675px wide and the select and **Log out** are off-screen (horizontal page scroll). | fixed (select capped, account block wraps, full width under 640px) |
| Sticks, mobile | Four summary tiles stacked full width (about 450px before any content). | fixed (2x2 under 640px) |
| Sticks, tablet | Tiles wrap 3+1 (orphan "Need a look"). | fixed (min tile 160px) |
| Sticks table | Headers "Logged in", "Last seen" wrap onto two lines; at 375 the Status column is off-screen on first view. | fixed (nowrap headers, tighter table on mobile) |
| Stick detail (tablet/mobile) | The detail row is as wide as the table, so its text (archive hint, facts) is cut off at the screen edge and needs sideways scrolling. | fixed (content sticks to the visible area) |
| Stick with empty team | Team cell is blank. | fixed (shows "Stick <id>") |
| Login / create account / forgot | A stray "·" shows before "Back to log in"; auth card margins too large on mobile. | fixed |
| Keeper OS downloads | Header problem above; links are 18-23px tall tap targets; SHA-256 not set in a monospace font. | fixed (padding, mono) |
| Forge connect | Classic theme: card has the modern flat border, no bevel; mobile margins. | fixed |
| People / Platform forms, mobile | Inputs keep 180px min width and wrap unevenly. | fixed (full-width fields under 640px) |
| 404 (`/nope`, `/download/forge` when unset) | Empty body, no page at all. | fixed in a follow-up PR (HTML 404 page in both themes; `/api/*` and non-GET keep the bare 404) |
| Login page, dark | OK. Classic and Modern both OK at 375/768/1280 for auth forms. | n/a |

## ck-website
| Part | Problem | Status |
|---|---|---|
| Landing (all widths, New/Old, light/dark) | No overflow, hero and download area fine. | none |
| Licences panel | Old look + dark scheme: panel border (#0a0a0a on #1B1F22) nearly invisible; summary "Licences" only ~18px tall tap target. | fixed |
| Licences panel | Overlays the hero; no Escape / click-outside to close. | fixed in a follow-up PR (Escape and outside click close it, focus returns to the summary, aria-expanded kept) |

Nothing known remains. Re-checked 404 page at 375 (modern, light), 768 (classic, dark); Licences close behaviour checked by events.
