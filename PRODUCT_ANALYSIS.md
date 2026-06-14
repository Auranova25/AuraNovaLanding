# AuraNova — Product Analysis

> **"Your Personal Glow AI Coach"**  
> Приложение для персонализированного ухода за кожей с AI-анализом.

---

## 1. Архитектура

### Стек технологий

| Уровень | Технология | Версия |
|---------|-----------|--------|
| Фреймворк | React Native (Expo Managed) | SDK 54 |
| Язык | TypeScript | 5.8 |
| React | — | 19.1.0 |
| HTTP-клиент | Axios | 1.16.1 |
| Навигация | Кастомная (Step-based wizard) | — |
| Безопасность | expo-secure-store (AES-шифрование) | 15.0.8 |
| Камера | expo-image-picker | 17.0.11 |
| Иконки | @expo/vector-icons (Feather, Ionicons, MaterialCommunityIcons) | 15.0.3 |
| UI-эффекты | expo-linear-gradient, Animated API | 15.0.8 |
| Адаптивность | react-native-safe-area-context | 5.6.0 |
| Билд | EAS Build | — |

### Структура проекта

```
src/
├── components/     # 7 переиспользуемых UI-компонентов
│   ├── ErrorBoundary.tsx      # Глобальный отлов ошибок с восстановлением
│   ├── GradientIcon.tsx       # Иконка в градиентном круге
│   ├── LoadingScreen.tsx      # Экран загрузки с анимацией
│   ├── NavigationButtons.tsx  # Back/Continue с логикой отправки данных
│   ├── ProgressBar.tsx        # Индикатор прогресса онбординга
│   ├── SelectionButton.tsx    # Кнопка выбора с 3 цветовыми темами
│   └── StepIndicators.tsx     # Иконки шагов (пройден/текущий/будущий)
│
├── screens/        # 9 экранов онбординга
│   ├── WelcomeScreen.tsx       # Шаг 0: лендинг с анимациями
│   ├── PhotoScreen.tsx         # Шаг 1: селфи + загрузка на сервер
│   ├── ProfileScreen.tsx       # Шаг 2: возраст + проблемы кожи
│   ├── EnvironmentScreen.tsx   # Шаг 3: климат
│   ├── HealthScreen.tsx        # Шаг 4: здоровье + добавки
│   ├── ProductsScreen.tsx      # Шаг 5: фото косметики (множественная)
│   ├── GoalsScreen.tsx         # Шаг 6: цели по уходу
│   ├── CommunityScreen.tsx     # Шаг 7: информация о сообществе
│   └── AnalysisScreen.tsx      # Шаг 8: AI-анализ (анимированный)
│
├── hooks/          # 5 кастомных хуков
│   ├── useOnboarding.ts       # Управление шагами визарда
│   ├── useCamera.ts            # Съёмка + загрузка одного фото
│   ├── useProductsCamera.ts    # Съёмка нескольких фото продуктов
│   ├── useUserDailyCare.ts     # Состояние данных пользователя
│   └── useUserId.ts            # Инициализация/сохранение userId
│
├── services/       # 6 сервисных модулей
│   ├── apiClient.ts            # Настроенный экземпляр axios
│   ├── photoService.ts         # POST /upload/photo (multipart)
│   ├── productsService.ts      # POST /upload/products (multipart)
│   ├── userService.ts          # GET /user-init
│   ├── userDailyCareService.ts # POST /user-daily-care (JSON)
│   └── secureStorage.ts        # Абстракция над expo-secure-store
│
├── constants/      # 2 конфигурационных файла
│   ├── api.ts                 # BASE_URL
│   └── theme.ts               # colors, gradients, spacing, radius, fontSize
│
└── types/          # 1 файл типов
    └── index.ts               # UserDailyCare, UserInputs, Step
```

---

## 2. Ключевые функции

### 2.1. Онбординг-визард (9 шагов)

Пошаговый сбор данных о пользователе с немедленной синхронизацией в `UserDailyCare` при каждом изменении. Данные агрегируются на протяжении всего флоу и отправляются одним POST-запросом на шаге Community (шаг 7).

**Поток данных:**
```
Screen → handleInputChange/ArrayToggle → useOnboarding (steps state)
     → updateUserDailyCare (дублирует в агрегированное состояние)
     → NavigationButtons (шаг 7) → submitUserDailyCare() → POST /user-daily-care
```

### 2.2. AI-анализ кожи

- **Шаг 1 (PhotoScreen)**: камера открывается фронтальной камерой, снимок обрезается до квадрата 1:1, сразу загружается на сервер через `POST /upload/photo` (multipart/form-data). При тапе на превью — пересъёмка.
- **Шаг 5 (ProductsScreen)**: множественная съёмка упаковок косметики. Фото накапливаются в массиве `localUris`, отправляются пачкой через `POST /upload/products` при нажатии кнопки "Upload all".
- **Шаг 8 (AnalysisScreen)**: анимированный экран с этапами анализа и мок-статистикой.

### 2.3. Идентификация пользователя

