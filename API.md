# GOOT Resorts — مرجع الـ API الكامل | API Reference

> **الرابط الأساسي | Base URL:** `http://127.0.0.1:3000` (محلي) — وعلى الإنترنت يكون دومينك
> **كل الردود JSON** ما عدا ملفات التصدير (CSV)

---

## 🔑 المفتاحان | The two keys

| النوع | المصادقة | للوصول إلى |
|---|---|---|
| **عام** | لا شيء (Origin فقط لحماية CSRF) | الصحة، التوفر، الحجز، طلبات الخدمة |
| **أدمن** | هيدر `X-Admin-Key: <ADMIN_KEY>` | العملاء، إدارة الحجوزات، تصدير Excel |

`ADMIN_KEY` قيمته في `server/.env` — غيّريها قبل أي نشر حقيقي.

---

## 🟢 عام — بدون مفتاح | Public endpoints

### 1) `GET /api/health` — حالة النظام
```bash
curl http://127.0.0.1:3000/api/health
```
```json
{ "ok": true, "db": true }
```
`db:false` = قاعدة البيانات غير متصلة. كود 503 عند فشل الاتصال.

---

### 2) `GET /api/availability` — الفيلات المتاحة
```bash
curl "http://127.0.0.1:3000/api/availability?checkIn=2026-11-10&checkOut=2026-11-14"
```
```json
{ "unavailable": ["Sawsan"] }
```
| باراميتر | القاعدة |
|---|---|
| `checkIn` | تاريخ `YYYY-MM-DD` |
| `checkOut` | بعد checkIn |

❌ `400 Invalid date range` لو التواريخ غلط أو معكوسة.

---

### 3) `POST /api/bookings` — إنشاء حجز
```bash
curl -X POST http://127.0.0.1:3000/api/bookings \
  -H "Content-Type: application/json" \
  -H "Origin: http://127.0.0.1:3000" \
  -d '{"name":"نورا آل سعود","email":"noura@example.com","checkIn":"2026-11-10","checkOut":"2026-11-14","guests":4,"villa":"Sawsan"}'
```
**الرد 201:**
```json
{ "ok": true, "id": "c0533019-7279-...", "createdAt": "2026-09-30T23:50:23.136Z" }
```

**قواعد التحقق | Validation rules:**
| الحقل | القاعدة |
|---|---|
| `name` | مطلوب، ≤ 120 حرف |
| `email` | مطلوب، صيغة إيميل، ≤ 180 |
| `checkIn` / `checkOut` | `YYYY-MM-DD`، والمغادرة **بعد** الوصول |
| `guests` | رقم صحيح 1–20 |
| `villa` | واحد من: `Sawsan` `Jouri` `Orchid` `Fayrouz` `Lazurd` |

**أكواد الأخطاء:**
| كود | السبب |
|---|---|
| 400 | بيانات ناقصة أو غلط |
| 403 | Origin ناقص أو من موقع خارجي (حماية CSRF) |
| 409 | **الفيلا محجوزة في نفس التواريخ** (قاعدة منع التعارض) |
| 413 | الطلب أكبر من 20KB |
| 429 | تجاوز حد الطلبات (100 لكل 15 دقيقة) |

> 📌 تلقائيًا مع كل حجز: **العميل يُضاف/يتحدّث في جدول `customers`** (عدد الحجوزات + تاريخ آخر حجز).

---

### 4) `POST /api/service` — طلب خدمة/استفسار
```bash
curl -X POST http://127.0.0.1:3000/api/service \
  -H "Content-Type: application/json" \
  -H "Origin: http://127.0.0.1:3000" \
  -d '{"name":"أحمد","email":"ahmed@mail.com","message":"عايز حجز عائلي في ديسمبر"}'
```
**الرد 201:** `{ "ok": true, "id": "...", "createdAt": "..." }`
- `message` مطلوب ≤ 1200 — `email` و `phone` اختياريين.

---

## 🔴 أدمن — بمفتاح `X-Admin-Key` | Admin endpoints

> جرّبهم مباشرة من **لوحة الإدارة**: `http://127.0.0.1:3000/admin.html`

