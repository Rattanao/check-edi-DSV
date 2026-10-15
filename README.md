# Check EDI DSV

เครื่องมือตรวจสอบเอกสารขาเข้าเรือก่อนจัดทำใบขนสินค้าขาเข้า (Thailand ocean import) —
เทียบ **MANIFEST** (รายงานจากสายเรือ, .xls/.xlsx) กับ **ENTER** (ฟอร์ม AMENDMENT OF MANIFEST
ของ DSV, .pdf — หน้าละ 1 B/L) ทีละ B/L ย่อย แล้วสร้างรายงาน `EDI.xlsx` แบบไฮไลต์สีบอกจุดที่ต้องแก้ก่อนยื่นใบขน

> **ข้อจำกัด:** หน้าเว็บรองรับเฉพาะ **ฟอร์ม AMENDMENT OF MANIFEST ของ DSV**
> ถ้าเจอฟอร์มของเจ้าอื่นหน้าเว็บจะแจ้งว่าไม่ใช่ฟอร์ม DSV · ไฟล์ CNTRS ไม่นำมาเทียบ ·
> ถ้า PDF รวมหลายฟอร์มไว้ในไฟล์เดียว หน้าเว็บอ่านเฉพาะหน้าที่เป็นฟอร์ม DSV

## Check EDI DSV บนเว็บ (ให้เพื่อนใช้ได้ ไม่ต้องติดตั้ง)

https://rattanao.github.io/check-edi-DSV/

1. ลากไฟล์ **MANIFEST (.xls/.xlsx)** และ **ENTER ของ DSV (.pdf ได้หลายไฟล์)** มาวางพร้อมกัน
   (ตั้งชื่ออะไรก็ได้ — แยกประเภทจากนามสกุล; ไฟล์ที่ชื่อมี `CNTRS` จะถูกข้าม)
2. ใส่ SHED NO. ที่แจ้ง (ถ้ามี) แล้วกด **ตรวจสอบ**
3. ดูผลบนหน้าเว็บ (ตารางสีเหมือน EDI.xlsx, ค้นหา/กรอง "แสดงเฉพาะที่ไม่ตรง") หรือกด **ดาวน์โหลด EDI.xlsx**

ไฟล์ถูกประมวลผลในเบราว์เซอร์ของผู้ใช้เท่านั้น ไม่มีการอัปโหลดไปที่ใด

โค้ดหน้าเว็บอยู่ที่ `docs/` (`index.html` + `core.js` — พอร์ตจาก `build_edi.py` layout G เฉพาะฟอร์ม DSV,
ฐานเดียวกับ [Check EDI Quality](https://rattanao.github.io/check-edi-Quality/),
[Check EDI AGN](https://rattanao.github.io/check-edi-AGN/) และ
[Check EDI Penanshin](https://rattanao.github.io/Penanshin/))

## ฟอร์ม DSV ที่อ่าน

ต่อหน้า: `B/L CHANGED:` + เลข B/L · `Consignee` (ชื่ออยู่บรรทัดถัดลงมาจนถึง `Notify party`) ·
`STATUS: "ลากตู้ด่วน" CY **HOT BOX**` · ตาราง `Mark & Numbers | PKGS TYPE | DESCRIPTION OF GOODS | WT & DIMS`
(เลขตู้ `XXXX1234567 / 40'HC` ท้ายคอลัมน์ DESCRIPTION แยกเป็น CONTAINER ไม่นับเป็น description)

## ตรวจอะไรบ้าง

- CONSIGNEE, CONTAINER NO., STATUS (CY / LCL / LCL-CFS), TOTAL PACKAGE, PACKAGING
- GROSS WEIGHT, MEASUREMENT
- MARKS, DESCRIPTION — เทียบทีละคู่ ไม่นับช่องว่าง/วรรคตอน แต่คำที่เกิน/ขาดต้องเตือน (สีแดง)
- REEFER TEMP, DG (CLASS/UN), TRANSIT/TRANSHIPMENT — พบฝั่งใดก็เตือนเสมอ
- SHED NO., ประเทศปลายทาง, CARGO MOVEMENT, TAX ID เทียบ NOTIFY — **ถูกต้องตามกฎ = แสดง `-`**
  (แสดงค่าจริงเฉพาะตอนไม่ตรง/ต้องตรวจ)
- ข้อความสีแดงที่ลูกค้าพิมพ์ทับ PDF (FreeText) ใช้กับ B/L ของหน้านั้น (มีคำว่า `ทุก BL` = ทุก B/L)
- ยอดรวมเทียบกับ MANIFEST GRAND TOTAL และ B/L ที่มีใน ENTER แต่ตกหล่นจาก MANIFEST (วิกฤต)

สี: แดง ⚠ = ไม่ตรง · ส้ม = ต้องให้คนยืนยัน · เหลือง = ไม่มี ENTER เทียบ · เขียว = ผ่าน (คอลัมน์สรุป)

## ใช้งานแบบโปรแกรม Python (ในเครื่อง)

`build_edi.py` อ่าน ENTER ได้หลายฟอร์ม (DSV, Penanshin, CMS / THE M CORP, AGN, Quality, Cargoport ฯลฯ)

```bash
pip install -r requirements.txt
python build_edi.py --manifest MANIFEST.xls --enter ENTER.pdf --outdir output --shed 0141
```

หรือวาง MANIFEST + ENTER ไว้ในโฟลเดอร์ `input/` แล้วรัน `python build_edi.py` (หาไฟล์ให้อัตโนมัติ)

## วิธีเปิดเว็บให้เพื่อนใช้ (GitHub Pages)

1. ขึ้น GitHub: https://github.com/Rattanao/check-edi-DSV → **Settings → Pages**
2. **Build and deployment → Source:** *Deploy from a branch*
3. **Branch:** `main` และโฟลเดอร์ `/ (root)` → กด **Save**
4. รอ ~1 นาที ลิงก์จะขึ้นที่หัวหน้า Pages: `https://rattanao.github.io/check-edi-DSV/`
   (หน้าแรกของ repo จะเด้งไป `docs/` เอง)
5. ส่งลิงก์ให้เพื่อนได้เลย — แก้โค้ดแล้ว `git push` หน้าเว็บอัปเดตเองภายในไม่กี่นาที

## หมายเหตุเรื่องข้อมูลลูกค้า

`input/`, `output/` และไฟล์ `.xls/.xlsx/.pdf` ถูก git-ignore ไว้ — repo นี้เก็บเฉพาะโปรแกรม
ไม่เก็บเอกสารจริงของลูกค้า
