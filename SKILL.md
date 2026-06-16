# lottery-app — Project Guide

> ดูภาพรวมแบบ visual ได้ที่ `workflow.html` (เปิดในเบราว์เซอร์)

## Stack
- **Frontend**: `index.html` — deploy บน Vercel (auto-deploy เมื่อ push to `master`)
- **Backend**: `worker/` — Cloudflare Worker, deploy แยกด้วย Wrangler
- **Database**: Google Sheets (ผ่าน Worker API)

## หลักการสำคัญ (อ่านก่อนแก้เลข)
**Sheets เก็บแค่ "ข้อมูลดิบ" — ไม่เก็บยอดคงเหลือของงวดไหนเลย**

ค่าเดียวที่เป็นตัวตั้งต้นคือ `initialBalance` (ยอดเริ่มต้นก่อนงวดแรก) ส่วน `ยอดเก่า`,
`72%`, `คงเหลือ` ของทุกงวด **คำนวณสดทุกครั้ง** ตอน render โดย `runningBalance()` ที่ไล่ replay
จาก `initialBalance` ผ่านทุกงวดตามลำดับ

ผลที่ตามมา: ถ้าแก้สูตรคำนวณ หรือแก้ข้อมูลดิบของงวดใดงวดหนึ่ง → ยอดคงเหลือของงวดนั้น
**และทุกงวดถัดไป** จะขยับตามทั้งหมด (เพราะมันต่อยอดกันเป็นลูกโซ่)

## Data model (Google Sheets)
ชีต **Settings** (`Settings!A:B`):
```
key            | value
agents         | ["แม่","ซ้อ",...]   (JSON array)
initialBalance | 0
```
ชีต **Periods** (`Periods!A:F`) — 1 แถว = 1 งวด:
```
id | date | createdAt | bets | winTypes | deduct
```
`bets`, `winTypes`, `deduct` เก็บเป็น JSON string เช่น
- `bets`     = `{"แม่":1000,"ซ้อ":500,...}`  (ยอดรับแต่ละคน)
- `winTypes` = `{"สองตัว":0,"สองอั้น":0,"โต๊ด":0,"สามตรง":0,"วิ่งบน":0,"อื่นๆ":0}`  (ตัวถูก)
- `deduct`   = `{"ค่าแรง":0,"คืน":0,"เบิก":0}`  (รายการหัก/เบิก)

## สูตรคำนวณต่องวด (`calcPeriod` ใน index.html)
```
total72   = ผลรวมยอดรับทุกคน × 0.72        (เจ้ามือเก็บ 72%)
totalWin  = ผลรวม winTypes                  (ตัวถูก ที่ต้องจ่าย)
totalDeduct = ค่าแรง + คืน                  (หักออก)
totalAdd  = เบิก                            (บวกเพิ่ม)

grandTotal = ยอดเก่า + total72 + totalAdd
คงเหลือ    = grandTotal − totalWin − totalDeduct
```
> เบิก = บวกเข้ายอด, ค่าแรง/คืน = หักออก, "อื่นๆ" ในรายการหักถูกเอาออกแล้ว
> (commit `2d5ed6a`) — งวดเก่าที่เคยมีค่าพวกนี้จะถูกคำนวณใหม่ตามกติกานี้

## Worker API
- `GET  /api/data` → `{ settings, periods }` (ดึงทั้งหมดในครั้งเดียว)
- `POST /api/periods` → body = 1 period object (เพิ่มงวดใหม่)
- `DELETE /api/periods/:id` → ลบงวดตาม id (frontend ปัจจุบันยังไม่เรียกใช้)
- `PUT  /api/settings` → `{ agents, initialBalance }`

Worker secrets (ตั้งผ่าน `wrangler secret put`): `SERVICE_ACCOUNT_JSON`, `SPREADSHEET_ID`
(ดู Google auth = JWT service account แบบเดียวกับ Insurance CRM)

## Frontend notes
- Worker URL เก็บใน `localStorage` (`lotto_api_url`) ตั้งในหน้า ⚙️ ตั้งค่า — ไม่ได้ hardcode
- `lotto_agents` ใน localStorage เป็น fallback รายชื่อ agent ตอน API ล่ม
- ไม่มี framework — plain HTML/CSS/JS ไฟล์เดียว, mobile-width 480px, มี bottom nav 4 หน้า

## Commit format
```
<Verb> <what> — <detail>
```
กฎ: ภาษาอังกฤษ, Verb แรก imperative (Fix/Add/Update/Remove/Refactor), ไม่มี period ท้าย

## Deploy
- **Frontend**: push to `master` → Vercel auto-deploy
- **Worker**: `cd worker && npx wrangler deploy`

## What to commit / NOT commit
- ✅ `index.html`, `worker/src/index.js`, `vercel.json`, `worker/wrangler.toml`, docs
- ❌ `.env`, secrets, API keys, `worker/node_modules/`
