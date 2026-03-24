# Power Platform — Servislar Haqida To'liq Ma'lumot

## Loyiha haqida

**Power Platform** — mikroservislar arxitekturasiga qurilgan katta miqyosli o'quv va savdo platformasi.  
Barcha servislar **Python 3.12 + FastAPI** asosida yozilgan, **PostgreSQL** (PgBouncer orqali), **Redis**, **RabbitMQ** (asinxron xabarlar), **MinIO/S3** (fayllar) ishlatadi.

Tizim `power-fastapi-gateway` (port **9000**) orqali umumiy kirish nuqtasiga ega — barcha tashqi so'rovlar o'sha yerdan tegishli servislarga yo'naltiriladi.  
Infratuzilma (`power-infra`) Docker Compose va Nginx yordamida boshqariladi, monitoring uchun Prometheus / Grafana / Loki / Jaeger ishlatiladi.

---

## Umumiy arxitektura

```
Internet
   │
Nginx (80/443, TLS, rate-limit)
   │
power-fastapi-gateway :9000
   │
   ├── /api/v1/auth            → power-auth-fastapi-service         :8000
   ├── /api/v1/users           → power-user-fastapi-service          :8001
   ├── /api/v1/products        → power-product-fastapi-service       :8002
   ├── /api/v1/cart            → power-cart-fastapi-service          :8003
   ├── /api/v1/orders          → power-order-fastapi-service         :8004
   ├── /api/v1/payments        → power-payment-fastapi-service       :8005
   ├── /api/v1/notifications   → power-notification-fastapi-service  :8006
   ├── /api/v1/media-storage   → power-media-storage-fastapi-service :8007
   ├── /api/v1/social-interaction → power-social-interaction-fastapi-service :8008
   ├── /api/v1/translations    → power-translation-fastapi-service   :8009
   ├── /api/v1/analytics       → power-analytics-fastapi-service     :8010
   ├── /api/v1/media           → power-media-fastapi-service         :8011
   └── /api/v1/follows + /social → power-social-fastapi-service     :8012
```

---

## Umumiy kutubxonalar (HTTP server yo'q)

### `power-fastapi-core`
Barcha mikroservislar o'rnatadigan umumiy platforma kutubxonasi.

| Modul | Vazifa |
|-------|--------|
| `app` | `create_app()` fabrikasi, `/health` endpoint |
| `database` | Async SQLAlchemy + `get_db` dependency |
| `repository` | `BaseRepository[T]` — universal async CRUD |
| `responses` | `DataResponse`, `PaginatedResponse`, `SuccessResponse` |
| `security` | JWT tekshirish, `get_current_user_id` |
| `messaging` | RabbitMQ producer/consumer, `publish_event()`, `ServiceEventConsumer` |
| `cache` | Redis kesh yordamchilari |
| `exceptions` | `NotFoundError`, `ForbiddenError`, `ConflictError`, `ValidationError` |
| `config` | `Settings` (pydantic-settings), `TagInfo`, `build_tags()` |
| `middleware` | HTTP request logging, CORS, GZip |
| `logging` | Strukturaviy loglash (`get_logger`) |
| `utils` | `slugify`, `handle_exceptions`, boshqalar |

---

