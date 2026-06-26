# MODULE-SCHEMA.md — พิมพ์เขียว Module แบบ Config-Driven

> เป้าหมาย: ทำให้ทุก module **นิยามด้วย config เล็ก ๆ** ไม่ใช่โค้ดยาว ๆ
> เครื่อง render กลาง 1 ตัว อ่าน config แล้วสร้าง ตาราง + ฟอร์ม + validation ให้อัตโนมัติ
> ผล: เพิ่ม/แก้ฟิลด์ = แก้ config ไม่กี่บรรทัด (ไม่ต้องแตะ logic) → แก้ง่าย บั๊กน้อย ทุกหน้าเหมือนกัน

---

## 1) โครงสร้าง Config ของแต่ละ Module

```js
const MODULE = {
  id: "actions",                 // คีย์ภายใน + localStorage sub-key
  icon: "✅",
  name: { th:"To-do", en:"To-do", jp:"タスク" },
  group: "work",                 // "overview" | "work" | "registry"
  type: "crud",                  // "crud" | "readonly" | "custom"
  primaryKey: "id",
  sort: { key:"due", dir:"asc" },
  search: ["title","owner"],     // ฟิลด์ที่ค้นหาได้
  fields: [
    { key:"title",  label:{th:"งาน",en:"Task",jp:"タスク"}, type:"text",     required:true,  width:"40%" },
    { key:"owner",  label:{th:"ผู้รับผิดชอบ"},               type:"text" },
    { key:"due",    label:{th:"กำหนดเสร็จ"},                 type:"date" },
    { key:"pri",    label:{th:"ความสำคัญ"},  type:"select",  options:"APRI" },     // อ้าง vocab กลาง
    { key:"status", label:{th:"สถานะ"},      type:"select",  options:"ASTAT", badge:true },
    { key:"note",   label:{th:"หมายเหตุ"},   type:"textarea" }
  ],
  actions: ["add","edit","duplicate","delete"],   // soft-delete = ถังขยะ/undo
};
```

### ชนิดฟิลด์ (type) ที่รองรับ
`text` · `textarea` · `number` · `date` · `select` (อ้าง vocab กลาง) · `multiselect` · `progress` (0–100) · `photos` (อัปโหลด+บีบ base64) · `richlist` (เช่น WI ในหน้า Training) · `raci` (เฉพาะหน้า Team)

### Vocab กลาง (ใช้ซ้ำทุก module — ของเดิมมีอยู่แล้ว)
`STATUS/sMeta` · `RES/resMeta` · `MSTAT` · `SEV` · `IST` · `WICAT` · `ASTAT` · `APRI` · `DCAT` · `DSTAT` · `RACI` · `LVL` · `RSTAT` · `riskLevel(l,i)`
แต่ละตัวเก็บ `{value: {label:{th,en,jp}, color}}` → ใช้สร้าง dropdown + badge สีอัตโนมัติ

---

## 2) เครื่อง Render กลาง (1 ตัว ใช้ทุก module)

```
renderModule(cfg):
  ถ้า type=="readonly"  → renderSummary(cfg)         // Dashboard/Timeline/Table
  ถ้า type=="crud"      → renderTable(cfg) + ปุ่ม add  // ส่วนใหญ่
  ถ้า type=="custom"    → เรียก renderer เฉพาะ (Signoff)

renderTable(cfg):
  - แถบเครื่องมือ: ค้นหา + กรอง + ปุ่ม add/import/export (ราย module)
  - หัวตารางจาก cfg.fields
  - จอเล็ก → แปลงแต่ละแถวเป็น "การ์ด" อัตโนมัติ
  - empty state มีปุ่มแนะนำ

openForm(cfg, record?):
  - สร้างฟอร์มจาก cfg.fields (label ตามภาษา, required ใส่ *)
  - validation: required + ชนิดข้อมูล → เตือนก่อนบันทึก
  - บันทึก → data layer → autosave → toast "บันทึกแล้ว"
```

### Data layer กลาง (ทุก module เรียกผ่านนี้)
```
db.list(moduleId)            db.get(moduleId, id)
db.add(moduleId, rec)        db.update(moduleId, id, patch)
db.remove(moduleId, id)      // soft-delete → trash
db.restore(moduleId, id)     // undo
db.search(moduleId, q)       db.exportModule(id)  db.importModule(id, json)
+ autosave ลง localStorage (STORAGE_KEY เดิม) + migration เมื่อเวอร์ชัน schema เปลี่ยน
```

---

## 3) Config ครบทั้ง 11 Module (ย่อ — Claude Code เติมรายละเอียดได้)

### กลุ่มภาพรวม (readonly — สรุปจาก module อื่น)
| id | icon | ชื่อ | สรุปจาก |
|---|---|---|---|
| cockpit | 📊 | Dashboard | KPI + readiness gates + สถานะเครื่อง |
| timeline | 📅 | Timeline (Gantt) | tasks (scale day/week/month) |
| table | 📋 | Table | tasks (ตารางเต็ม) |

### กลุ่มบันทึกงาน (CRUD — แก้บ่อย)
| id | icon | ฟิลด์หลัก |
|---|---|---|
| actions | ✅ | title*, owner, due(date), pri(APRI), status(ASTAT,badge), note |
| daily | 📝 | date*, by, summary(textarea), photos |
| issues | ⚠️ | title*, sev(SEV,badge), status(IST,badge), owner, due, detail, photos |
| risks | 🛡️ | title*, likelihood(1-5), impact(1-5)→riskLevel(badge), mitigation, owner, status(RSTAT) |
| tasks | 🗂️ | desc*, start(date), end(date), pic, status(STATUS), progress, cat, remark |

### กลุ่มทะเบียน/ปิดงาน
| id | icon | ฟิลด์หลัก / หมายเหตุ |
|---|---|---|
| docs | 📄 | name*, cat(DCAT), status(DSTAT), owner, link/file, note |
| team | 👥 | name*, role, contact, raci(type:raci) |
| signoff | ✍️ | **type:"custom"** — 8 jobs × cells[4]{result,note,photos} + ผู้อนุมัติ/ตรวจ/ออก |
| training | 🎓 | wis(type:richlist): {wi, trainer, date, attendees, status(WICAT)} |

> `*` = required · ชื่อ vocab ในวงเล็บ = อ้าง vocab กลาง (สร้าง dropdown+badge เอง)

---

## 4) แผนทำทีละขั้น (กัน regression)
1. สร้าง **data layer + vocab กลาง + เครื่อง render** โดยยังไม่แตะ module เดิม
2. แปลง **1 module ง่ายสุดก่อน** (เช่น `actions`) เป็น config → เทียบผลกับของเดิม
3. แปลงกลุ่ม CRUD ที่เหลือ (issues, risks, daily, docs, team, training, tasks)
4. ต่อกลุ่ม readonly (cockpit/timeline/table) ให้ดึงจาก data layer กลาง
5. `signoff` เป็น custom renderer (คงฟอร์มเฉพาะเดิม)
6. ทุกขั้น: รัน self-test + เทียบ output กับเวอร์ชันเดิม ก่อนไปต่อ

> หลักการ: **migration ต้องอ่าน state เดิมได้ 100%** — ผู้ใช้ที่มีข้อมูลอยู่แล้วต้องไม่สูญหาย
