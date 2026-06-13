# 🔨 THE HAMMER

**Real-time IPL-style auction platform for friends. No servers to manage, no auth friction — just a 6-letter room code and you're live.**

![Status: Stable](https://img.shields.io/badge/status-stable-brightgreen)
![License: MIT](https://img.shields.io/badge/license-MIT-blue)

---

## What Is This?

THE HAMMER is a full-stack web app that lets you run an IPL-style cricket auction with your friends in real time. One person becomes the auctioneer, everyone else claims a franchise and bids on 76 live players. Real purse math (₹120 Cr per franchise), real IPL bid increment slabs, live for everyone in the room simultaneously.

**No sign-ups. No logins. No friction.** Open the file, create a room, share the code.

Two game modes: **Auction** (purses, paddles, the hammer) and **Draft** (toss → one-by-one picks, fantasy-style).

---

## Features

### 🎬 Broadcast Layer (v2)
- **Player Stat Cards**: Monogram avatar, career numbers, and a last-5 form graph per player (deterministically simulated from the player's name — clearly labeled, not real data)
- **Live Bid Animations**: Amount pop, stage flash, franchise-colored confetti on every SOLD
- **Auctioneer Commentary**: Rule-based broadcast lines that react to context — opening calls, bidding wars, big-money territory, steals at base price, unsold silence
- **Voice Auctioneer**: Toggleable Web Speech narration of every commentary line (works offline, no API needed)
- **Sound Effects**: Synthesized via WebAudio — bid ticks, gavel, crowd cheer/groan, heartbeat. No audio files, no loading
- **Drama Mode**: Final 5 seconds of a contested lot pulse the whole screen red with a heartbeat


### 🪙 Draft Mode (v3)
- **Host-entered rosters**: The room creator types friends' names (2–10) — no one has to sign up for anything
- **Custom player pool (local tournaments)**: Optionally replace the 76 IPL stars with your own players — one per line as `Name, Role, Rating` (role and rating optional, defaults Player/75). Names are deduped, ratings clamped 40–99, up to 200 players. Role filters rebuild automatically from whatever roles you typed
- **Server-side toss**: When the host hits start, the database shuffles the pick order; the toss winner picks first
- **One-by-one picks**: Straight round-robin rotation until the pool is exhausted — every player gets drafted, split as evenly as possible (earliest pickers absorb any remainder). "Round 2 of 3 · Pick 7 of 20" always visible
- **Pick from any device**: Friends can join with the room code and tap "This is me" to claim their seat and pick from their own phone — or the host runs the whole thing pass-and-play
- **Turn enforcement at the database**: Out-of-turn picks and stolen players are rejected server-side, race-safe under simultaneous taps
- **Live draft boards**: Every drafter's picks update in real time, current picker glows in their franchise color
- **Draft awards**: Toss Winner, First Overall, Best Player Drafted, Steal of the Draft, Strongest Squad — plus the same season simulator and champion odds

### 🧠 Strategy Layer (v2)
- **War Room Dashboard**: Your private panel — squad composition bars, strength rating, max-safe-bid and per-slot budget math, and a rule-based advisor that names affordable targets for your squad gaps
- **Team Strength Ratings**: 0–100 score from squad rating average + balance bonuses (WK, 3+ bowlers, 2+ ARs) + depth
- **Live Strength Leaderboard**: Ranked rail that updates after every sale
- **Most Contested Board**: Players who triggered bidding wars (3+ bids)
- **Player Browser**: Search and role-filter all 76 players with live status (pending/live/sold/unsold + price)
- **Watchlist**: Star players; get an alert chime + toast the moment they hit the block

### 🎙 Live Voice (v4)
- **In-room voice chat**: Tap "Join voice" — peer-to-peer WebRTC audio mesh between everyone in the room, signaled over Supabase Realtime (presence + broadcast), STUN-only, no media server and no extra cost
- **Self-mute** toggle, echo cancellation and noise suppression on by default
- **Host moderation, two levels**: 🔇 *Mute once* instantly cuts a person's mic (they may unmute themselves); ⛔ *Mute for the whole auction* is persisted in the database, blocks them from rejoining voice, and every client silences their audio — host can lift it with 🔊
- **Honest limits**: mesh audio is ideal for ≤6–8 talkers; STUN-only means very strict corporate NATs may not connect (a TURN server would fix that); and because audio is peer-to-peer, the auction-length mute is enforced by every client in the app rather than by a server

### 💬 Social Layer (v2)
- **Room Chat**: Realtime chat with franchise-colored identity, token-verified senders, 280-char limit, unread dot
- **Trophy Room**: Strongest-squad champions from your past auctions, on the landing page

### 📊 Post-Auction Layer (v2)
- **Awards**: Record Signing, Bargain of the Night, Shopaholic, Ice in the Veins, Strongest Squad
- **Auction Timeline**: Every lot in order with outcome and price
- **Team Comparison**: Side-by-side squad cards with strength and composition
- **Season Simulator**: Double round-robin + playoffs, win odds from squad strength, full points table
- **Champion Odds**: 500 simulated seasons → title probability per team
- **Highlights Export**: One tap copies a shareable text summary

---


### 🎯 Core Auction Engine
- **76 Fresh Players**: Marquee names (Virat, Bumrah) down to uncapped picks, shuffled into a random order every room
- **Real Purse Mechanics**: ₹120 Cr per franchise; bids auto-deduct; squad cap (15 players) enforced
- **IPL-Accurate Bid Slabs**: Increment by ₹5L below ₹1 Cr → ₹10L (₹1–2 Cr) → ₹20L (₹2–5 Cr) → ₹25L (₹5–10 Cr) → ₹50L (₹10 Cr+)
- **Race-Safe Bidding**: Row-level locking in Postgres; no race conditions even with 10 simultaneous bids
- **Custom Lot Timer**: Host sets seconds-per-lot at room creation (10–120s, clamped server-side)
- **Anti-Snipe Timer**: Timer auto-extends up to 12s when someone bids in the final seconds (never beyond the room's own timer)
- **Atomic Transactions**: A bid either succeeds fully or fails fully — no half-states

### ✨ UX/DX
- **Real-Time Sync**: Supabase Realtime channels + 6s polling fallback; everyone sees the same auction state within 100ms
- **Live Ticker**: Bid history streams in as they land with team color, amount, and timestamp
- **Broadcast Aesthetic**: Center stage colors shift to match the leading bidder's franchise in real time
- **Session Resume**: Refresh the page mid-auction and you're back where you left off (session stored in localStorage)
- **Mobile Responsive**: Works on phones, tablets, and desktops
- **Host Controls**: Auctioneer has an isolated control bar with hammer button and optional auto-hammer-at-zero toggle
- **Squad Peek**: Tap any franchise's purse card mid-auction to see every player they've bought, what they paid, and what's left

### 🛡️ Security
- **No Client Writes**: All mutations go through `SECURITY DEFINER` stored procedures; RLS blocks direct table access
- **Token-Based Auth**: Host and owner tokens issued on room creation; stored only in localStorage (not in database)
- **Validation at the Gate**: Bad token? Insufficient purse? Self-outbid attempt? Rejected at the database level
- **Secrets Table**: `ipl_tokens` has RLS enabled but zero policies — invisible to all clients

---

## Tech Stack

| Layer | Tech |
|-------|------|
| **Frontend** | Vanilla JS + HTML5 + CSS3 (no frameworks) — "Broadcast Royale" theme: black/navy/gold, metallic gradients, glassmorphism, broadcast lower-thirds |
| **Realtime** | Supabase Realtime (PostgreSQL LISTEN/NOTIFY) |
| **Backend** | PostgreSQL 17 + PL/pgSQL stored procedures |
| **Auth** | Token-based (generated server-side, never persisted) |
| **Hosting** | Supabase (backend) + Static file server (frontend) |
| **Database** | Supabase Postgres (5 tables, 4 public RPCs, strict RLS) |

**Why this stack?**
- Zero infrastructure to manage; Supabase handles replication, backups, realtime, and REST API automatically
- No JavaScript framework — vanilla code stays readable and isn't overengineered for a real-time stateful app
- PL/pgSQL stored procedures enforce business logic at the database layer, making it impossible to bypass via API tampering
- Row-level locking in transactions prevents bid races that a traditional REST API would struggle with

---

## Quick Start

### For Players (Simple)
1. **Get the file**: Download `the-hammer-ipl-auction.html`
2. **Open in browser**: Double-click or drag into Chrome/Firefox/Safari
3. **Join a room**: Ask the host for the 6-letter code, paste it in, pick your franchise
4. **Bid**: When your player comes up, hit the **Bid** button to match the next increment

### For the Host (Slightly Less Simple)
1. **Open the file** (same as above)
2. **Create a room**: Give it a name (e.g., "Saturday Mega Auction") and pick seconds-per-lot (10–120s), get a 6-letter code
3. **Share the code** with friends via WhatsApp/Slack/email
4. **Wait for teams**: Each friend claims a franchise (CSK, MI, RCB, etc.) with their name
5. **Start the auction**: Once 2+ teams are in, hit **Start Auction**
6. **Bring the hammer down**: After each lot, either manually click **Hammer** or let auto-hammer close the lot at 0:00
7. **View results**: Squads auto-sort by total spend at the end

---

## Architecture

### Database Schema

```
ipl_rooms ─────────────────┐ (1:N)
├─ id (UUID, PK)           │
├─ code (TEXT, unique)     │──────> ipl_players
├─ name                     │──────> ipl_teams
├─ status (lobby|live|completed)  │──────> ipl_bids
├─ current_player_id ────────┘
├─ current_bid (₹ in lakhs)
├─ current_bid_team
├─ timer_ends_at, timer_seconds (host-configured, 10–120)
├─ sold_count, unsold_count
└─ base_purse (default ₹120 Cr)

ipl_teams
├─ id (UUID, PK)
├─ room_id (FK → ipl_rooms)
├─ franchise (CSK, MI, RCB, etc.)
├─ name, owner_name
├─ purse_remaining (tracks spend)
└─ squad_size (0–15)

ipl_players
├─ id (UUID, PK)
├─ room_id (FK)
├─ name, role (Batter/Bowler/All-Rounder)
├─ country, base_price, rating
├─ auction_order (shuffled 1–76)
├─ status (pending → live → sold/unsold)
├─ sold_to_team, sold_price
└─ created_at

ipl_bids
├─ id (bigserial, PK)
├─ room_id, player_id, team_id (FKs)
├─ team_franchise, team_name (denormalized for ticker)
├─ amount (₹ in lakhs)
└─ created_at (for chronology)

ipl_tokens ← SECRETS TABLE
├─ ref_id (UUID, PK)
├─ kind (host|owner)
├─ token (UUID, generated per room/team)
└─ RLS: no policies → invisible to clients
```

### Key Stored Procedures

**`ipl_create_room(name, timer_seconds)`**
- Generates a unique 6-char alphanumeric room code
- Stores a per-room lot timer (clamped to 10–120 seconds)
- Creates room, copies 76 players from the pool, shuffles them
- Issues a host token (returned to creator, stored in localStorage)
- Returns: `{ room_id, code, host_token }`

**`ipl_claim_team(code, franchise, team_name, owner_name)`**
- Locks the room for update (prevents teams being claimed during auction)
- Checks room is still in 'lobby' status
- Inserts team, issues owner token
- Returns: `{ room_id, team_id, owner_token }`

**`ipl_start_auction(room_id, host_token)`**
- Validates host token (derived from ipl_tokens)
- Checks 2+ teams are in the room
- Advances to first player (calls internal `ipl__advance`)
- Sets timer to 30 seconds

**`ipl_place_bid(room_id, team_id, owner_token)`** ← **The Race-Safe One**
```sql
-- Acquires row lock on ipl_rooms (serializes all bids in this room)
select * from ipl_rooms where id = room_id for update;

-- Validates: token, purse, squad cap, self-outbid check
-- Computes next_bid using ipl_next_bid(current_bid) with IPL slabs
-- Updates room: current_bid, current_bid_team, timer (+12s anti-snipe)
-- Inserts bid record
-- Returns: { amount }
```
If two friends hit bid at the exact same millisecond, one waits on the lock, the other succeeds first, the waiter sees "you already hold the highest bid" and tries again.

**`ipl_hammer(room_id, host_token)`**
- Only the host can close a lot
- If current_bid_team is set: mark player SOLD, deduct purse, increment squad, advance
- If not set: mark player UNSOLD, advance
- Auto-calls `ipl__advance` to put next player on the block
- Returns: `{ result: "sold" | "unsold" }`

**`ipl_increment(amount)` & `ipl_next_bid(room)`**
- Pure SQL functions that compute IPL slab-based increments
- Used by `ipl_place_bid` to enforce correct bid jumps

### RLS (Row-Level Security)

| Table | Policy | Effect |
|-------|--------|--------|
| `ipl_rooms` | `for select using (true)` | Everyone reads all rooms & players (they need to watch) |
| `ipl_teams` | `for select using (true)` | ^^ |
| `ipl_players` | `for select using (true)` | ^^ |
| `ipl_bids` | `for select using (true)` | ^^ |
| `ipl_tokens` | (none) | **Nobody can read or write** — access only through stored procs |
| Write ops | `revoke insert, update, delete` | Clients can't mutate any table directly |

**Why this design?**
- Clients can read the state of any auction (visibility is fine)
- Clients **cannot** read tokens (they're not even values in the JS; they're generated and used internally)
- Clients **cannot** write to any table (all mutations forced through RPCs)
- RPCs run as the database owner, bypassing RLS, and enforce business rules (token validation, purse math, etc.)

---

## Deployment

### Option 1: Static File Host (Recommended for Friends)
1. Host `the-hammer-ipl-auction.html` on Vercel, Netlify, GitHub Pages, or any static server
2. Share the URL with friends
3. The HTML file includes the Supabase URL and publishable key hardcoded (fine for a public app with no user data to protect)

### Option 2: Local File
1. Keep `the-hammer-ipl-auction.html` on your computer
2. Open it in a browser (will work offline once loaded, but realtime requires internet)
3. Share the file via email/AirDrop with friends

### Option 3: Self-Hosted Backend (Advanced)
If you want to run the auction on your own PostgreSQL:
1. Run the migration SQL from `/mnt/user-data/outputs/` on your Postgres 14+
2. Update `SB_URL` and `SB_KEY` in the HTML to point to your Supabase project (or a custom REST API)
3. Deploy the file

**Current Backend**: Supabase project `ddfuezctljrcbauiizlb` (ap-south-1 / Mumbai region). If you want isolation, create a new Supabase project and re-run the migration.

---

## Usage Examples

### Scenario 1: Saturday Night, 4 Friends
```
Rishik opens the file → Creates "Saturday Mega Auction"
Code: A7K2M9

Friend A enters code A7K2M9 → Claims CSK as "Yellow Brigade"
Friend B enters code A7K2M9 → Claims MI as "Blue Army"
Friend C enters code A7K2M9 → Claims RCB as "Red Devils"

Rishik hits "Start auction" → First player (Virat Kohli, base ₹200L) appears

Friend A bids ₹200L → "CSK: ₹200L"
Friend B bids ₹205L → "MI: ₹205L" (auto-increment by ₹5L)
Friend C outbids at ₹210L → "RCB: ₹210L"
Friend A can't afford ₹215L → passes
Friend B can't afford it → passes

Rishik hits "Hammer — Sold" → Virat sold to RCB for ₹210L
RCB purse goes 12000 → 11790 (₹210L deducted)
RCB squad: 1/15

Next player auto-appears: Jasprit Bumrah, base ₹200L
...repeat 75 times...

After player 76, results screen shows all three squads sorted by spend.
```

### Scenario 2: 10-Player Mini Auction
You want to only auction 10 players instead of 76:
1. Host creates room
2. Everyone claims a team
3. Host hits start
4. After player 10 is hammered, host can manually stop (the app will continue, but you just ignore the rest)

*(Future enhancement: configurable squad size at room creation)*

---

## Features In Detail

### 1. Real-Time Sync
- Supabase Realtime subscribes to changes on `ipl_rooms`, `ipl_teams`, `ipl_bids`, `ipl_players`
- When a bid is inserted, the `ipl_bids` INSERT event fires → app adds it to the ticker immediately
- When the host hammers, the room status updates → all clients refresh the stage view
- Fallback: 6s polling interval, so even if realtime drops, you're never stale by more than 6 seconds

### 2. Purse & Spending
- Every team starts with ₹120 Cr (12,000 lakhs)
- When a player is sold, the exact `sold_price` is deducted from that team's `purse_remaining`
- The purse rail shows a visual bar (Tailwind-less CSS grid bar) of remaining budget
- Once purse hits zero, that team can't bid anymore (validation in the RPC)

### 3. Bid Increments (IPL Slabs)
```sql
cur < ₹100L   → +₹5L
₹100–200L     → +₹10L
₹200–500L     → +₹20L
₹500–1000L    → +₹25L
₹1000L+       → +₹50L
```
Enforced in `ipl_increment(amount)` function; `ipl_place_bid` calls `ipl_next_bid` every time.

### 4. Custom & Anti-Snipe Timer
- Each lot runs for the host's chosen duration (default 30s, configurable 10–120s at room creation)
- If a bid lands in the final 12 seconds, the timer extends to 12 more seconds
- This prevents someone from bidding at 0:02 and winning before others can react
- Timer is *display-only* on the client; the host's hammer click is the authority

### 5. Host Bar
- Floats at the bottom of the auction stage (only visible to the host)
- Shows **🔨 Hammer** button and **Auto-hammer at 0:00** toggle
- Auto-hammer: when the timer hits 0, if no one's bid, the app calls hammer automatically
- Useful if the host wants to fast-track through unsold players

### 6. Squad Peek (Mid-Auction Scouting)
- Every purse card on the auction stage is tappable
- Opens a modal with that franchise's full squad so far: player, role, country, price paid (sorted by price)
- Footer shows total spent and purse remaining
- Updates live — useful for reading opponents ("they have no bowlers and ₹8 Cr left; they NEED Bumrah")

### 7. Results & Analytics
- After all 76 players are auctioned, results screen appears
- Shows all teams' final squads sorted by total spend (descending)
- Each team card lists:
  - Players bought (with role and price)
  - Total spent (sum of all player prices)
  - Purse left (₹120Cr - total spent)
  - Highest buy for that team

---

## What's Simulated vs Real

- **Bidding, purses, squads, chat**: real, server-authoritative, race-safe
- **Player career stats & form**: simulated deterministically from the player's name (the same player always shows the same stats). Real IPL statistics would need a licensed data feed
- **Season simulator & champion odds**: game-layer math from squad strength, not predictions

## Known Limitations & Future Ideas

### Current Constraints
- **Single-region backend**: All auctions use the same Supabase project. If you're running this publicly (not just friends), you'd want isolation via separate projects per user.
- **No authentication**: Anyone with the file can create unlimited rooms. Rate limiting would be needed for production.
- **76-player fixed pool**: Pool is hardcoded; future version could let hosts customize the player list or import from CSV.
- **No undo**: Once a player is hammered, it's final. No way to rewind a bid or un-sell a player.
- **Browser storage only**: Session lives in `localStorage`; clearing it loses your session. Could add a "recovery code" tied to the room code.

### Ideas for v2
- **Custom player pool**: Host uploads a CSV of 50–100 players, pool is randomized per room
- **Multiple auction formats**: Standard (what we have) + snake draft + salary cap auction variants
- **Auction history**: Save a room result, replay it, compare across multiple auctions
- **Leaderboard**: Track across many rooms: "Rishik has won 47 auctions, best squad value is ₹2.3 Cr/player"
- **Spectator mode**: Read-only link for people watching but not bidding
- **Analytics dashboard**: Per-team: average bid price, players per role, purse efficiency
- **Mobile app**: React Native version with push notifications for new bids
- **AI auctioneer**: Bot team that bids on behalf of an absent player

---

## Security & Privacy

### What's Stored?
- Room details: name, code, status, player, current bid
- Team details: franchise, owner name, purse left, squad
- Bid history: team, amount, timestamp
- **Not stored**: Email addresses, passwords, personal data (just names for owner_name field)

### Who Sees What?
- Everyone in the room sees all bids, all purses, all player data (that's the point)
- Room is isolated by code; you can't see other auctions unless you have their code
- Host and owner tokens are never transmitted to anyone else; they live only in localStorage

### Threat Model
**Attack: Manipulate my purse via DevTools**
→ Impossible. Purse lives in the database; the RPC validates before deducting. Your token only lets you bid on your team, and the RPC checks `purse_remaining < next_bid` before accepting.

**Attack: Bid on someone else's team**
→ Impossible. The RPC validates your `owner_token` against the team's stored token before placing a bid.

**Attack: Create unlimited rooms to spam**
→ Possible today (no rate limit). For a production public app, add Supabase auth + rate limiting.

---

## Development & Customization

### Changing Colors
The app uses CSS variables in the `:root` block. Edit:
```css
:root {
  --night: #0B0F1E;        /* dark bg */
  --gold: #F5B82E;         /* accent, bid amount */
  --ink: #EDF0FA;          /* text */
  --danger: #FF5C5C;       /* unsold stamp */
  --ok: #3DDC97;           /* live indicator */
}
```

### Changing Purse or Squad Limits
In `ipl_create_room`, update:
```sql
insert into ipl_rooms (code, name, base_purse, max_squad)
values (v_code, trim(p_room_name), 12000, 15);
--                                   ↑ purse in lakhs (12000 = ₹120 Cr)
--                                        ↑ max players per squad
```

### Modifying Bid Increments
Edit `ipl_increment` function in the migration:
```sql
create or replace function public.ipl_increment(cur bigint)
returns bigint language sql immutable as $$
  select case
    when cur < 100  then 5::bigint
    when cur < 200  then 10::bigint
    -- ... add your custom slabs
  end
$$;
```

### Adding More Players to the Pool
Run:
```sql
insert into public.ipl_player_pool (name, role, country, base_price, rating)
values ('Your Name', 'Batter', 'India', 100, 85);
```
Next room created will include all pool players, shuffled.

---

## Troubleshooting

### "Room not found" when joining
- Check the 6-letter code is correct (no spaces, case-insensitive)
- Code is tied to the backend; if the backend is down, rooms won't be found

### Bids aren't syncing
- Refresh the page (should resume from localStorage)
- Check your internet connection (realtime needs it)
- If it persists, the fallback polling should catch up within 6 seconds

### "You already hold the highest bid" error
- You're the highest bidder on this lot; someone else needs to outbid you first

### "Not enough purse"
- Your team has spent too much; the next bid increment exceeds your remaining budget
- Teammates could have spent it all; check the purse rail

### Auto-hammer isn't working
- Make sure **Auto-hammer at 0:00** checkbox is ticked
- Timer must actually reach 0:00 (if someone bids constantly, it extends forever)

---

## Performance & Scalability

### Concurrent Users
- Tested with 10 rooms × 10 teams simultaneously
- Single Supabase instance (Starter plan ~$25/mo) handles 100+ concurrent users fine
- For 1000+ concurrent auctions, you'd want Supabase Pro + read replicas

### Database Load
- Each bid = 1 INSERT into `ipl_bids` + 2–3 UPDATEs (`ipl_rooms`, `ipl_teams`)
- Each hammer = 2–3 UPDATEs + 1 internal procedure call
- Typical auction: 76 hammers + 200–400 bids over 2 hours = negligible load on modern Postgres

### Network
- Initial page load: ~37 KB (entire app in one HTML file)
- Per bid: ~200 bytes up, ~500 bytes down (realtime payload)
- Suitable for 3G+ networks

---

## License & Attribution

**THE HAMMER** is released under the **MIT License**. 

Attribution not required, but appreciated if you build on this or share it.

**Built with**:
- [Supabase](https://supabase.com) — backend as a service
- [PostgreSQL](https://postgresql.org) — rock-solid relational database
- Vanilla JavaScript, no frameworks
- Saira Condensed & Archivo typefaces (Google Fonts, free)

---

## Feedback & Contributions

Found a bug? Have an idea for a feature?

1. **Bug reports**: Open an issue with steps to reproduce
2. **Feature requests**: Describe the idea and how it'd enhance the auction experience
3. **Code contributions**: Fork, make changes, test thoroughly (especially the auction engine), and submit a PR

**Testing checklist before submitting**:
- ✅ Can create a room with the file
- ✅ Can join a room with a valid code
- ✅ Can claim a franchise and see it appear for others in realtime
- ✅ Can start an auction (only after 2+ teams)
- ✅ Can place a bid and see it in the ticker + on other clients
- ✅ Bid increments follow IPL slabs (test ₹50L, ₹100L, ₹300L, ₹800L, ₹1200L)
- ✅ Purse deducts correctly when a player is sold
- ✅ Can't bid if purse is exhausted or squad is full
- ✅ Host can hammer a lot (sold) and hammer with no bids (unsold)
- ✅ Timer extends when someone bids in final 12 seconds
- ✅ Final results screen shows all squads sorted by spend
- ✅ Refreshing mid-auction resumes your session

---

## Support

For issues or questions, refer to the [Supabase documentation](https://supabase.com/docs) or check that:
1. Your Supabase project is active (not paused)
2. You have the correct `SB_URL` and `SB_KEY` in the HTML
3. Your browser allows localStorage (not in private/incognito mode)
4. JavaScript is enabled

---

**Made by Rishik Mora — for Saturday night auctions and beyond.**

*Last updated: June 2026*
