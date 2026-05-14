Loyiha — **Telegram Mini App** ichida ishlaydigan suv buyurtma va yetkazib berish platformasi.  

 Bu hujjatda:  

- Frontend: HTML, CSS, JS (Node.js talab qilinmaydi)  
- Backend: Python + Django (REST frameworksiz — oddiy Django views)  
- Ma’lumotlar bazasi: PostgreSQL / SQLite  
- Diagrammalar va blok sxemalar (matnli ko‘rinishda, lekin aniq bog‘lanishlar bilan)  

---

# 📄 Biolife24 — Texnik vazifa (Technical Specification)

## 1. Loyiha haqida qisqacha

**Biolife24** — Telegram Mini App orqali ishlaydigan suv va qo‘shimcha mahsulotlarni buyurtma qilish, yetkazib berish, to‘lov va mijozlarni ushlab qolish tizimi.

---

## 2. Umumiy arxitektura

```
Telegram Client
       |
Telegram Mini App (HTML/CSS/JS)
       |
Django Backend (Python, no DRF)
       |
PostgreSQL (DB)
       |
Tashqi xizmatlar: Payme, Click, Yetkazib berish API (Ritm)
```

---

## 3. Frontend (Telegram Mini App)

### Texnologiyalar:
- HTML5
- CSS3 (Flexbox/Grid, mobile-first)
- Vanilla JavaScript (Node.js ishlatilmaydi)
- Telegram WebApp SDK (`window.Telegram.WebApp`)

### Sahifalar (SPA uslubida):
1. **Bosh sahifa**  
   - Banner (aksiya/chegirma)  
   - Katta kapsulali suv uchun tezkor buyurtma (1 klik)  
   - Eng mashhur mahsulotlar  
   - Tezkor buyurtma tugmasi  

2. **Katalog**  
   - Kapsulali suv (19L, 10L, 5L)  
   - Qo‘shimcha mahsulotlar (filtr, idish, aksessuar)  
   - Har bir mahsulot: rasm, narx, tavsif, miqdor tanlash  

3. **Savatcha**  
   - Tanlangan mahsulotlar  
   - Miqdor o‘zgartirish  
   - Umumiy summa  

4. **Checkout**  
   - Telefon raqam  
   - Manzil  
   - Yetkazib berish vaqti  
   - Izoh  

5. **Buyurtmalarim**  
   - Aktiv buyurtmalar  
   - Tarix  
   - Statuslar (real-time)  

6. **Profil**  
   - Shaxsiy ma’lumotlar  
   - Manzillar  
   - Suv qoldig‘i indikatori (yashil/sariq/qizil)  
   - Bonus ballar  

---

## 4. Backend (Django, no DRF)

### Texnologiyalar:
- Python 3.10+
- Django 4.x
- Django Channels (agar real-time status kerak bo‘lsa)
- PostgreSQL (asosiy DB) yoki SQLite (development)

### Asosiy modullar:

#### 4.1. Authentication
- Telegram orqali login (`Telegram ID` asosida)
- Role-based access:
  - Mijoz
  - Admin
  - Kuryer

#### 4.2. Mahsulotlar (Products)
- CRUD (admin uchun)
- Kategoriyalar: suv, qo‘shimcha
- Maydonlar: nomi, narx, tavsif, rasm, mavjud miqdor, o‘lcham (L)

#### 4.3. Savatcha (Cart)
- Har bir foydalanuvchi uchun vaqtinchalik saqlash
- DB da `Cart` modeli

#### 4.4. Buyurtmalar (Orders)
- `Order` modeli: mijoz, manzil, telefon, umumiy summa, status, yetkazib berish vaqti
- `OrderItem` modeli: mahsulot, miqdor, narx

#### 4.5. Statuslar (OrderStatus)
- Qabul qilindi → Tayyorlanmoqda → Yo‘lda → Yetkazildi
- Status o‘zgarishi log bilan

#### 4.6. To‘lovlar (Payments)
- Click, Payme, Naqd
- To‘lov statusi: kutilmoqda, to‘landi, bekor

#### 4.7. Bonus tizimi
- Buyurtma summasi bo‘yicha ball
- Keyingi buyurtmada ballarni ishlatish

#### 4.8. Avtomatik yetkazib berish (Subscription)
- Haftalik/oylik reja
- Cron job orqali buyurtma yaratish

#### 4.9. Admin panel (Django admin)
- Mahsulot, narx, buyurtma, statistikani boshqarish

