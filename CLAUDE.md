# CLAUDE.md — UAM Testing Machine Dashboard

> ไฟล์นี้คือ "ความจำโปรเจกต์" ของ Claude Code อ่านก่อนเริ่มทุกครั้ง
> เนื้อหาเต็มอยู่ที่ `docs/HANDOFF.md` · พิมพ์เขียว module อยู่ที่ `docs/MODULE-SCHEMA.md`

## โปรเจกต์นี้คืออะไร
Dashboard บริหารงาน **ติดตั้ง + ตรวจรับ (acceptance)** เครื่องทดสอบ 4 เครื่อง จากซัพพลายเออร์ Beijing HZNETINFO ที่โรงงาน UAM Factory 3 (Union Auto Parts Manufacturing, ไทย)

- เครื่อง: HZPJ10(II) Radial Load Fatigue · HZJ700 Impact · HZPN3000 Torsion Fatigue · HZPQ800 Rolling/Bend Fatigue
- เจ้าของงาน: คุณซ้อน (Natthapol Mahaboon) — Senior Supervisor, Product Development
- เส้นตายลงนามรับมอบ: **2026-06-30**

## สถานะปัจจุบัน (v1 — ใช้งานได้จริง)
- ไฟล์เดียว standalone: `index.html` (~205 KB, vanilla JS, ไม่มี build step, โลโก้ฝัง base64)
- เก็บข้อมูลใน **localStorage** (ไม่มี backend):
  - ข้อมูลแผน: `uam-testing-machine-plan-html-v1`
  - ภาษา UI: `uam-lang`
- **11 modules** (แท็บ): Cockpit · Timeline · Table · Actions · Daily · Issues · Risks · Docs · Team · Signoff · Training
- i18n เฉพาะ "กรอบ UI" (TH/EN/JP, 46 จุด `data-i18n`) — ภาษาเริ่มต้น = ไทย (key หาย → ไม่พัง คงไทยไว้)
- พิมพ์รายงาน: เลือกหัวข้อ → พิมพ์ผ่าน **iframe แยก** (กันพิมพ์แท็บที่เปิดอยู่) → A4/Letter, แนวตั้ง/นอน
- Export/Import: ปุ่ม Export ได้ `.json` ทั้ง state (รวมรูป base64) · Import คืนค่ากลับ · Excel `.xls` หลายชีต

## โมเดลข้อมูลที่ตัดสินใจแล้ว (เฟส 1)
**แบบ 2 — "อ่านร่วม เจ้าของแก้คนเดียว + แชร์ .json"**
host แอปปัจจุบันบน GitHub Pages → ทีมเปิดดูได้ทุกคน · คุณซ้อนเป็นเจ้าของข้อมูล แก้คนเดียวแล้ว export `.json` ขึ้นไดรฟ์กลาง
> ⚠️ ตามจริง: GitHub Pages ให้แค่ "เปิดแอปได้" — ข้อมูลยังแยกตาม browser/เครื่อง ไม่ sync กันเอง
> เฟสถัดไป (หลังลงนาม) → อัปเกรดเป็นแบบ 1 (ข้อมูลร่วม real-time) ด้วย **Supabase**

## ❗ สิ่งที่ห้ามทำพังเมื่อแก้ไข
1. **อย่าทำลายข้อมูลผู้ใช้** — STORAGE_KEY ต้องคงเดิม; ถ้าเปลี่ยนโครงสร้าง state ต้องมี migration ตอน Import
2. **อย่าทำให้พิมพ์เพี้ยน** — ระบบพิมพ์ใช้ iframe แยก (อย่ากลับไปใช้ `body.report-print` + `afterprint` ที่เคยพังบนแท็บเล็ต)
3. **i18n ต้อง fallback เป็นไทยเสมอ** — key หายห้าม render ค่าว่าง
4. **ต้องออฟไลน์ได้** — ห้ามผูก CDN ที่จำเป็นต่อการทำงานหลัก (โลโก้/ฟอนต์หลักฝังในไฟล์)
5. **รองรับจอเล็ก (MatePad/มือถือ)** — ตารางกว้างต้องยุบเป็นการ์ดได้

## ขนบการเขียน (conventions)
- UI/ข้อความผู้ใช้ = **ไทย** (ทางการ) · technical term/โค้ด/คอมเมนต์ = อังกฤษได้
- ทำ QA ทุกครั้งก่อนส่ง (มี harness ตัวอย่าง `selftest.js` แนวทางใน HANDOFF) — **ห้าม introduce bug**
- รายงานปัญหาตามจริง ห้ามบิดเบือน

## Roadmap (อ่านเต็มใน docs/HANDOFF.md)
- **เฟส 0 (ทันที):** ขึ้น GitHub + เปิด Pages ด้วยไฟล์ปัจจุบัน (ดู docs/GITHUB-SETUP.md)
- **เฟส 1 (หลัง 30 มิ.ย.):** refactor ภายในให้เป็น **config-driven** ตาม docs/MODULE-SCHEMA.md (ยังไฟล์เดียว/ออฟไลน์)
- **เฟส 2:** ทำเป็น **PWA** (เพิ่มลงหน้าจอ MatePad ได้ ไม่ต้องผ่าน Play Store)
- **เฟส 3:** **Supabase** — ข้อมูลร่วม + login/สิทธิ์ + audit log

## Stack
v1 คงไฟล์เดียว HTML+JS (เร็วสุด, ใช้ได้ทันที). สำหรับ refactor เฟส 1+ ให้ Claude Code เสนอ stack ตามโจทย์ (ตัวเลือกที่เหมาะ: คง vanilla + แตกเป็นหลายไฟล์, หรือ Vite + เฟรมเวิร์กเบา) — รักษาเงื่อนไข "ออฟไลน์ + แท็บเล็ต + ไม่มี build ที่ยุ่งยากเกินจำเป็น"