### `power-quiz-core`
Quiz/exam mantiqini bajaradigan sof Python kutubxonasi (HTTP/DB yo'q). `power-quiz-fastapi-service` tomonidan ishlatiladi.

- **15 turdagi savol:** bir tanlama, ko'p tanlama, to'ldiring, tartiblang, mos keltiring va boshqalar
- **Xususiyatlar:** qisman baholash, salbiy ball, urinish limiti, savollarni aralashtirish
- **Modullar:** `engines`, `question_registry`, `schemas`, `validators`, `types`, `utils`

---

### `power-telegram-core`
`aiogram 3` asosida qurilgan Telegram bot framework kutubxonasi.

- **Modullar:** `handlers`, `middlewares`, `keyboards`, `fsm`, `filters`, `i18n`, `payments`, `broadcast`, `rate_limiter`, `deep_linking`, `web_apps`, `commands`, `support`

---

## Servis shabloni

### `power-service-fastapi-template`
Yangi mikroservis yaratish uchun tayyor shablon (`git clone` + o'zgartirish).

- Demo model, demo router, standart papka tuzilmasi
- `.env.example`, `Dockerfile`, `alembic.ini`, `ecosystem.config.js` tayyor holda keladi
- `app/events/` (RabbitMQ consumer), `migrations/`, `app/config/settings.py` shablonlari

---

## Gateway va Infratuzilma

### `power-fastapi-gateway` — port **9000**
Barcha tashqi trafikni URL prefix bo'yicha koordinatsiya qiluvchi teskari proksi.  
`httpx` orqali asinxron proxylash. Ma'lumotlar bazasi yo'q.

- **Router:** `app/routes/proxy_router.py`

---

### `power-infra`
Docker Compose asosida butun platformani boshqaradi.

- **Compose fayllar:** `docker-compose.yml` (infratuzilma), `docker-compose.dev.yml` (UI toollar), `docker-compose.full.yml` (hamma servislar)
- **Komponentlar:** Nginx, PgBouncer (:6432), Redis, MinIO, RabbitMQ, Prometheus, Grafana, Loki, Jaeger

---

## A. Identifikatsiya va Autentifikatsiya

### `power-auth-fastapi-service` — port **8000**
JWT asosida markaziy autentifikatsiya xizmati. Barcha token issuance/refresh, OAuth ijtimoiy login, sessiya va API kalit boshqaruvi.

| Model | Jadval | Tavsif |
|-------|--------|--------|
| `User` | `auth_users` | Asosiy foydalanuvchi (email, password hash) |
| `Session` | `auth_sessions` | Faol sessiyalar |
| `RefreshToken` | `auth_refresh_tokens` | Refresh token zanjiri |
| `ApiKey` | `auth_api_keys` | Dasturchilar uchun API kalitlar |
| `SocialAccount` | `auth_social_accounts` | OAuth ijtimoiy ulashlar (Google, GitHub...) |
| `PasswordReset` | `auth_password_resets` | Parolni tiklash tokenlari |
| `VerificationToken` | `auth_verification_tokens` | Email tasdiqlash tokenlar |

**Routerlar:** `auth_router`, `token_router`, `session_router`, `account_router`, `api_key_router`, `provider_router`, `exchange_router`, `password_router`, `verification_router`

---

### `power-user-fastapi-service` — port **8001**
Foydalanuvchi profili, sozlamalari va faollik tarixi.

| Model | Tavsif |
|-------|--------|
| `UserProfile` | Ism, avatar, cover, bio, tug'ilgan sana |
| `UserSettings` | Bildirishnoma, maxfiylik, til, mavzu |
| `UserActivityLog` | Harakatlar tarixi (login, upload...) |
| `UserSocialLink` | Instagram, Twitter, GitHub va boshqa havolalar |

**Routerlar:** `user_router`, `admin_router`, `settings_router`, `social_link_router`

---

### `power-organization-fastapi-service` — port *belgilanmagan*
Ko'p tenantli tashkilot boshqaruvi. Rol asosida a'zolik va email orqali taklif tizimi. RabbitMQ consumer orqali `user.deleted` hodisasini kuzatadi.

| Model | Jadval | Tavsif |
|-------|--------|--------|
| `Organization` | `org_organizations` | Tashkilot (maktab, do'kon, kompaniya). slug, subdomain, custom_domain |
| `Membership` | `org_memberships` | Foydalanuvchi a'zoligi: OWNER / ADMIN / MANAGER / MEMBER |
| `Invitation` | `org_invitations` | Email orqali taklif, token, TTL, holat |

**Routerlar:** `organization_router` (CRUD, a'zolik, ownership transfer), `invitation_router` (yuborish, qabul qilish, bekor qilish)  
**RabbitMQ:** `org.events` queue — `user.deleted` hodisasini tutib, barcha a'zoliklarni bloklayd

---

### `power-integration-fastapi-service` — port *belgilanmagan*
Foydalanuvchi OAuth integratsiyalarini (GitHub, Google) saqlaydi.

| Model | Tavsif |
|-------|--------|
| `UserIntegration` | Provider, access/refresh token, scope, status |

**Routerlar:** `integration_router`, `github_router`, `google_router`

---

## B. E-Tijorat

### `power-product-fastapi-service` — port **8002**
Mahsulot katalogi — SKU variantlar, brend, rasm galereyasi.

| Model | Tavsif |
|-------|--------|
| `Product` | Mahsulot (nom, narx, tavsif, holat) |
| `ProductVariant` | SKU variantlar (hajm, rang, narx) |
| `ProductImage` | Mahsulot rasmlari |
| `Brand` | Brend ma'lumotlari |

**Routerlar:** `product_router`, `admin_product_router`, `variant_router`, `brand_router`, `image_router`

---

### `power-cart-fastapi-service` — port **8003**
Foydalanuvchi savatchasi — bitta faol savatcha, elementlar, miqdor boshqaruvi.

| Model | Tavsif |
|-------|--------|
| `Cart` | Foydalanuvchining faol savatchasi |
| `CartItem` | Savatcsha elementi (mahsulot, variant, miqdor) |

**Routerlar:** `cart_router` (qo'shish, yangilash, o'chirish, tozalash)

---

### `power-order-fastapi-service` — port **8004**
Buyurtma hayot-davri: `pending → confirmed → processing → shipped → delivered | cancelled`.

| Model | Tavsif |
|-------|--------|
| `Order` | Buyurtma sarlavhasi (holat, umumiy summa, manzil) |
| `OrderItem` | Buyurtma qatorlari (mahsulot, variant, miqdor, narx) |

**Routerlar:** `order_router`

---

### `power-payment-fastapi-service` — port **8005**
To'lov ishlovi va qaytarish. Holat: `pending → processing → completed | failed | refunded`.

| Model | Tavsif |
|-------|--------|
| `Payment` | To'lov (summa, provider, holat) |
| `PaymentEvent` | To'lov voqealar tarixi (webhook log) |

**Routerlar:** `payment_router`

---

### `power-subscription-fastapi-service` — port *belgilanmagan*
Obuna rejalari va foydalanuvchi obunasini boshqarish.

| Model | Tavsif |
|-------|--------|
| `SubscriptionPlan` | Reja (nom, narx, davr, xususiyatlar) |
| `Subscription` | Foydalanuvchi obunasi (holat, boshlanish/tugash vaqti) |
| `SubscriptionHistory` | Obuna o'zgarishlari tarixi |

**Routerlar:** `subscription_router` (foydalanuvchi), `admin_router` (admin)

---

### `power-promo-fastapi-service` — port *belgilanmagan*
Promo-kod yaratish va foydalanish statistikasi.

| Model | Tavsif |
|-------|--------|
| `PromoCode` | Kod (chegirma, limit, amal qilish muddati) |
| `PromoUsage` | Kimlar ishlatgani tarixi |

**Routerlar:** `promo_router`

---

### `power-referral-fastapi-service` — port *belgilanmagan*
Ko'p darajali referal dasturi — daraxt tuzilmasi, mukofot tartiblari.

| Model | Tavsif |
|-------|--------|
| `Referral` | Referal havolasi |
| `ReferralCode` | Unikal taklif kodi |
| `ReferralTree` | Ko'p darajali daraxt tuzilmasi |
| `ReferralLevel` | Darajalar (1-level, 2-level...) |
| `ReferralReward` | Mukofot qoidalari |
| `ReferralConfig` | Umumiy konfiguratsiya |

**Routerlar:** `referral_router`

---

## C. LMS / Ta'lim

### `power-course-fastapi-service` — port *belgilanmagan*
Kurs katalogi: Kurs → Bo'lim (chapter) → Dars iyerarxiyasi.

| Model | Tavsif |
|-------|--------|
| `Course` | Kurs (nom, tavsif, muallif, narx, holat) |
| `CourseSection` | Kurs bo'limi/boblar |
| `CourseLesson` | Alohida darslar (video, matn, vaqt) |

**Routerlar:** `course_router`, `section_router`, `lesson_router`

---

### `power-quiz-fastapi-service` — port *belgilanmagan*
Savol bazasi, vaqtlangan sessiyalar, javob qabul qilish, avtomatik baholash. `power-quiz-core` kutubxonasi asosida.

| Model | Tavsif |
|-------|--------|
| `Quiz` | Quiz (nom, vaqt limiti, o'tish bali) |
| `Exam` | Imtihon konfiguratsiyasi |
| `Question` | Savol (matn, tur, ball) |
| `Option` | Savol variantlari |
| `Session` | Foydalanuvchi quiz sessiyasi |
| `QuizSubmission` | Topshiriq |
| `SubmissionAnswer` | Har bir savol javobi |
| `Result` | Yakuniy natija va baholash |

**Routerlar:** `exam_router`, `question_router`, `session_router`, `result_router`

---

### `power-enrollment-fastapi-service` — port *belgilanmagan*
Kurslarga yozilish va yozilish holatini boshqarish.

| Model | Tavsif |
|-------|--------|
| `Enrollment` | Foydalanuvchi–Kurs bog'liqliği (holat, yozilgan sana) |

**Routerlar:** `enrollment_router`

---

### `power-progress-fastapi-service` — port *belgilanmagan*
Foydalanuvchi o'quv progressini kurs va dars darajasida kuzatish.

| Model | Tavsif |
|-------|--------|
| `CourseProgress` | Kurs butun progressi (%) |
| `LessonProgress` | Alohida darsning progressi (ko'rilgan, vaqt) |

**Routerlar:** `progress_router`

---

### `power-certificate-fastapi-service` — port *belgilanmagan*
Kurs yakunlaganda sertifikat berish, saqlash va olish.

| Model | Tavsif |
|-------|--------|
| `Certificate` | Sertifikat (foydalanuvchi, kurs, chiqarilgan sana, fayl URL) |

**Routerlar:** `certificate_router`

---

## D. Kontent va Ijtimoiy

### `power-category-fastapi-service` — port **8019**
Ierarxik kategoriya daraxti va polimorfik entity–kategoriya/tag bog'lash tizimi.

| Model | Tavsif |
|-------|--------|
| `Category` | Ierarxik kategoriya (parent, slug, tartib) |
| `Tag` | Global teglar |
| `Entity` | Entity–kategoriya va entity–tag ko'p tomonlama bog'lash |

**Routerlar:** `category_router`, `tag_router`, `entity_router`

---

### `power-tag-fastapi-service` — port *belgilanmagan*
Global teglash tizimi — universal tag CRUD va entity–tag ulanishlari.

| Model | Tavsif |
|-------|--------|
| `Tag` | Global teg (nom, slug) |
| `EntityTag` | Ko'p tomonlama entity–tag ulanishi |

**Routerlar:** `tag_router`

---

### `power-post-fastapi-service` — port *belgilanmagan*
Blog/kontent postlar va kategoriyaga bog'lash.

| Model | Tavsif |
|-------|--------|
| `Post` | Post (sarlavha, matn, holat, muallif) |
| `PostCategory` | Post uchun kategoriya |
| `PostCategoryLink` | Ko'p tomonlama post–kategoriya ulanishi |

**Routerlar:** `post_router`

---

### `power-review-fastapi-service` — port *belgilanmagan*
Ishonar sharh, ovoz berish va suiistifmolni (abuse) modalatsiya qilish.

| Model | Tavsif |
|-------|--------|
| `Review` | Har qanday entity'ga sharh (1–5 yulduz, matn) |
| `ReviewVote` | Sharh uchun up/down ovoz |
| `ReviewReport` | Suiistifmoil shikoyati va moderatsiya |

**Routerlar:** `review_router`

---

### `power-social-fastapi-service` — port **8012**
Ijtimoiy grafik — follow/unfollow, shaxsiy hisob uchun tasdiqlash so'rovi, blok.

| Model | Tavsif |
|-------|--------|
| `Follow` | Kuzatish ulanishi (follower → following) |
| `Block` | Bloklash ulanishi |

**Routerlar:** `follow_router`, `follow_request_router`, `block_router`

---

### `power-social-interaction-fastapi-service` — port **8008**
Umumiy kontent interaksiyalari: sharh, layk, reyting, ko'rishlar.

| Model | Tavsif |
|-------|--------|
| `Comment` | Treed sharhlar (ixtiyoriy parent) |
| `Like` | Toggl layk (har qanday entity uchun) |
| `Rating` | 1–5 yulduz reytingi |
| `View` | Ko'rish / taassurot kuzatish |

**Routerlar:** `comment_router`, `like_router`, `rating_router`, `view_router`

---

### `power-chat-fastapi-service` — port *belgilanmagan*
Real-vaqt chat — xona boshqaruvi, a'zo rollari, xabar treding, reaksiyalar. WebSocket qo'llab-quvvatlanadi.

| Model | Tavsif |
|-------|--------|
| `Room` | Chat xonasi (tip, nomi, sozlamalar) |
| `RoomMember` | Xona a'zosi va roli |
| `Message` | Xabar (matn, media, javob) |
| `MessageReaction` | Emoji reaksiyalari |

**Routerlar:** `chat_router` (REST), `ws_router` (WebSocket)

---

## E. Bildirishnomalar va Muloqot

### `power-notification-fastapi-service` — port **8006**
Ilovalar ichidagi bildirishnomalar yetkazish, o'qilgan/o'qilmagan holat, foydalanuvchi sozlamalari.

| Model | Tavsif |
|-------|--------|
| `Notification` | Bildirishnoma (tur, sarlavha, matn, o'qilgan holat) |
| `NotificationPreference` | Turlar bo'yicha foydalanuvchi sozlamalari |

**Turlari:** `info`, `warning`, `success`, `error`, `order_update`, `payment`, `social`, `system`  
**Routerlar:** `notification_router`

---

### `power-notifier-fastapi-service` — port *belgilanmagan*
Web Push bildirishnomalarini yetkazish va push-obuna boshqaruvi.

| Model | Tavsif |
|-------|--------|
| `PushSubscription` | Foydalanuvchi push-endpoint va kalitlar |

**Routerlar:** `notifier_router`, `push_router`

---

### `power-email-fastapi-service` — port **8013**
Tranzaksion email yuborish — SMTP, SendGrid, SES providers; shablon boshqaruvi; tarix.

| Model | Tavsif |
|-------|--------|
| `EmailTemplate` | Email shabloni (tur, mavzu, HTML/matn tanasi) |
| `EmailLog` | Yuborish tarixi (holat, provider, xato) |

**Routerlar:** `send_router`, `template_router`, `history_router`

---

### `power-sms-fastapi-service` — port **8014**
SMS yuborish va OTP tekshirish — Eskiz, Playmobile, Mock provayderlar.

| Model | Tavsif |
|-------|--------|
| `SmsLog` | SMS yuborish tarixi |
| `OtpCode` | OTP kod (telefon raqam, kod, TTL, holat) |

**Routerlar:** `sms_router`

---

## F. Fayl va Media

### `power-file-upload-fastapi-service` — port **8015**
Fayl yuklash — MIME-tur tekshiruvi, hajm limiti, yumshoq o'chirish.

| Model | Tavsif |
|-------|--------|
| `FileUpload` | Yuklangan fayl (nom, tur, hajm, yo'l, yuklagan foydalanuvchi) |

**Routerlar:** `file_router`

---

### `power-media-fastapi-service` — port **8011**
Media kontent metama'lumotlar katalogi (maqolalar, rasmlar, videolar) — teglash, kategoriyalash, nashr holati.

| Model | Tavsif |
|-------|--------|
| `MediaItem` | Media element (tur, sarlavha, URL, holat, teglar) |

**Routerlar:** `media_router`

---

### `power-media-storage-fastapi-service` — port **8007**
Ikkilik fayl saqlash — lokal fayl tizimi yoki S3-mos backend (MinIO); imzolangan URL yaratish.

| Model | Tavsif |
|-------|--------|
| `File` | Saqlangan fayl (nom, hajm, MIME, bucket, kalit, holat) |

**Routerlar:** `file_router`

---

## G. Sun'iy Intellekt va Qidiruv

### `power-ai-fastapi-service` — port *belgilanmagan*
LLM gateway — matn generatsiya, tool/function calling, ko'p provayder marshrutlash, xarajat kuzatish.

| Model | Tavsif |
|-------|--------|
| `AiGeneration` | Generatsiya so'rovi va natijasi |
| `AiProviderRequest` | Provayderga so'rov tafsilotlari |
| `AiToolCall` | Tool/function chaqiruv qaydi |

**Provayderlar:** `app/providers/`; **Routerlar:** `ai_router`

---

### `power-vector-fastapi-service` — port *belgilanmagan*
Vektor do'kon — hujjat qabul qilish, bo'laklash, embedding, kolleksiya boshqaruvi, semantik qidiruv.

| Model | Tavsif |
|-------|--------|
| `VectorDocument` | Manba hujjat |
| `VectorChunk` | Hujjat bo'laklari |
| `VectorCollection` | Kolleksiya (namespace) |
| `VectorIndexJob` | Indekslash ish tarixi |

**Modullar:** `app/chunking/`, `app/embeddings/`, `app/extractors/`, `app/vectorstores/`  
**Routerlar:** `vector_router`

---

### `power-websearch-fastapi-service` — port *belgilanmagan*
Veb-qidiruv proksi — tashqi qidiruv provayderlarga so'rov yuborish va natijalarni saqlash.

| Model | Tavsif |
|-------|--------|
| `WebSearchRequest` | Qidiruv so'rovi (kalit so'z, provayderlar) |
| `WebSearchResult` | Qidiruv natijalari (URL, sarlavha, parcha) |

**Routerlar:** `search_router`

---

### `power-recommendation-fastapi-service` — port *belgilanmagan* *(ishlanmoqda)*
Kontentva mahsulot tavsiya mexanizmi — hali ishlab chiqilmagan (stub).

---

### `power-search-fastapi-service` — port *belgilanmagan* *(ishlanmoqda)*
Platforma bo'ylab birlikli qidiruv — hali ishlab chiqilmagan (stub).

---

## H. Platforma Operatsiyalari

### `power-analytics-fastapi-service` — port **8010**
Hodisalar kuzatish — `page_view`, `purchase`, `click` va boshqalar yig'ish hamda agregatsiya.

| Model | Tavsif |
|-------|--------|
| `AnalyticsEvent` | Hodisa (tur, entiti, foydalanuvchi, meta, vaqt) |

**Routerlar:** `event_router`

---

### `power-audit-fastapi-service` — port *belgilanmagan*
Audit yo'li — entity darajasida maydon o'zgarishlari va xavfsizlik hodisalarini qayd etish.

| Model | Tavsif |
|-------|--------|
| `AuditEvent` | Umumiy audit hodisas |
| `AuditEntityChange` | Maydon darajasida o'zgarish (eski → yangi qiymat) |
| `AuditSecurityEvent` | Xavfsizlik hodisalari (login, ruxsat o'zgarishi) |

**Routerlar:** `audit_router`

---

### `power-config-fastapi-service` — port *belgilanmagan*
Dinamik konfiguratsiya va xususiyat bayroqlari (feature flags) — foydalanuvchi/org/foiz bo'yicha nishonlash.

| Model | Tavsif |
|-------|--------|
| `AppSetting` | Ilova parametri (kalit–qiymat) |
| `FeatureFlag` | Xususiyat bayrog'i (yoqilgan/o'chirilgan) |
| `FeatureFlagTarget` | Nishonlash qoidasi (foydalanuvchi, org, foiz) |

**Routerlar:** `config_router`

---

### `power-translation-fastapi-service` — port **8009**
Markazlashgan i18n saqlash — bir nechta lokale (uz/ru/en) uchun kalit–qiymat tarjimalar.

| Model | Tavsif |
|-------|--------|
| `TranslationKey` | Tarjima kaliti (nom, kontekst, tavsif) |
| `Translation` | Kalit + lokale uchun qiymat |

**Routerlar:** `key_router`, `translation_router`

---

### `power-support-fastapi-service` — port *belgilanmagan*
Mijozlarga yordam tizimi — ticketlar, xabarlar, fayllar, FAQ maqolalari.

| Model | Tavsif |
|-------|--------|
| `SupportTicket` | Yordam so'rovi (mavzu, holat, ustuvorlik) |
| `SupportMessage` | Ticket xabar tarmog'i |
| `SupportAttachment` | Fayl biriktirishlar |
| `SupportSubject` | Mavzular taksonomiyasi |
| `Faq` | FAQ maqolasi |
| `FaqCategory` | FAQ kategoriyasi |

**Routerlar:** `support_router`

---

### `power-project-fastapi-service` — port *belgilanmagan*
Loyiha boshqaruvi (ishlanmoqda).

---

## Telegram

### `power-telegram-bot-template`
Ishlab chiqishga tayyor Telegram bot shahloni. `power-telegram-core` kutubxonasi asosida.

- **Imkoniyatlar:** Majburiy kanal obunasi, Telegram Stars to'lovlar, ko'p til (uz/ru/en), admin panel, Redis FSM holati, tezlik cheklash, Docker
- **Tuzilma:** `bot/handlers/`, `bot/locales/`, `bot/config.py`, `bot/setup.py`

---

## Port jadvali

| Port | Servis |
|------|--------|
| **8000** | `power-auth-fastapi-service` |
| **8001** | `power-user-fastapi-service` |
| **8002** | `power-product-fastapi-service` |
| **8003** | `power-cart-fastapi-service` |
| **8004** | `power-order-fastapi-service` |
| **8005** | `power-payment-fastapi-service` |
| **8006** | `power-notification-fastapi-service` |
| **8007** | `power-media-storage-fastapi-service` |
| **8008** | `power-social-interaction-fastapi-service` |
| **8009** | `power-translation-fastapi-service` |
| **8010** | `power-analytics-fastapi-service` |
| **8011** | `power-media-fastapi-service` |
| **8012** | `power-social-fastapi-service` |
| **8013** | `power-email-fastapi-service` |
| **8014** | `power-sms-fastapi-service` |
| **8015** | `power-file-upload-fastapi-service` |
| **8019** | `power-category-fastapi-service` |
| **9000** | `power-fastapi-gateway` (umumiy kirish) |

---

## Statistika

| Kategoriya | Soni |
|------------|------|
| Umumiy kutubxonalar | 3 (`power-fastapi-core`, `power-quiz-core`, `power-telegram-core`) |
| Faol mikroservislar | 40+ |
| Jami modellar (jadvallar) | 70+ |
| Portga bog'langan servislar | 17 |
| Infratuzilma | 1 (`power-infra`) |
| Shablon | 2 (`power-service-fastapi-template`, `power-telegram-bot-template`) |
