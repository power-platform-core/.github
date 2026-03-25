# OpenKnowledge — 5 ta service asosidagi yagona tizim arxitekturasi

## 1. Maqsad

Ushbu tizimning vazifasi — foydalanuvchi bergan **har qanday turdagi ma’lumotni** qabul qilish, qayta ishlash, semantik ko‘rinishga o‘tkazish, saqlash va keyinchalik so‘rov bo‘yicha **eng mos natijalarni** qaytarishdir.

Tizim quyidagi ma’lumot turlari bilan ishlay olishi kerak:

- oddiy text
- PDF, DOCX, CSV va boshqa hujjatlar
- rasm
- audio
- video
- chat tarixlari
- web search natijalari
- kelajakda API yoki database’dan keladigan structured data

Bu tizimning asosiy g‘oyasi shuki:

1. Ma’lumot keladi
2. Fayl yoki content xavfsiz saqlanadi
3. Content text yoki multimodal embedding ko‘rinishiga o‘tkaziladi
4. Vector bazaga indekslanadi
5. So‘rov kelganda eng mos bo‘laklar topiladi
6. Kerak bo‘lsa AI yordamida javob generatsiya qilinadi

---

## 2. Tizimning 5 ta asosiy service’i

Bu arxitekturada quyidagi 5 ta service ishlaydi:

1. **power-knowledge-ingest-fastapi-service**
2. **power-knowledge-processing-fastapi-service**
3. **power-vector-fastapi-service**
4. **power-knowledge-retrieval-fastapi-service**
5. **power-ai-fastapi-service**

Bular birgalikda bitta yagona OpenKnowledge tizimini hosil qiladi.

---

## 3. Har bir service’ning vazifasi

### 3.1. power-knowledge-ingest-fastapi-service

Bu service tizimning **kirish nuqtasi** hisoblanadi.

### Vazifalari:

- foydalanuvchidan fayl qabul qilish
- text qabul qilish
- chat history qabul qilish
- web search result yoki boshqa tashqi data qabul qilish
- upload qilingan faylni MinIO yoki object storage’ga saqlash
- dastlabki metadata yaratish
- processing uchun event yuborish
- ingest jarayonining holatini qayd etish

### Nimalarni o‘zi qilmaydi:

- OCR qilmaydi
- speech-to-text qilmaydi
- embedding yaratmaydi
- qidiruv qilmaydi
- javob generatsiya qilmaydi

Uning vazifasi: **qabul qilish, saqlash va pipeline’ga uzatish**.

### Misol:

Foydalanuvchi video yuboradi.

Ingest service:
- videoni qabul qiladi
- storage’ga joylaydi
- record yaratadi
- RabbitMQ orqali processing service’ga event yuboradi

---

### 3.2. power-knowledge-processing-fastapi-service

Bu service tizimning **processing engine** qismi.

### Vazifalari:

- kelgan content turini aniqlash
- text extraction qilish
- OCR ishlatish
- audio transcription qilish
- video’dan frame va audio ajratish
- image caption olish
- content’ni tozalash va normalize qilish
- chunking qilish
- metadata boyitish
- kerak bo‘lsa AI service’dan yordam olish
- tayyor bo‘lgan chunklarni vector service’ga yuborish

### Misollar:

#### PDF:
- text extract
- sahifa bo‘yicha bo‘lish
- chunk qilish

#### Audio:
- speech-to-text
- transcript tozalash
- chunk qilish

#### Rasm:
- OCR
- image caption
- detected text + caption ni birlashtirish

#### Video:
- audio extract
- speech-to-text
- frame sampling
- image caption
- transcript + visual description ni birlashtirish
- vaqt bo‘yicha chunk qilish

### Nega bu service alohida:

Chunki bu qism:
- CPU heavy
- GPU heavy bo‘lishi mumkin
- batch processing talab qiladi
- queue bilan ishlashi kerak

---

### 3.3. power-vector-fastapi-service

Bu service semantik qidiruvning **index va storage layer** qismi.

### Vazifalari:

- embedding qabul qilish
- vectorlarni saqlash
- payload metadata saqlash
- similarity search qilish
- filter bilan qidirish
- collection / namespace boshqarish
- top-k natijalarni qaytarish

