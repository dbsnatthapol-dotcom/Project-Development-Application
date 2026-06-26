# HANDOFF.md — แพ็กส่งต่อสำหรับ Claude Code

เอกสารนี้คือ "บริบทรวม" ของโปรเจกต์ UAM Testing Machine Dashboard
ใช้เป็นข้อมูลตั้งต้นให้ Claude Code เข้าใจงานทั้งหมดก่อนลงมือ refactor/พัฒนาต่อ

---

## 1) ภาพรวมโปรเจกต์

| หัวข้อ | รายละเอียด |
|---|---|
| ชื่อแอป | UAM — Development Plan Testing Machine |
| วัตถุประสงค์ | บริหารงานติดตั้ง + ตรวจรับ (acceptance) เครื่องทดสอบ 4 เครื่อง |
| สถานที่ | UAM Factory 3 (Union Auto Parts Manufacturing, ไทย) |
| ซัพพลายเออร์ | Beijing HZNETINFO |
| เจ้าของงาน | คุณซ้อน (Natthapol Mahaboon) — Senior Supervisor, Product Development (lvl 13) |
| เส้นตายลงนาม | **2026-06-30** |
| อุปกรณ์หลักของผู้ใช้ | Huawei MatePad (แท็บเล็ต, ไม่มี Google Play) |

เครื่องทดสอบ 4 เครื่อง:
1. **HZPJ10(II)** — Radial Load Fatigue
2. **HZJ700** — Impact
3. **HZPN3000** — Torsion Fatigue
4. **HZPQ800** — Rolling / Bend Fatigue

บริบทอุตสาหกรรม: งานชุบ Ni-Cr ขอบล้อมอเตอร์ไซค์ (ลูกค้า Honda/Yamaha/Kawasaki), ระบบ IATF 16949 / APQP — รายงานต้องดูเป็นทางการ พิมพ์/ลงนามได้

---

## 2) สถานะปัจจุบัน (v1 ใช้งานจริง)

- **ไฟล์เดียว** `index.html` (~205 KB) — vanilla HTML/CSS/JS, **ไม่มี build step**, โลโก้ UAM ฝัง base64
- เก็บข้อมูล **localStorage** (ไม่มี backend):
  - `uam-testing-machine-plan-html-v1` — state ทั้งหมด
  - `uam-lang` — ภาษา UI (`th`/`en`/`jp`)
- ทำงาน **ออฟไลน์ได้** 100%

### 11 Modules (แท็บ)
| id | ชื่อ | กลุ่ม | ชนิด |
|---|---|---|---|
| viewCockpit | Dashboard/Cockpit | ภาพรวม | read-only (สรุป) |
| viewTimeline | Timeline (Gantt) | ภาพรวม | read-only |
| viewTable | Table | ภาพรวม | read-only |
| viewActions | To-do / Actions | บันทึกงาน | CRUD |
| viewDaily | บันทึกประจำวัน | บันทึกงาน | CRUD |
| viewIssues | ปัญหา | บันทึกงาน | CRUD |
| viewRisks | ความเสี่ยง | บันทึกงาน | CRUD |
| viewDocs | เอกสาร | ทะเบียน/ปิดงาน | CRUD |
| viewTeam | ทีม (RACI) | ทะเบียน/ปิดงาน | CRUD |
| viewSignoff | ตรวจรับ/ลงนาม | ทะเบียน/ปิดงาน | แบบฟอร์มเฉพาะ |
| viewTraining | อบรม (WI) | ทะเบียน/ปิดงาน | CRUD |

> หมายเหตุ: นอกจาก 11 แท็บ ยังมีงาน/แผน (tasks) ที่ป้อนเป็น Gantt ในหน้า Timeline และตารางในหน้า Table

