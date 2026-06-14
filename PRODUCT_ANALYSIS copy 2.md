# AuraNova Backend — Product Analysis

## 1. Архитектура

**Тип:** Монолитный REST API

**Структура:**
```
Client (Flutter/Web)
        ↓
   Express.js API (:3000)
        ↓
   ┌────┼────────────┬──────────────┐
   ↓    ↓            ↓              ↓
Upload  Prisma     OpenAI        AuraNovaBrain
(Multer) ORM     GPT-4o Vision   Wiki (RAG)
          ↓
    PostgreSQL (Docker :6432)
```

**Паттерны:**
- **Singleton** — `PrismaClient` в едином экземпляре (`src/prisma.ts`)
- **RAG (Retrieval-Augmented Generation)** — Wiki-база знаний AuraNovaBrain подаётся в промпт GPT-4o
- **Chain of AI** — последовательный пайплайн: анализ кожи → анализ продуктов → синтез советов

## 2. Ключевые функции

### `POST /upload/photo`
- Принимает одно фото (Multer `single`)
- Генерирует UUID-имя файла для защиты от коллизий
- Переименовывает через `fs.rename`
- Логирует размер, MIME-тип, исходное и новое имя

### `POST /upload/products`
- Принимает массив фото продуктов (Multer `array`)
- Каждому — уникальное UUID-имя
- Логирует каждый файл отдельно

### `POST /user-daily-care`
**Центральный эндпоинт.** Принимает:
- `userId`, `photo`, `ageRange`, `skinConcerns`, `climate`, `healthFactors`, `supplements`, `products`, `goals`

**Пайплайн обработки:**

| Шаг | Функция | Модель | Что делает |
|---|---|---|---|
| 1 | `analyzeSkin()` | GPT-4o Vision | Анализ фото кожи: тон, текстура, акне, пигментация, поры, увлажнённость |
| 2 | `analyzeProducts()` | GPT-4o Vision | Анализ фото средств: бренд, состав, назначение, безопасность |
| 3 | `getSkincareAdvice()` | GPT-4o + Wiki RAG | Персональные рекомендации на основе результатов шагов 1–2 + AuraNovaBrain |

Каждый шаг обёрнут в `try/catch` — ошибка одного не ломает остальные.

### `GET /user-init`
- Создаёт пользователя с UUID-идентификатором
- Возвращает `userId`

### `GET /health`
- Проверяет подключение к PostgreSQL через `SELECT 1`
- Возвращает статус `ok` или `disconnected` с деталями ошибки

## 3. Применённые технологии

| Категория | Технология | Роль |
|---|---|---|
| **Runtime** | Node.js ≥ 18 | Серверная платформа |
| **Язык** | TypeScript 5.4 | Статическая типизация |
| **Фреймворк** | Express 4.19 | HTTP-роутинг, middleware |
| **ORM** | Prisma 5.14 | Типобезопасные запросы к БД, миграции |
| **БД** | PostgreSQL 16 (Docker) | Хранение данных |
| **Файлы** | Multer 2.1 | Multipart-загрузка фото |
| **AI Vision** | OpenAI GPT-4o | Анализ фото кожи и продуктов |
| **AI Text** | OpenAI GPT-4o | Синтез рекомендаций + RAG |
| **RAG** | AuraNovaBrain Wiki | Медицинская база знаний (markdown) |
| **DevOps** | Docker + Makefile | Управление инфраструктурой |
| **Dev** | ts-node-dev | Hot-reload при разработке |
| **Конфиг** | dotenv | Переменные окружения |
| **Логи** | Кастомные `console.log` | Структурированные логи с эмодзи-тегами |

## 4. Уникальные достижения

### 🧠 AI-дерматолог из коробки
GPT-4o Vision анализирует фото кожи с клинической точностью. Промпт построен на медицинской терминологии: *skin surface assessment*, *erythema*, *hyperpigmentation*, *pore congestion*. Системный промпт позиционирует модель как профессионального дерматолога, а не как general-purpose AI.