### Nimalarni saqlaydi:

Vector service odatda quyidagilarni saqlaydi:

- vector
- text chunk
- file_url
- source type
- tenant_id
- user_id yoki project_id
- chunk index
- timestamp
- language
- extra metadata

### Muhim nuance:

Vector DB faylning o‘zini saqlamaydi. Fayl storage’da saqlanadi.
Vector DB esa:
- embedding
- text representation
- metadata
saqlaydi.

---

### 3.4. power-knowledge-retrieval-fastapi-service

Bu service foydalanuvchi so‘rovlari bilan ishlaydigan **search orchestration layer**.

### Vazifalari:

- foydalanuvchi query’sini qabul qilish
- query type’ni aniqlash
- text query embedding yaratish
- kerak bo‘lsa image query yoki multimodal query bilan ishlash
- vector service’dan eng mos natijalarni olish
- filter, rerank, deduplicate qilish
- chunklarni yig‘ish va formatlash
- kerak bo‘lsa AI service’ga context yuborish
- search natija yoki final answer qaytarish

### Bu service qaysi endpointlarni beradi:

- `POST /search`
- `POST /ask`
- `POST /similar`
- `POST /retrieve`

### Farqi:

#### `/search`
Faqat mos natijalarni qaytaradi.

#### `/ask`
Topilgan natijalarni AI’ga yuborib tayyor javob qaytaradi.

#### `/similar`
Masalan rasmga o‘xshash rasm yoki videoga o‘xshash video topish uchun ishlaydi.

---

### 3.5. power-ai-fastapi-service

Bu service tizimning **LLM va AI gateway** qismi.

### Vazifalari:

- LLM provider’lar bilan ishlash
- text generation
- summarization
- reasoning
- tool calling
- speech-to-text modelga ulanish
- image caption modelga ulanish
- embedding provider bilan ishlash
- kerak bo‘lsa translation yoki classification qilish

### OpenKnowledge ichidagi roli:

AI service bu tizimda o‘zi hamma ishni qilmaydi. U yordamchi va intellektual qatlam sifatida ishlaydi.

Misol:
- processing service rasm uchun caption olishda AI service’dan foydalanadi
- retrieval service topilgan context asosida answer yaratishda AI service’dan foydalanadi

### Muhim qaror:

AI logic alohida service’da qoladi. Shu bilan boshqa barcha platforma servislarida ham qayta foydalanish mumkin bo‘ladi.

---

## 4. Tizim qanday ishlaydi

Quyida umumiy jarayon bosqichma-bosqich berilgan.

---

## 5. Ingest pipeline

### Step 1. Foydalanuvchi content yuboradi

Masalan:
- PDF
- rasm
- video
- audio
- text
- chat history

So‘rov gateway orqali `knowledge-ingest-service` ga keladi.

### Step 2. Ingest service content’ni saqlaydi

- fayl bo‘lsa object storage’ga yozadi
- text bo‘lsa DB’da raw content record yaratadi
- metadata yozadi
- `document_id` yoki `knowledge_item_id` yaratadi

### Step 3. Event yuboradi

RabbitMQ orqali `knowledge.item.ingested` event yuboriladi.

### Step 4. Processing service event’ni qabul qiladi

U content type bo‘yicha kerakli pipeline’ni ishga tushiradi.

### Step 5. Content textga yoki semantik representation’ga aylantiriladi

Masalan:
- video → transcript + visual description
- rasm → OCR + caption
- audio → transcript
- pdf → extracted text

### Step 6. Chunklar yaratiladi

Content ma’lum o‘lchamdagi bo‘laklarga bo‘linadi.

### Step 7. Embedding yaratiladi

Har chunk embedding ko‘rinishiga o‘tkaziladi.

### Step 8. Vector service’ga saqlanadi

Vector + payload saqlanadi.

### Step 9. Holat yangilanadi

Record status: `pending → processing → indexed`

---

## 6. Search pipeline

### Step 1. User so‘rov yuboradi