### ฟีเจอร์ที่ทำเสร็จแล้ว (ต้องรักษาไว้)
- **i18n กรอบ UI** TH/EN/JP — 46 จุด `data-i18n` + dictionary overlay; ไทย = ค่า default/fallback (key หาย → คงไทย ไม่พัง). เนื้อหา *ในแท็บ* + รายงานพิมพ์ ยังเป็นไทย (เฟสถัดไปค่อยขยาย)
- **พิมพ์รายงาน** = เลือกหัวข้อ (10 กลุ่ม) → render เฉพาะที่ติ๊ก → พิมพ์ผ่าน **iframe แยก** (`#printFrame`) เพื่อกัน "พิมพ์แท็บที่เปิดอยู่" → เลือก A4/Letter + แนวตั้ง/นอน
- **Export/Import** — Export `.json` = state ทั้งหมด (รวมรูป base64, self-contained) · Import คืนค่ากลับ · Excel `.xls` หลายชีต (ไม่มีรูป)
- **Help ❓** — modal วิธีใช้ในแอป
- **Responsive** — ปรับตามจอ (รวมแท็บเล็ต)

---

## 3) สถาปัตยกรรมที่ตัดสินใจ + Roadmap

### โมเดลข้อมูล (สำคัญสุด)
ตัดสินใจ **เฟส 1 = แบบ 2 "อ่านร่วม เจ้าของแก้คนเดียว + แชร์ .json"**

- host แอปบน GitHub Pages → ทุกคนเปิดดูได้
- คุณซ้อน = เจ้าของข้อมูล แก้คนเดียว → export `.json` ขึ้นไดรฟ์กลาง (OneDrive/Google Drive) เป็นชุดอ้างอิงเดียว
- ⚠️ **ตามจริง:** GitHub Pages ให้แค่ตัวแอป — ข้อมูลยังอยู่ใน localStorage แยกตามเครื่อง/บราวเซอร์ **ไม่ sync เอง**
- **อัปเกรดเป็นแบบ 1 (ข้อมูลร่วม real-time)** = เฟส 3 ด้วย Supabase

### Roadmap เป็นเฟส
- **เฟส 0 (ทันที, ก่อน 30 มิ.ย.):** ขึ้น GitHub + เปิด Pages ด้วย `index.html` ปัจจุบัน — *ความเสี่ยงต่ำ ไม่แตะโค้ด* (ดู `GITHUB-SETUP.md`). ช่วงนี้ **feature freeze** กัน regression ก่อนลงนาม
- **เฟส 1 (หลังลงนาม):** refactor ภายในให้เป็น **config-driven** ตาม `MODULE-SCHEMA.md` — ทุก module หน้าตา/พฤติกรรมเดียวกัน, แก้ = แก้ config ไม่กี่บรรทัด. ยังไฟล์เดียว/ออฟไลน์ (อาจแตกหลายไฟล์ + รวมตอน build เบา ๆ)
- **เฟส 2:** ทำเป็น **PWA** — manifest + service worker → "เพิ่มลงหน้าจอ" บน MatePad ได้ ใช้ออฟไลน์เหมือนแอป **ไม่ต้องผ่าน Play Store**
- **เฟส 3:** **Supabase** — ข้อมูลร่วม + login/บทบาท (เจ้าของแก้/ทีมดู) + audit log (ใครแก้อะไรเมื่อไร — มีค่ามากกับ IATF) + เปิด URL เดียวทั้งทีม

---

## 4) ชั้นจัดการข้อมูลที่อยากได้ (เป้าหมาย refactor)

ทำ "data layer" กลางให้ทุก module เรียกใช้ร่วม:
- **CRUD มาตรฐาน:** add / edit / duplicate / delete + autosave
- **ลบแบบกู้คืนได้ (trash / undo)** — กันลบพลาด (สำคัญกับงานจริง)
- **Validation + ช่องจำเป็น** — เตือนก่อนบันทึก
- **ค้นหา / กรอง / เรียง** ในแท็บที่ข้อมูลเยอะ
- **Import/Export ราย module** (ไม่ใช่แค่ทั้งแอป)
- **Schema migration** ตอน Import — กันไฟล์สำรองเก่าพังเมื่อโครงสร้างเปลี่ยน

---

