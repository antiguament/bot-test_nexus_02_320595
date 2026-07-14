# N1 NEXUS VIP — Agent Guide

## Stack
- **Runtime:** Node.js (CommonJS), no build step
- **Backend:** Express 5 + Socket.IO 4 — single file (`server.js`)
- **Frontend:** Vanilla HTML/CSS/JS — single file (`public/index.html`, ~3600 lines)
- **WhatsApp:** `@whiskeysockets/baileys` v7 (direct WebSocket, no browser/Puppeteer)
- **AI:** Mistral AI (`mistral-tiny`, max 300 tokens, 25s timeout)
- **Database:** Flat JSON files (no MySQL, no Redis)

## Commands
```sh
npm start            # node server.js
```
No test, lint, typecheck, or build commands exist.

## Entrypoints
- `server.js` — backend (Express routes, Socket.IO events, Baileys WhatsApp bot)
- `public/index.html` — frontend (monolithic, served as static file)

## Data Files (all JSON, flat)
| File | Purpose |
|------|---------|
| `contacts.json` | Contact list with botEnabled/unreadCount/notes |
| `conversations.json` | All message history (keyed by phone number) |
| `bot-knowledge.json` | AI system prompt, plans, FAQ, wisdom phrases |
| `database/settings.json` | Bot config (model, tokens, business info) |
| `database/saved-contacts.json` | Operator's saved contacts |
| `database/predefined-texts.json` | Canned response templates |
| `database/call-log.json` | Rejected call records |

## Known Dead Code (fix before relying on)
- `contactEnabled()` — defined but never called in message pipeline (`messages.upsert`)
- `detectUserContext()` — defined but unused (context detection handled by Mistral prompt)
- `operatorContexts` map — populated by `operator-inject-context` socket event but never consumed by `getMistralReply()`

## Architecture Quirks
- **Lock system:** `.bot.lock` (PID\|timestamp) prevents duplicate instances. Heartbeat every 15s. Stale after 60s.
- **Deploy marker:** `.deploy.marker` — if a newer instance writes this, old instance self-terminates.
- **LID resolution:** `@lid` JIDs resolved to phone numbers via 3 fallback methods (remoteJidAlt, in-memory map, Baileys signalRepository).
- **AI history:** Separate schema (`{role, body, ts}`) from conversations array (`{id, type, from, number, body, timestamp}`). Max 50 entries per contact.
- **Port:** Default 3000, but `.env` sets `PORT=3060`.
- **No auth:** Panel has no login. No rate limiting. No HTTPS (relies on Hostinger proxy).

## Agent System (`.nexus/`)
| Agent | Role |
|-------|------|
| Zion | Architect — designs blueprint, delegates to Core/Orion |
| Core | Backend — implements APIs, data logic, caching |
| Orion | Frontend — builds UI/UX, follows "Nexus Pulse" design |

All agents **must** update `changelog.md` after completing work.

## Git Convention
Conventional commits: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`.

## Environment
Copy `.env` template with: `MISTRAL_API_KEY`, `PORT`, `NODE_ENV`.