Masalan:
- “shu mavzuga oid hujjatlarni top”
- “mana bu rasmga o‘xshashini top”
- “shu videoda nima gapirilgan?”
- “o‘tgan chatlardan shu masalaga yaqin gaplarni top”

So‘rov `knowledge-retrieval-service` ga keladi.

### Step 2. Query tahlil qilinadi

Tizim aniqlaydi:
- oddiy text searchmi
- semantic searchmi
- similarity searchmi
- AI answer kerakmi

### Step 3. Query embedding yaratiladi

Text query bo‘lsa embedding qilinadi.
Rasm bo‘lsa multimodal embedding ishlatiladi.

### Step 4. Vector service’dan natijalar olinadi

- similarity score bo‘yicha top-k
- tenant yoki source filter
- vaqt yoki type bo‘yicha filter

### Step 5. Natijalar rerank qilinadi

Kerak bo‘lsa:
- duplicate’lar olib tashlanadi
- relevance oshiriladi
- source grouping qilinadi

### Step 6A. Search natija qaytariladi

Agar `/search` bo‘lsa, tizim topilgan chunklar va metadata’ni qaytaradi.

### Step 6B. AI answer yaratiladi

Agar `/ask` bo‘lsa, retrieval service topilgan context’ni AI service’ga yuboradi va final javob oladi.

---

## 7. Multimodal qidiruv qanday ishlaydi

Bu tizimning eng kuchli tomoni — bir nechta format bilan ishlashidir.

### 7.1. Text → text search

Oddiy semantic qidiruv.

### 7.2. Text → image / video search

Masalan user yozadi:
- “qizil mashina oldida turgan odam rasmi”

Agar rasm va video’lar caption va visual description bilan indekslangan bo‘lsa, tizim ulardan moslarini topa oladi.

### 7.3. Image → image search

Agar rasm embedding alohida saqlansa, o‘xshash rasm topish mumkin.

### 7.4. Image → video search

Agar video frame embedding’lari yoki caption’lari saqlangan bo‘lsa, rasmga o‘xshash video epizodlar topish mumkin.

### 7.5. Audio / Video → semantic search

Transcript va description orqali semantik qidiruv ishlaydi.

---

## 8. Qaysi umumiy komponentlardan foydalaniladi

Yangi kodni ko‘paytirmaslik uchun mavjud Power Platform komponentlaridan foydalaniladi.

### 8.1. power-fastapi-core

Barcha yangi service’lar uchun asosiy baza:
- app factory
- config
- logging
- database helpers
- response schema
- exceptions
- RabbitMQ producer/consumer helpers
- Redis cache helpers

### 8.2. power-service-fastapi-template

Har bir yangi service shu template asosida yaratiladi.
Bu bilan:
- papka struktura bir xil bo‘ladi
- event handling bir xil bo‘ladi
- config bir xil bo‘ladi
- deployment va docker sozlamalar standart bo‘ladi

### 8.3. power-vector-fastapi-service

Yangi vector logic qayta yozilmaydi. Mavjud service ishlatiladi.

### 8.4. power-ai-fastapi-service

Caption, transcription, summarization, answer generation shu service orqali bajariladi.

---

## 9. Saqlanadigan asosiy entity’lar

Bu tizim uchun kamida quyidagi asosiy entity’lar kerak bo‘ladi.

### Ingest service database:

#### KnowledgeItem
- id
- tenant_id
- source_type
- mime_type
- title
- description
- storage_url
- raw_text
- status
- language
- created_by
- created_at

#### IngestJob
- id
- knowledge_item_id
- status
- error_message
- retry_count
- created_at
- updated_at

### Processing service database:

#### ProcessingJob
- id
- knowledge_item_id
- status
- started_at
- finished_at
- processor_version
- error_message

#### ExtractedArtifact
- id
- knowledge_item_id
- artifact_type
- content
- metadata

### Vector payload metadata:

- knowledge_item_id
- chunk_id
- chunk_index
- text
- file_url
- source_type
- language
- timestamp_start
- timestamp_end
- tenant_id
- user_id
- tags
- extra metadata

---

## 10. Holatlar va statuslar

Har bir content item quyidagi holatlardan o‘tishi mumkin:

- `uploaded`
- `queued`
- `processing`
- `processed`
- `indexed`
- `failed`
- `deleted`

Bu statuslar admin panel, monitoring va retry mexanizmi uchun juda muhim.

---

## 11. Event’lar qanday ishlaydi

Bu tizim event-driven bo‘lishi kerak.

### Asosiy event’lar:

#### `knowledge.item.ingested`
Ingest service yuboradi.

#### `knowledge.item.processing.started`
Processing service yuboradi.

#### `knowledge.item.processed`
Processing tugaganda yuboriladi.

#### `knowledge.item.indexed`
Vector service muvaffaqiyatli saqlanganda qayd qilinadi.

#### `knowledge.item.failed`
Xato bo‘lsa yuboriladi.

Bu eventlar keyinchalik:
- audit
- analytics
- retry workers
- billing
- notification
uchun ham ishlatiladi.

---

## 12. Multi-tenant va universal ishlash

Bu tizim universal bo‘lishi uchun har bir record tenant yoki namespace bilan bog‘lanadi.

Masalan:
- bitta userning shaxsiy knowledge base’i
- bitta kompaniyaning umumiy knowledge base’i
- bitta loyiha uchun alohida collection
- bitta bot yoki product uchun alohida namespace

Shu sababli barcha service’larda quyidagi tushuncha bo‘lishi kerak:

- `tenant_id`
- `project_id` yoki `workspace_id`
- `source_type`
- `visibility`

Bu kelajakda OpenKnowledge’ni alohida SaaS product qilishga yordam beradi.

---

## 13. Nega aynan 5 ta service yaxshi yechim

Bu bo‘linish amaliy jihatdan eng to‘g‘ri balansni beradi.

### Ingest alohida:
Upload va storage bilan ishlash uchun.

### Processing alohida:
CPU/GPU heavy pipeline uchun.

### Vector alohida:
Semantic index va qidiruv storage uchun.

### Retrieval alohida:
Qidiruv logikasi, filter, rerank va answer assembly uchun.

### AI alohida:
LLM va multimodal model provider qatlamini reuse qilish uchun.

Agar bularni bitta service’ga tiqib yuborilsa:
- maintain qilish qiyinlashadi
- scale qilish qiyinlashadi
- kod chalkashib ketadi

Agar juda mayda-mayda service’ga bo‘linsa:
- early stage’da overengineering bo‘ladi
- network complexity oshadi

Shuning uchun aynan shu 5 ta service universal va balansli arxitektura beradi.

---

## 14. Minimal ishga tushirish tartibi

Boshlash uchun quyidagi ketma-ketlik eng to‘g‘ri:

1. `power-vector-fastapi-service` ni tayyorlash yoki tekshirish
2. `power-ai-fastapi-service` ni kerakli capability’lar bilan kengaytirish
3. `power-knowledge-ingest-fastapi-service` yaratish
4. `power-knowledge-processing-fastapi-service` yaratish
5. `power-knowledge-retrieval-fastapi-service` yaratish
6. Gateway’da route ulash
7. RabbitMQ event flow’ni ulash
8. MinIO integratsiya qilish
9. test pipeline yozish

---

## 15. Yakuniy xulosa

OpenKnowledge uchun ushbu 5 ta service birgalikda ishlaydigan yagona, universal va kengaytiriladigan tizim hosil qiladi.

### Ularning umumiy vazifasi:

- **Ingest service** ma’lumotni qabul qiladi va pipeline’ga uzatadi
- **Processing service** ma’lumotni semantik ishlovdan o‘tkazadi
- **Vector service** embedding va metadata’ni indekslaydi
- **Retrieval service** kerakli natijalarni topadi va yig‘adi
- **AI service** intellektual qayta ishlash va javob generatsiyasini bajaradi

Bu arxitektura:
- universal
- reusable
- scaleable
- Power Platform Core bilan mos
- future-proof

Aynan shu yondashuv bilan boshqa servislar ichidagi mavjud kodlar qayta yozilmaydi, balki mavjud `core`, `vector` va `ai` qatlamlaridan to‘g‘ri foydalaniladi.