`useUserId` реализует двухуровневую стратегию:
1. **SecureStore** — проверяет наличие сохранённого `user_id` (AES-шифрование через Android Keystore / iOS Keychain)
2. **Backend** — если userId отсутствует, запрашивает новый через `GET /user-init`
3. **Fallback** — при любой ошибке возвращает пустую строку, приложение продолжает работу

### 2.4. Отказоустойчивость

`ErrorBoundary` (классовый компонент) оборачивает всё приложение, перехватывает ошибки рендеринга, показывает стек-трейс и кнопку «Попробовать снова» для восстановления без перезагрузки.

---

## 3. Применённые технологии

| Технология | Где используется |
|-----------|-----------------|
| **React Native Animated API** | WelcomeScreen (пульсация + подпрыгивание), AnalysisScreen (пульсация), LoadingScreen (fade-in) |
| **expo-linear-gradient** | Фоны, иконки в кругах, карточки — 8 градиентов в теме |
| **expo-secure-store** | Хранение userId с аппаратным шифрованием |
| **expo-image-picker** | Доступ к фронтальной/основной камере |
| **react-native-safe-area-context** | Корректные отступы на устройствах с notch/punch-hole |
| **TypeScript generics** | Типизированные `useState`, `useCallback` с keyof-ограничениями |
| **FormData (React Native)** | Multipart-загрузка файлов через axios |
| **EAS Build** | Корпоративная сборка через `eas.json` |

---

## 4. Уникальные достижения

### 4.1. Двойная синхронизация состояния

Данные пользователя живут одновременно в двух состояниях:
- `useOnboarding` — для экранов (UI-отображение выбранных опций)
- `useUserDailyCare` — агрегированное состояние для отправки на сервер

Это позволяет отправлять данные в реальном времени (фото загружается сразу после съёмки), а текстовые данные — одним пакетом в конце визарда.

### 4.2. Два режима работы с камерой

- **PhotoScreen**: одиночное фото, немедленная загрузка, повторная съёмка по тапу
- **ProductsScreen**: множественная съёмка с превью-сеткой, удаление по long-press, пакетная отправка

### 4.3. Самонастраивающийся apiClient

После обнаружения проблем с multipart-загрузкой (конфликт Content-Type между дефолтным `application/json` и FormData в React Native) — дефолтный заголовок убран. Axios теперь сам определяет тип контента: JSON для объектов, multipart для FormData.

### 4.4. Безопасное хранение с fallback

`useUserId` не блокирует приложение при ошибке сети или SecureStore — возвращает пустую строку и позволяет пользователю продолжить onboarding.

### 4.5. Эмодзи-дизайн и Gen Z tone-of-voice

Весь UI использует эмодзи, поддерживает дружелюбный «bestie»-стиль общения: "✨ Let's see that beautiful skin! ✨", "No judgment, just science! 💜"

---

## 5. Интересные решения

### 5.1. Гибкий компонент GradientIcon

```typescript
interface GradientIconProps {
  iconFamily: 'Feather' | 'MaterialCommunityIcons' | 'AntDesign';
  iconName: string;
  colors: readonly [string, string, ...string[]];
  size?: number;
  iconSize?: number;
}
```

Позволяет комбинировать любые иконки из трёх библиотек с любым градиентом и размером. Используется на каждом экране для единообразия.

### 5.2. SelectionButton с вариантностью

Три цветовые темы (`purple`, `pink`, `green`) через `variantMap` — чистый pattern matching без switch-case:

```typescript
const variantMap: Record<Variant, { border: string; bg: string; text: string }> =
  { purple: {...}, pink: {...}, green: {...} };
```

### 5.3. Step-индикаторы с иконками вместо точек

Вместо стандартных точек — круги с иконками из тех же библиотек, что и на экранах. Пройденные шаги — зелёные с галочкой, текущий — фиолетовый, будущие — серые.

### 5.4. Отправка на предпоследнем шаге

Данные отправляются на сервер не на последнем экране (Analysis), а на шаге 7 (Community). Это даёт серверу время обработать запрос, пока пользователь смотрит анимированный экран анализа.

### 5.5. Типизированное переключение массивов

```typescript
handleArrayToggle(
  field: keyof Pick<UserInputs, 'skinConcerns' | 'goals' | 'healthConditions' | 'supplements'>,
  item: string
)
```

TypeScript не даст передать строковое поле (как `age` или `climate`) — только массивы. Опечатка в имени поля будет отловлена на этапе компиляции.

### 5.6. Пересъёмка фото в PhotoScreen

При тапе на загруженное превью открывается камера для повторной съёмки. Во время загрузки тап заблокирован (`disabled={isUploading}`). В правом нижнем углу — подсказка «Переснять» с иконкой.

### 5.7. Анимации с useNativeDriver

Все анимации (пульсация, bounce, fade-in) используют `useNativeDriver: true` — выполняются на нативном UI-потоке, не блокируя JS-поток.

---

## Итого

**AuraNova** — это хорошо структурированное Expo-приложение на TypeScript с чётким разделением на слои (components, screens, hooks, services, constants, types). Пошаговый визард с AI-анализом кожи, мультикамерой и безопасным хранением. Код демонстрирует зрелые паттерны: кастомные хуки, двойная синхронизация состояния, типобезопасные утилиты, graceful degradation при ошибках.
