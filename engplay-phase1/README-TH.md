# EngPlay — Phase 1 (Account + Cloud Save)

รุ่นนี้ต่อยอดจากไฟล์ `index.html` เดิม โดยคงระบบเกม แบบฝึกหัด XP และสถิติเดิมไว้ และเพิ่ม Supabase Auth + Cloud Save

## ไฟล์ในโปรเจกต์
- `index.html` — เว็บแอป EngPlay ที่เพิ่มระบบสมาชิกและซิงก์ข้อมูล
- `supabase-config.js` — ใส่ Supabase Project URL และ anon/public key
- `supabase-schema.sql` — ตารางและ Row Level Security สำหรับข้อมูลผู้ใช้

## 1) สร้าง Supabase Project
1. ไปที่ https://supabase.com/ แล้วสมัคร/เข้าสู่ระบบ
2. กด New project ตั้งชื่อ `engplay` เลือก Region Singapore (ใกล้ไทย)
3. ตั้ง Database Password และเก็บไว้ให้ปลอดภัย
4. เมื่อ Project พร้อม ไปที่ Project Settings → API (หรือ API Keys)
5. คัดลอก Project URL และ `anon` / `publishable` key (ห้ามใช้ `service_role` / `secret` key)

## 2) สร้างตารางและสิทธิ์ความปลอดภัย
1. เปิด SQL Editor ใน Supabase
2. เปิดไฟล์ `supabase-schema.sql` คัดลอกทั้งหมดไปวาง
3. กด Run
4. ไปที่ Authentication → URL Configuration แล้วตั้ง Site URL เป็น URL ที่จะ Deploy จริงภายหลัง เช่น `https://ชื่อโปรเจกต์.pages.dev`
5. ใน Redirect URLs เพิ่ม `https://ชื่อโปรเจกต์.pages.dev/**` และ URL สำหรับทดสอบ เช่น `http://localhost:8788/**`

## 3) ใส่ค่าการเชื่อมต่อ
เปิด `supabase-config.js` แล้วแทนค่า placeholder:

```js
window.ENGPLAY_CONFIG = {
  SUPABASE_URL: "https://YOUR_PROJECT.supabase.co",
  SUPABASE_ANON_KEY: "YOUR_SUPABASE_ANON_OR_PUBLISHABLE_KEY"
};
```

Project URL และ anon/publishable key ใช้ใน frontend ได้เมื่อเปิด RLS ตาม SQL นี้แล้ว แต่ห้ามนำ `service_role` หรือ secret key มาใส่ในไฟล์นี้เด็ดขาด

## 4) ทดสอบในเครื่อง
ต้องเปิดผ่าน local web server ไม่ควรเปิด `index.html` ด้วย file:// โดยตรง

```bash
cd engplay-phase1
python3 -m http.server 8788
```

เปิด http://localhost:8788 → กดสมัครสมาชิก → ยืนยันอีเมล (ถ้าเปิด Email Confirmation) → เข้าสู่ระบบ → ทำแบบฝึกหัดให้มี XP → รอสักครู่แล้วดูสถานะ Cloud Save

ทดสอบข้ามอุปกรณ์: เปิดเว็บอีก Browser/อุปกรณ์ เข้าสู่ระบบบัญชีเดียวกัน ควรเห็น XP และความคืบหน้าเดียวกัน

## 5) Deploy ด้วย Cloudflare Pages (Free)
วิธีที่ง่ายที่สุดสำหรับโปรเจกต์ static:
1. สร้าง GitHub repository ชื่อ `engplay`
2. อัปโหลดไฟล์ทั้ง 3 ไฟล์นี้เข้า root ของ repository
3. เข้า https://dash.cloudflare.com/ → Workers & Pages → Create → Pages → Connect to Git
4. เลือก repository `engplay`
5. Framework preset เลือก None; Build command เว้นว่าง; Build output directory ใส่ `/` หรือ `.` ตามหน้าจอที่แสดง
6. กด Save and Deploy
7. นำ URL `*.pages.dev` ที่ได้ไปตั้งเป็น Site URL และ Redirect URL ใน Supabase ตามข้อ 2

## พฤติกรรมการซิงก์ในรุ่นนี้
- ไม่ได้ Login: เกมยังบันทึกใน Browser เครื่องนั้นด้วย localStorage
- สมัคร/เข้าสู่ระบบครั้งแรก: ถ้ายังไม่มีข้อมูลบน Cloud จะย้ายข้อมูล local เดิมขึ้น Cloud
- มีข้อมูลบน Cloud อยู่แล้ว: ข้อมูล Cloud จะเป็นข้อมูลหลักและโหลดมาแทนข้อมูล local ของ Browser นั้น
- หลัง Login: การเปลี่ยนแปลงจะถูกส่งขึ้น Cloud แบบหน่วงสั้น ๆ เพื่อไม่ยิงฐานข้อมูลทุกครั้งที่มีการเปลี่ยน state
- Chat history ยังไม่ซิงก์ใน Phase 1; จะทำเป็น Phase ถัดไป

## ข้อควรทราบก่อนเปิด Public
1. รุ่นนี้ทำ Cloud Save และ Account ก่อน ยังไม่ได้ย้าย AI ไป backend; อย่าใส่ Anthropic API key จริงในหน้าเว็บที่เปิด Public
2. ในหน้า AI Settings ของโค้ดเดิมมีโหมดใส่ API key ใน Browser ซึ่งไม่ปลอดภัยสำหรับเว็บสาธารณะ ควรปิด/ซ่อนโหมดนี้ก่อนเปิดใช้ AI จริง และ Phase 2 จะทำ AI proxy ฝั่ง server
3. ตั้งค่า Supabase Auth email confirmation และ rate limits ให้เหมาะสม
4. Cloud Save ในรุ่นแรกใช้ payload JSON ต่อผู้ใช้ เหมาะกับ MVP; หากข้อมูลโตขึ้นค่อยแยกตาราง chat, vocab และ learning events
