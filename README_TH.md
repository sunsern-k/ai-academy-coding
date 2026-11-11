# AI Academy Coding

เดโม Cursor ที่ออกแบบและสร้างเว็บแอป **Todo List** จาก [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md) เอเจนต์สองตัวใช้บอร์ดงานร่วมกัน คำสั่ง git, เวลาประทับ, ติดตั้ง, ทดสอบ และสตาร์ตแอปใช้ได้ทั้งบน macOS zsh และ Windows PowerShell

ผลิตภัณฑ์คือแอป Vite + TypeScript: รายการ todo เดียว เก็บใน `localStorage` ทดสอบด้วย Vitest ผู้ใช้เพิ่ม ดู แก้ไข เปลี่ยนสถานะ และลบรายการได้ แหล่งความจริงของพฤติกรรมคือไฟล์ความต้องการ เอเจนต์จะไม่เพิ่มฟีเจอร์ที่ไม่ได้ระบุไว้

ยังไม่มีอะไรใน `src/` จนกว่า builder จะรัน รีพอซิทอรีนี้เริ่มต้นด้วยเอเจนต์ สกิล และเอกสารความต้องการ

ฉบับภาษาอังกฤษ: [README.md](README.md)

## เอเจนต์

| เอเจนต์ | บทบาท |
| --- | --- |
| `app-architect` | อ่านความต้องการ สร้างกิตบรานช์ (`cursor/todo-list` หรือชื่อที่มี timestamp หากบรานช์นั้นมีอยู่แล้ว) และเขียน `docs/DESIGN.md`, `docs/TASKS.md`, `docs/GITBRANCH.md`, และ `docs/RUN.md` หลังแต่ละรายงาน QA จะทำเครื่องหมายงานเป็น `done` หรือเพิ่มงานแก้ เอเจนต์นี้ไม่เขียนโค้ดแอป |
| `app-builder` | ลงมือทำงาน `open` **หนึ่ง** งาน รัน Vitest หนึ่งครั้ง แล้วเขียน `docs/QA-REPORT-<id>.md` งานที่ผ่านจะเป็น `implemented` จากนั้น architect เป็นผู้ทำเครื่องหมายเป็น `done` |

สกิลอยู่ที่ `.cursor/skills/` กติกาเชลล์สำหรับ Windows และ macOS อยู่ใน `.cursor/skills/app-delivery-contracts/SKILL.md` (**Shell commands**) ให้ใช้ Node สำหรับเวลา UTC เมื่อมี Node บน `PATH` รูปแบบ `date` / PowerShell เป็นทางเลือกสำรอง งานหนึ่งแก้ได้สูงสุดสองรอบ (`T1-FIX1` แล้วตามด้วย `T1-FIX2`)

## สิ่งที่ต้องมีก่อนเริ่ม

- [Cursor](https://cursor.com)
- Git
- Node.js บน `PATH` (จำเป็นเมื่อ builder ติดตั้งและทดสอบแอป และสำหรับเวลา UTC ที่ใช้ข้ามแพลตฟอร์ม)

บน macOS เชลล์ของ Cursor มักเป็น zsh บน Windows มักเป็น PowerShell ควรใช้ `node -e …` สำหรับเวลาประทับบนทั้งสองระบบ Git Bash บน Windows ใช้ `date` แบบ Unix ได้ แต่ PowerShell ใช้ไม่ได้ (`date` ที่นั่นคือ `Get-Date`)

## รันเดโม

เปิดโฟลเดอร์นี้ใน Cursor แล้วเปิดแชต Agent ใหม่

1. ออกแบบแอป:

   ```text
   /app-architect
   ```

   พาธความต้องการเริ่มต้นคือ `docs/REQUIREMENTS.md` หากต้องการไฟล์อื่น: `/app-architect path/to/file.md`

2. สร้างงานถัดไป (ทำซ้ำจนกว่าบอร์ดจะไม่มีงาน `open`):

   ```text
   /app-builder
   ```

   ระบุงานเมื่อต้องการงานเฉพาะ เช่น `/app-builder T1` หลัง builder เขียนรายงาน QA ให้รัน `/app-architect` อีกครั้งเพื่อทำเครื่องหมายเป็น `done` หรือเพิ่มงานแก้

3. เมื่องานต้นฉบับทุกชิ้นเป็น `done` หรือหยุดหลังครบขีดจำกัดการแก้ ให้อ่าน **How to run** ใน `docs/RUN.md`

ขอให้ builder ทำงานขนานได้เฉพาะเมื่อระบุรหัสงาน (เช่น T1–T4) builder จะตรวจว่าไฟล์ไม่ทับกันก่อนเริ่มพร้อมกัน

เอเจนต์จะไม่ commit หรือ push จนกว่าคุณจะสั่ง

## รันแอป Todo

หลัง builder เพิ่ม `package.json` แล้ว ให้ใช้สามคำสั่งเดียวกันบน macOS และ Windows รันแยกทีละคำสั่ง PowerShell 5.1 ไม่รองรับ `&&`

```bash
npm install
npx vitest run
npm run dev
```

เปิด URL ที่สคริปต์ dev พิมพ์ ค่าเริ่มต้นของ Vite คือ [http://localhost:5173](http://localhost:5173)

## โครงสร้าง

| พาธ | จุดประสงค์ |
| --- | --- |
| `docs/REQUIREMENTS.md` | แหล่งผลิตภัณฑ์: ชื่อ สแต็ก เกณฑ์การยอมรับ |
| `docs/DESIGN.md` | เขียนโดย architect: โมดูล UI วิธีรัน |
| `docs/TASKS.md` | บอร์ดงาน (`open`, `implemented`, `done`, `failed`) |
| `docs/GITBRANCH.md` | ชื่อบรานช์และพาธความต้องการที่ builder ต้องใช้ |
| `docs/RUN.md` | เวลาเริ่ม บันทึก QA และสรุปวิธีรันท้ายสุด |
| `docs/QA-REPORT-<id>.md` | รายงานทดสอบต่อหนึ่งงาน |
| `.cursor/agents/` | `app-architect` และ `app-builder` |
| `.cursor/skills/` | เพลย์บุ๊กและสัญญาไฟล์/เชลล์ร่วม |