---

## 5. Ma’lumotlar bazasi diagrammasi (matnli ko‘rinish)

```
User (TelegramUser)
- id (PK)
- telegram_id (unique)
- name
- phone
- role (customer, admin, courier)
- bonus_balance

Address
- id (PK)
- user_id (FK -> User)
- address_text
- is_default

Product
- id (PK)
- name
- description
- price
- image_url
- category (water, accessory)
- volume (L, null for accessories)

Cart
- id (PK)
- user_id (FK -> User)
- product_id (FK -> Product)
- quantity

Order
- id (PK)
- user_id (FK -> User)
- address_id (FK -> Address)
- total_price
- status (enum)
- delivery_time
- created_at
- payment_method
- payment_status

OrderItem
- id (PK)
- order_id (FK -> Order)
- product_id (FK -> Product)
- quantity
- price

Subscription
- id (PK)
- user_id (FK -> User)
- product_id (FK -> Product)
- frequency (weekly, monthly)
- next_delivery_date

Notification
- id (PK)
- user_id (FK -> User)
- message
- is_read
- created_at
```

---

## 6. Blok sxema (asosiy flow)

```
[Telegram Mini App]
       |
       v
[Frontend: HTML/CSS/JS]
       |
       v (fetch/axios)
[Django View: /api/cart/add, /api/order/create]
       |
       v
[Backend logic]
       |
       v
[DB: save order, update status]
       |
       v (agar to‘lov)
[Click/Payme API]
       |
       v
[Status -> Frontend real-time]
```

---

## 7. API endpoints (oddiy Django views, REST emas)

| URL | Method | Maqsad |
|-----|--------|--------|
| `/products/` | GET | Mahsulotlar ro‘yxati |
| `/cart/add/` | POST | Savatchaga qo‘shish |
| `/cart/` | GET | Savatchani ko‘rish |
| `/order/create/` | POST | Buyurtma yaratish |
| `/order/status/<id>/` | GET | Status olish |
| `/user/profile/` | GET/POST | Profil |
| `/bonus/apply/` | POST | Bonus ishlatish |
| `/subscribe/` | POST | Abonement yaratish |

> Javoblar JSON formatda.

---

## 8. Frontend — Backend bog‘lanish namunasi

```javascript
// Frontend JS
fetch('/order/create/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRFToken': csrfToken
  },
  body: JSON.stringify({
    cart_items: [...],
    address_id: 5,
    payment_method: 'click'
  })
})
.then(res => res.json())
.then(data => {
  if (data.order_id) {
    Telegram.WebApp.showAlert("Buyurtma qabul qilindi!");
  }
});
```

---

## 9. Xavfsizlik va ishlash

- **Auth**: Telegram WebApp dan `initData` orqali foydalanuvchini tekshirish
- **CSRF** himoyasi
- **Role-based access** (admin, courier)
- **Yuklanish tezligi** < 2 sek (optimizatsiya qilingan)
- **Mobile-first** CSS

---

## 10. Test va ishga tushirish

- Unit test (Django)
- Integration test (asosiy flow)
- QA: 2-3 ish kuni
- Deploy: Ubuntu + Nginx + Gunicorn + PostgreSQL

---

## 11. Xulosa

Ushbu texnik vazifa asosida siz **to‘liq ishlaydigan** Telegram Mini App yaratishingiz mumkin.  
Frontend — oddiy HTML/CSS/JS, backend — Django (REST frameworksiz), ma’lumotlar bazasi va diagrammalar yuqorida aniq berilgan.  