### 5) `GET /api/admin/customers` — قاعدة العملاء
```bash
curl http://127.0.0.1:3000/api/admin/customers -H "X-Admin-Key: $ADMIN_KEY"
# بحث اختياري:
curl "http://127.0.0.1:3000/api/admin/customers?q=نورا" -H "X-Admin-Key: $ADMIN_KEY"
```
```json
{ "count": 3, "customers": [
  { "id": "...", "name": "نورا آل سعود", "email": "noura@example.com",
    "phone": null, "country": null, "total_bookings": 1,
    "total_spend_sar": "0.00", "created_at": "...", "last_booking_at": "..." } ] }
```

### 6) `GET /api/admin/bookings` — كل الحجوزات
```bash
curl http://127.0.0.1:3000/api/admin/bookings -H "X-Admin-Key: $ADMIN_KEY"
```
```json
{ "count": 3, "bookings": [
  { "id": "...", "name": "نورا آل سعود", "email": "...", "villa": "Sawsan",
    "guests": 4, "check_in": "2026-11-10", "check_out": "2026-11-14",
    "status": "pending", "created_at": "..." } ] }
```
`status` تكون: `pending` | `confirmed` | `cancelled`

### 7) `PATCH /api/admin/bookings/:id` — تغيير حالة حجز
```bash
curl -X PATCH "http://127.0.0.1:3000/api/admin/bookings/<الحجز-ID>" \
  -H "Content-Type: application/json" \
  -H "Origin: http://127.0.0.1:3000" \
  -H "X-Admin-Key: $ADMIN_KEY" \
  -d '{"status":"confirmed"}'
```
الرد: `{ "ok": true, "booking": { "id": "...", "status": "confirmed" } }`
- ❌ 400 لو الحالة مش من الثلاثة المسموحة. ❌ 404 لو الـID مش موجود.

### 8) `GET /api/admin/export/customers.csv` — تصدير العملاء Excel
```bash
curl http://127.0.0.1:3000/api/admin/export/customers.csv -H "X-Admin-Key: $ADMIN_KEY" -o customers.csv
```
ملف CSV بـ**UTF-8 BOM** — يفتح في إكسل بالعربي مباشرة بدون تعديل.

### 9) `GET /api/admin/export/bookings.csv` — تصدير الحجوزات Excel
```bash
curl http://127.0.0.1:3000/api/admin/export/bookings.csv -H "X-Admin-Key: $ADMIN_KEY" -o bookings.csv
```

**أكواد أدمن:** 401 مفتاح ناقص/غلط (+تأخير 350ms ضد التخمين) · 403 Origin خارجي · 429 تجاوز الحد (60 لكل 15 دقيقة).

---

## 🧪 اختبار سريع من المتصفح | Quick browser test
افتح **Console** في أي صفحة من الموقع والصقي:
```js
// حجز جديد:
fetch("/api/bookings", {method:"POST", headers:{"Content-Type":"application/json"},
  body: JSON.stringify({name:"تجربة", email:"t@t.com", checkIn:"2027-01-10",
  checkOut:"2027-01-12", guests:2, villa:"Orchid"})}).then(r=>r.json()).then(console.log);

// العملاء (أدمن):
fetch("/api/admin/customers", {headers:{"X-Admin-Key":"ضعي-المفتاح-هنا"}})
  .then(r=>r.json()).then(console.log);
```

---

## 🚀 إعادة تشغيل السيرفر | Restart
```powershell
# 1) قاعدة البيانات (لو مقفولة):
& "C:\Users\HP\Downloads\` -Your-Uploaded-Images\pgsql\bin\pg_ctl.exe" `
  -D "C:\Users\HP\Downloads\ -Your-Uploaded-Images\pgdata" -o "-p 5432" start
# 2) سيرفر Node:
powershell -File "$env:USERPROFILE\start-goot.ps1"
# 3) التحقق:
curl http://127.0.0.1:3000/api/health
```
اختبارات الأمان الكاملة (15 اختبار):
```bash
cd server && node --test ../tests/server.test.mjs
```