## 5) ข้อกำหนด UX (ให้ "ใช้ง่าย")
- **ความสม่ำเสมอ** — ทุก module ปุ่ม/ตำแหน่งเหมือนกัน (เรียนรู้ครั้งเดียว ใช้ได้ทุกแท็บ)
- **จอเล็กแสดงเป็นการ์ด** แทนตารางกว้าง
- **Empty state มีคำแนะนำ** ("ยังไม่มีข้อมูล — กดเพิ่ม")
- **Feedback ชัด** — toast "บันทึกแล้ว", เตือน validation
- **Help / tooltips** (มี ❓ แล้ว) + ปุ่มแตะใหญ่ (touch target)

---

## 6) ❗ ข้อห้าม (อย่าทำพัง)
1. **ข้อมูลผู้ใช้** — STORAGE_KEY คงเดิม; เปลี่ยนโครงสร้าง state ต้องมี migration ตอน Import
2. **ระบบพิมพ์** — ใช้ iframe แยก (`#printFrame`); **ห้าม**กลับไปใช้ `body.report-print` + `afterprint` (เคยพังบนแท็บเล็ต: `afterprint` ยิงเร็วเกินจน snapshot ถ่ายไม่ทัน → พิมพ์แท็บปัจจุบันแทน)
3. **i18n** — fallback ไทยเสมอ; key หายห้าม render ว่าง
4. **ออฟไลน์** — ห้ามผูก CDN ที่จำเป็นต่อ core (โลโก้/ฟอนต์หลักฝังในไฟล์)
5. **จอเล็ก** — ตารางกว้างต้องยุบเป็นการ์ดได้

---

## 7) แนวทาง QA (กัน regression)
มีแนวทาง self-test ด้วย Node (fake-DOM Proxy) ในประวัติงาน — หลักการ:
- โหลดโค้ดแอปใน sandbox, render ทุก module → ต้องไม่ throw และมี output > 0
- timeline ที่ scale day/week/month
- `buildPrint(เลือก 10 หัวข้อ)` = 10 sections; เลือก subset = ตรงจำนวน (print fidelity)
- `buildExcel` well-formed
- empty-state (ล้าง array) ทุก module ยัง render ได้
- static check: วงเล็บ/backtick สมดุล, `new Function(app)` parse ได้, ทุก `getElementById` มี id จริงใน HTML, ไม่มี `page:tl`/`psec-landscape` หลงเหลือ, จำนวน `data-i18n` = จำนวน key EN = JP

> สิ่งที่ Node ตรวจแทนไม่ได้ (ต้องทดสอบบนเครื่องจริง): การพิมพ์/กล่อง dialog จริง, layout/overflow ที่ตามอง, persistence ข้ามรอบ, อัปโหลด/บีบรูป

---

## 8) Prompt ตั้งต้นสำหรับ Claude Code (คัดลอกไปวาง)

```
อ่าน CLAUDE.md และ docs/HANDOFF.md, docs/MODULE-SCHEMA.md ก่อน

งานของฉัน: ฉันมีแอป index.html (ไฟล์เดียว ~205KB) ที่ใช้งานจริงแล้ว
ตอนนี้อยากทำ "เฟส 1: refactor ภายในให้เป็น config-driven" ตาม MODULE-SCHEMA.md
โดยยังคง: ไฟล์ทำงานออฟไลน์ได้, ข้อมูลเดิมใน localStorage ต้องใช้ต่อได้ (มี migration),
ระบบพิมพ์ iframe เดิม, i18n เดิม

ขอให้:
1) วางแผนเป็นขั้นเล็ก ๆ ที่ไม่ทำของเดิมพัง (บอกความเสี่ยงแต่ละขั้น)
2) เสนอ stack ที่เหมาะกับเงื่อนไข (แท็บเล็ต/ออฟไลน์/ไม่มี build ยุ่งยากเกินจำเป็น)
3) ทำทีละ module เริ่มจากกลุ่ม CRUD (Actions, Issues) เป็น proof-of-concept
4) เขียน self-test กัน regression ทุกขั้น
ตอบกลับเป็นภาษาไทย และถามถ้าข้อมูลไม่พอ ก่อนลงมือ
```

> เฟส 2 (PWA) / เฟส 3 (Supabase) ค่อยสั่งเป็นรอบแยกหลังเฟส 1 ผ่าน QA
