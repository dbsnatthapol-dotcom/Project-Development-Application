# GITHUB-SETUP.md — ขึ้น GitHub + เปิดให้ทีมเข้าถึง (เฟส 0)

> เป้าหมาย: เอาแอปปัจจุบัน (`index.html`) ขึ้น GitHub แล้วเปิด **GitHub Pages**
> ให้ทีมเปิดผ่านลิงก์เดียวได้ทุกคน — *ทำได้ทันที ไม่ต้องแก้โค้ด*

---

## ⚠️ อ่านก่อน (ตามจริง)
- GitHub Pages ให้แค่ **"ตัวแอป"** — ทีมเปิดดูได้ทุกคน
- **ข้อมูลไม่ sync กันเอง** (อยู่ใน localStorage แยกตามเครื่อง/บราวเซอร์)
- โมเดลที่ใช้ = **เจ้าของแก้คนเดียว (คุณซ้อน) แล้วแชร์ `.json`** ขึ้นไดรฟ์กลาง ให้คนอื่น Import ดู
- repo ควรตั้งเป็น **Public** (Pages ฟรีต้อง public) → ระวังอย่าใส่ข้อมูลลับ/ส่วนตัวลงในไฟล์ที่ push
  (ข้อมูลจริงอยู่ในไฟล์ `.json` ที่ export — **ไม่ต้อง push ขึ้น GitHub**)

---

## วิธีที่ง่ายสุดบนแท็บเล็ต (MatePad) — อัปโหลดผ่านเว็บ
ไม่ต้องลง git/โปรแกรมอะไรเลย ใช้เบราว์เซอร์อย่างเดียว:

1. สมัคร/เข้า **github.com** → กด **New repository**
2. ตั้งชื่อ เช่น `uam-testing-dashboard` → เลือก **Public** → **Create repository**
3. ในหน้า repo กด **uploading an existing file** (หรือ Add file → Upload files)
4. ลากไฟล์เหล่านี้เข้าไป (จากแพ็กนี้):
   - `index.html` ← **สำคัญสุด** (ตัวแอป)
   - `README.md`
   - โฟลเดอร์ `docs/` (ถ้าต้องการเก็บเอกสารไว้ด้วย)
5. กด **Commit changes**
6. เปิด Pages: **Settings → Pages → Source = Deploy from a branch → Branch: main / (root) → Save**
7. รอ ~1 นาที จะได้ลิงก์รูปแบบ:
   `https://<ชื่อผู้ใช้>.github.io/uam-testing-dashboard/`
8. ส่งลิงก์นี้ให้ทีม → เปิดได้ทุกคน (มือถือ/แท็บเล็ต/PC)

> อัปเดตแอปครั้งต่อไป: กลับมาที่ repo → Upload files ทับ `index.html` เดิม → Commit → Pages อัปเดตเอง

---

## วิธีสำหรับคนที่ใช้คอม + git (เร็วกว่าเวลาทำบ่อย)
```bash
cd uam-dashboard
git init
git add .
git commit -m "v1: UAM testing machine dashboard"
git branch -M main
git remote add origin https://github.com/<ผู้ใช้>/uam-testing-dashboard.git
git push -u origin main
# จากนั้นเปิด Pages ตามขั้น 6 ด้านบน
```

---

## ทางเลือก host อื่น (ถ้าไม่อยากใช้ GitHub Pages)
- **internal web / SharePoint / Google Sites** ของบริษัท — วาง `index.html` แล้วแชร์ URL ภายใน (เหมาะถ้าต้องการจำกัดเฉพาะคนในองค์กร)
- ทุกทางเลือกเหมือนกันตรงที่ **ข้อมูลยังแยกตามเครื่อง** — ถ้าต้องการข้อมูลร่วมจริง ต้องไปเฟส 3 (Supabase)

---

## วินัยข้อมูล (โมเดลแบบ 2 — ต้องทำ)
1. คุณซ้อนเป็น **เจ้าของข้อมูลหลัก** — แก้ในเครื่องตัวเอง
2. **Export `.json` สม่ำเสมอ** (อย่างน้อยสัปดาห์ละครั้ง + ก่อนทำอะไรเสี่ยง) ขึ้น OneDrive/Google Drive
3. ตั้งชื่อไฟล์มีวันที่ เช่น `uam_testing_plan_2026-06-26.json` (เก็บหลายเวอร์ชัน)
4. ทีมที่อยากดูข้อมูลล่าสุด → เปิดแอป (ลิงก์ Pages) → **Import** ไฟล์ `.json` ล่าสุดที่แชร์
5. **ห้าม push ไฟล์ `.json` ข้อมูลจริงขึ้น GitHub public** — เก็บบนไดรฟ์กลางที่จำกัดสิทธิ์แทน
