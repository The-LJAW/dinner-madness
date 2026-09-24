# Dinner Madness

Can't decide where to eat? Dinner Madness pulls the restaurants near you, seeds them into a tournament-style bracket, and lets everyone at the table vote from their own phone, round by round, until one place is left.

Everything lives in one file, `index.html`. It has no backend, and players need no accounts. The only key inside is PostHog's public project key, which can send usage events but can't read any data.

## How it plays

1. **Where are you?** The app asks for your location. If you say no, or the phone can't find you, type an address, city, or ZIP instead.
2. **How far will you drive?** Pick a 5, 10, 15, 20, or 30-minute drive (15 is the default). Each restaurant also shows its estimated drive time.
3. **Who's eligible?** Choose whether to include fast food, skip the big chains, or include coffee, dessert, and snack shops.
4. **Seed the field.** There are two modes:
   - **Cuisine first** (the default). Cuisines face off first, like Mexican vs. Chinese and Pizza vs. BBQ. The winning cuisine then gets its own bracket of restaurants.
   - **Restaurants only.** Specific restaurants face off, from any cuisine or just one.

   You choose 2–16 contenders. Cuisines are seeded by how many nearby spots they have; restaurants are seeded by distance, or shuffled.
5. **Share it.** A QR code and link open the bracket on anyone's phone. Friends type a first name and they're in.
6. **Vote.** Everyone picks a winner in every matchup of the round, then locks in. A round closes on its own once everyone has voted. The organizer can close a round early if someone wanders off.
   - Ties are settled by a coin flip that plays on every phone. The better seed takes heads (the Memento Vivere sun side) and the other seed takes tails. The result comes from the bracket's own data, so every phone sees the same flip.
   - Someone without a phone can be added under **Invite → Someone at the table without a phone?** They then vote on your phone through a "Voting as" switch.
7. **Winner.** The winning restaurant comes with a Directions link, an "Hours & reviews" link (Google Maps), and its website and phone number when those are on file.

## Put it online (GitHub Pages, about 3 minutes)

The share link and QR code only work for other people when the page is hosted online.

1. On GitHub, create a new **public** repository, for example `dinner-madness`.
2. Upload `index.html` (and this README) to the repository's main branch.
3. Go to **Settings → Pages**. Under "Build and deployment", set **Source: Deploy from a branch**, **Branch: main**, and folder **/ (root)**. Save.
4. After a minute the app is live at `https://<your-username>.github.io/dinner-madness/`.

GitHub Pages serves over HTTPS, and phones need HTTPS before they'll share their location. Opening the file directly from your computer works for trying it out, but friends can't join that way.

## What it runs on

| Job | Service | Notes |
|---|---|---|
| Restaurants and their cuisine | OpenStreetMap via the Overpass API (`overpass-api.de`, with `overpass.kumi.systems` as a backup) | Free, no key. A 15-minute drive around Westfield, IN (about 6 miles) returns about 180 places in a few seconds. |
| Address search and "near …" label | Nominatim (OpenStreetMap), with Photon as a backup | Free, no key. |
| Live voting between phones | ntfy.sh, a public message relay | Free, no account. Each bracket is a private-by-obscurity channel named `dinnermadness-<code>`. Messages expire after about 12 hours. |
| QR codes | qrcode-generator (MIT license), bundled into the page | Works offline. |

Every phone replays the same ordered list of messages (joins, votes, round closes), so every phone works out the same bracket on its own. Nobody's phone has to stay open, including the organizer's.

## Usage analytics (PostHog)

The app sends a small set of named, anonymous events to PostHog (US cloud). There's no autocapture, no screen recording, and no cookies. Each phone gets a random ID that's stored only on that phone.

| Stage | Events |
|---|---|
| Arrive | `app_opened` (with `via`: `qr`, `link`, `share`, or `direct`) |
| Set up | `location_set`, `location_failed`, `restaurant_search` (ok, places, attempts, seconds) |
| Create and invite | `bracket_created`, `invite_shared`, `join_viewed`, `guest_joined`, `proxy_added`, `bracket_missing` |
| Vote | `voting_started`, `picks_locked`, `round_closed` (auto or organizer), `cuisine_decided`, `stage2_started`, `stage2_skipped` |
| Finish and act | `bracket_finished` (winner, runner-up, minutes, players), `winner_action` (directions, hours and reviews, website, call), `start_own_clicked`, `new_bracket_clicked` |
| Problems | `error` (voting server or bracket creation) |

Round and bracket results are sent only from the organizer's phone, so each one is counted once. Voter names, typed addresses, and exact coordinates are never sent; the location is city-level only.

To turn analytics off, set `PH_KEY` to an empty string in `index.html`.

## Good to know

- **Links expire.** A bracket lasts about 12 hours, which covers one dinner decision. After that the link shows a "left the building" page.
- **Privacy.** Anyone with the link or code can join and vote. The shared bracket data includes only the city name and a location rounded to about 1 km. The exact address never leaves the organizer's phone.
- **Data quality.** OpenStreetMap coverage is good in US metros but not perfect. Places with no cuisine tag are sorted by keywords in their names (for example "Taqueria…" goes to Mexican and "…Ale House" goes to American & Pub). Whatever still can't be sorted goes into a **Wildcard** category. Hours and ratings aren't included, so the winner card links to Google Maps for those.
- **Drive time.** Drive times are estimated from straight-line distance at about 2.5 minutes per mile, which fits suburban roads with lights: 15 minutes searches about 6 miles. There's no live traffic. To retune it, change `MIN_PER_MILE` in `index.html`.
- **Your own relay.** If you ever outgrow the free public relay, you can run your own ntfy server. Set `window.DM_NTFY_BASE = 'https://your-ntfy-host'` in a `<script>` before the app script.
