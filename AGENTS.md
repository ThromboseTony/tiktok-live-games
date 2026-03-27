# TikTok Live Games

TikTok Live overlay platform — viewers play games via chat during livestreams.
Express + Socket.io server, multi-tenant (1 room per streamer).

## Structure

```
src/
├── server.js              # Express + Socket.io entry (port 3000)
└── services/
    └── TikTokService.js   # Singleton: TikTok Live connections, event forwarding
public/
├── index.html             # Dashboard: username input → overlay URL generator
├── css/styles.css
├── js/dashboard.js        # Game selection, URL generation, clipboard
├── lib/tiktok-bridge.js   # Client SDK: Socket.io → game event bridge
└── games/
    ├── boss-raid/         # Phaser 3 — co-op boss fight (chat: join/hit, gifts=damage)
    └── onslaught-arena/   # Canvas engine — arcade survival (chat: movement commands)
```

## Tech Stack

- **Runtime:** Node.js >= 18 (ES Modules, `"type": "module"`)
- **Server:** Express 4.18 + Socket.io 4.7
- **TikTok:** tiktok-live-connector 1.1.9 (WebcastPushConnection)
- **Games:** Phaser 3 (boss-raid), custom canvas engine (onslaught-arena)
- **Package manager:** npm

## Commands

```bash
npm start        # node src/server.js (production)
npm run dev      # node --watch src/server.js (dev, auto-reload)
```

## Data Flow

```
TikTok Live → tiktok-live-connector → TikTokService.js → Socket.io rooms → tiktok-bridge.js → Game
```

Events: `tiktok_chat`, `tiktok_gift`, `tiktok_like`, `tiktok_share` + legacy `player_join`, `player_attack`, `gift_received`.

## Key Patterns

- **Multi-tenant isolation:** Room ID = TikTok username. `io.to(username).emit()` ensures data isolation.
- **Singleton service:** `TikTokService` reuses connections; auto-disconnects after 5min with 0 clients.
- **Client SDK:** `tiktok-bridge.js` auto-connects from URL params (`?id=` or `?username=`).
- **Gift categorization:** small (<10 diamonds), medium (10-99), large (100+).

## Adding a New Game

1. Create `public/games/{name}/` with entry HTML
2. Include `<script src="/lib/tiktok-bridge.js">` + Socket.io CDN
3. Use `TikTokBridge.on('chat', cb)` / `TikTokBridge.on('gift', cb)`
4. Add game card in `public/index.html` with `data-game`, `data-entry`, `data-param`

## Boundaries

**Never:** Commit .env files, modify node_modules, break Socket.io event names (games depend on them)
**Ask first:** Add new dependencies, change TikTokService event structure, modify tiktok-bridge.js API
