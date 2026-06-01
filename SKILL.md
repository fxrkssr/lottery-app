# lottery-app — Project Guide

## Stack
- **Frontend**: `index.html` — deploy บน Vercel (auto-deploy เมื่อ push to `master`)
- **Backend**: `worker/` — Cloudflare Worker, deploy แยกด้วย Wrangler
- **Database**: Google Sheets (ผ่าน Worker API)

## Commit format
```
<Verb> <what> — <detail>
```
ตัวอย่างจาก git log:
- `Fix bet table layout for mobile — use row cards instead of wide table`
- `Remove delete button — data editable via admin in Sheets only`

กฎ:
- ภาษาอังกฤษ
- Verb แรกเป็น imperative: Fix, Add, Update, Remove, Refactor
- ไม่ต้องมี period ท้าย

## Deploy frontend
Push to `master` → Vercel auto-deploy ทันที (ไม่ต้องทำอะไรเพิ่ม)

## Deploy worker
```powershell
cd worker
npx wrangler deploy
```
Worker URL เก็บใน `wrangler.toml` → ต้องตั้งค่าใน Settings ของแอป

## What to commit
- `index.html` — ทุกการเปลี่ยนแปลง UI/logic
- `worker/src/index.js` — เมื่อ API เปลี่ยน
- `vercel.json`, `worker/wrangler.toml` — เมื่อ config เปลี่ยน

## What NOT to commit
- `.env`, secrets, API keys
- `worker/node_modules/`