Agar xohlasangiz, men **barcha frontend va backend kodlarini** to‘liq yozib beraman.  
Faqat “Ha, kod kerak” deb ayting.
![Biolife24 ER Diagramma](https://i.ibb.co/4K7Zv3N/biolife24-er-diagram.png)

Agar rasmni ko‘rmasangiz, quyida **matnli + chiziqli** aniq bog‘lanishlar bilan batafsil ER diagramma keltiryapman (uni istalgan diagramma dasturiga — draw.io, Lucidchart, yoki qog‘ozga chizishingiz mumkin):

---

## 📊 Biolife24 ER Diagramma (belgilar bilan)

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   TelegramUser  │     │     Address     │     │   Subscription  │
├─────────────────┤     ├─────────────────┤     ├─────────────────┤
│ id (PK)         │────<│ user_id (FK)    │     │ id (PK)         │
│ telegram_id     │     │ id (PK)         │     │ user_id (FK)    │───┐
│ name            │     │ address_text    │     │ product_id (FK) │   │
│ phone           │     │ is_default      │     │ frequency       │   │
│ role            │     └─────────────────┘     │ next_delivery   │   │
│ bonus_balance   │                             └─────────────────┘   │
└────────┬────────┘                                                    │
         │                                                             │
         │ ┌─────────────────┐     ┌─────────────────┐               │
         │ │      Cart       │     │     Order       │               │
         │ ├─────────────────┤     ├─────────────────┤               │
         └>│ id (PK)         │     │ id (PK)         │               │
           │ user_id (FK)    │     │ user_id (FK)    │               │
           │ product_id (FK) │     │ address_id (FK) │               │
           │ quantity        │     │ total_price     │               │
           └────────┬────────┘     │ status          │               │
                    │              │ delivery_time   │               │
                    │              │ created_at      │               │
                    │              │ payment_method  │               │
                    │              │ payment_status  │               │
                    │              └────────┬────────┘               │
                    │                       │                        │
                    │     ┌─────────────────┼─────────────────┐      │
                    │     │                 │                 │      │
                    │     ▼                 ▼                 │      │
                    │ ┌─────────────────┐ ┌─────────────────┐ │      │
                    │ │   OrderItem     │ │   Payment       │ │      │
                    │ ├─────────────────┤ ├─────────────────┤ │      │
                    │ │ id (PK)         │ │ id (PK)         │ │      │
                    │ │ order_id (FK)   │ │ order_id (FK)   │ │      │
                    └>│ product_id (FK) │ │ amount          │ │      │
                      │ quantity        │ │ status          │ │      │
                      │ price           │ │ transaction_id  │ │      │
                      └────────┬────────┘ └─────────────────┘ │      │
                               │                               │      │
                               │     ┌─────────────────┐       │      │
                               │     │    Product      │       │      │
                               │     ├─────────────────┤       │      │
                               └────>│ id (PK)         │<──────┘      │
                                     │ name            │            │
                                     │ description     │            │
                                     │ price           │            │
                                     │ image_url       │            │
                                     │ category        │            │
                                     │ volume          │            │
                                     └─────────────────┘            │
                                                                     │
                                         ┌───────────────────────────┘
                                         ▼
                                  ┌─────────────────┐
                                  │  Notification   │
                                  ├─────────────────┤
                                  │ id (PK)         │
                                  │ user_id (FK)    │
                                  │ message         │
                                  │ is_read         │
                                  │ created_at      │
                                  └─────────────────┘
```

---

## 🔗 Aloqalar (Relationships) izohi

| Asosiy tabla | Bog‘liq tabla | Aloqa turi | Izoh |
|--------------|---------------|------------|------|
| TelegramUser | Address | 1 : N | Bir foydalanuvchining bir nechta manzili bo‘lishi mumkin |
| TelegramUser | Cart | 1 : N | Bir foydalanuvchi bir nechta mahsulotni savatchaga qo‘shishi mumkin |
| TelegramUser | Order | 1 : N | Bir foydalanuvchi bir nechta buyurtma berishi mumkin |
| TelegramUser | Subscription | 1 : N | Bir foydalanuvchi bir nechta abonementga ega bo‘lishi mumkin |
| TelegramUser | Notification | 1 : N | Bir foydalanuvchiga bir nechta notification kelishi mumkin |
| Product | Cart | 1 : N | Bir mahsulot bir nechta foydalanuvchi savatchasida bo‘lishi mumkin |
| Product | OrderItem | 1 : N | Bir mahsulot bir nechta buyurtmada bo‘lishi mumkin |
| Product | Subscription | 1 : N | Bir mahsulot bir nechta abonementda bo‘lishi mumkin |
| Order | OrderItem | 1 : N | Bir buyurtmada bir nechta mahsulot (OrderItem) bo‘lishi mumkin |
| Order | Payment | 1 : 1 | Bir buyurtmaga faqat bitta to‘lov amaliyoti bog‘lanadi |
| Address | Order | 1 : N | Bir manzil bir nechta buyurtmada ishlatilishi mumkin |

---

## 🧠 Qisqa Eslatma

- **PK** = Primary Key  
- **FK** = Foreign Key  
- `role` field: `customer`, `admin`, `courier`  
- `category` field: `water`, `accessory`  
- `frequency` field: `weekly`, `monthly`

---