### 📚 RAG на собственном Wiki
AuraNovaBrain — локальная база знаний в markdown. Система маппит skin concern → wiki-страницу (напр. `acne` → `entities/acne.md`), подгружает 4 общие страницы (skincare recommendation systems, skin type classification, cosmetic ingredient safety, fuzzy logic skincare), обрезает YAML-frontmatter, и подаёт всё в контекст GPT-4o.

### 🔗 Трёхэтапный AI-пайплайн
Не просто «спроси у GPT» — а цепочка из трёх независимых AI-вызовов:
1. Vision анализирует фото лица
2. Vision анализирует фото продуктов
3. LLM синтезирует рекомендации, учитывая оба результата + wiki-контекст

Каждый этап fail-safe: если OpenAI недоступен или вернул ошибку — остальные продолжают работу.

### 🛡️ PrismaClient с адаптивным логированием
В зависимости от `NODE_ENV` включает либо полные query-логи (development), либо только ошибки (production) — экономит ресурсы.

### 🔐 UUID-именование файлов
Все загруженные фото получают `crypto.randomUUID()` + оригинальное расширение. Исключены коллизии имён и угадывание URL.

## 5. Интересные решения

### 1. Продуманная обработка ошибок
```typescript
// Каждый AI-шаг изолирован — один упал, остальные живы
try { analysisResult = await analyzeSkin(photo, {...}) }
catch (error: any) { console.error("Skin analysis failed:", error.message) }
```
Пользователь всегда получает ответ, даже если OpenAI лёг.

### 2. Graceful shutdown
```typescript
process.on("SIGINT", async () => {
  await prisma.$disconnect();
  process.exit(0);
});
```
Корректное закрытие соединения с БД при остановке сервера.

### 3. Docker-инкапсуляция
PostgreSQL на нестандартном порту `6432` (вместо 5432) — избегает конфликтов с локальной БД разработчика. Makefile-команды для `create-deps`, `destroy-deps`, `deploy-migrations`.

### 4. Умная очистка Wiki-frontmatter
```typescript
function stripFrontmatter(raw: string): string {
  const match = raw.match(/^---[\s\S]*?---\n*/);
  if (!match || typeof match.index !== "number") return raw;
  return raw.slice(match.index + match[0].length);
}
```
Извлекает только содержимое markdown-страниц, отбрасывая YAML-метаданные — GPT получает чистый текст.

### 5. Декларативный маппинг skin concern → wiki page
```typescript
const CONCERN_TO_ENTITY: Record<string, string> = {
  acne: "entities/acne.md",
  rosacea: "entities/rosacea.md",
  // ... 15+ skin conditions
};
```
Добавление новой болезни = одна строка в словаре.

### 6. Система логирования с эмодзи-тегами
```
[UPLOAD] 📸 Received: photo.jpg (245.3 KB)
[SKIN ANALYSIS] 🔍 Analyzing photo: abc.jpg
[SKIN ANALYSIS] 🤖 Sending to OpenAI Vision...
[SKIN ANALYSIS] ✅ Done
```
Визуально различимые этапы обработки — удобно при отладке в production.

### 7. Массивные поля в Prisma
```prisma
skinConcerns    String[] @default([])
healthFactors   String[] @default([])
supplements     String[] @default([])
products        String[] @default([])
goals           String[] @default([])
```
Нативные PostgreSQL-массивы через Prisma — без отдельных таблиц many-to-many для простых списков.

### 8. Единый `UserDailyCare` как snapshot
Каждый запрос создаёт **полный снимок** состояния: профиль, фото, результаты всех трёх AI-анализов. Это позволяет отслеживать прогресс пользователя во времени — каждая запись = одна консультация.

---

## Итог

**AuraNova Backend** — это AI-powered платформа для персонализированного ухода за кожей. Сочетает:
- Компьютерное зрение (GPT-4o Vision) для анализа фото
- RAG на собственной медицинской Wiki (AuraNovaBrain)
- Трёхэтапный AI-пайплайн с fail-safe архитектурой
- Чистый TypeScript + Prisma + PostgreSQL стек
- Docker-инкапсуляцию для воспроизводимой среды

Ключевая инновация — **retrieval-augmented skincare advisor**, который не просто выдаёт советы от LLM, а подкрепляет их evidence-based knowledge base.
