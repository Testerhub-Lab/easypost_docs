# Обзор API для QA

Этот документ предоставляет обзор ключевых групп API, участвующих в основном бизнес-процессе системы bot-admin-backend. Он фокусируется на конечных точках и DTO, используемых в онбординге, публикации, планировании и управлении статусами.

## 1. Онбординг/Аутентификация

**Назначение:** Регистрация пользователей, аутентификация и управление этапами онбординга.

**Основные конечные точки:**
- `POST /auth/register` - Регистрация нового пользователя (public)
- `POST /auth/login` - Вход пользователя (public)
- `POST /auth/refresh` - Обновление токена доступа (public)
- `GET /auth/me` - Получение информации о текущем пользователе (auth)
- `PATCH /auth/me` - Обновление этапа онбординга (auth)
- `POST /auth/logout` - Выход пользователя (auth)

**Ключевые поля DTO:**
- RegisterDto: `email` (строка, email), `password` (строка, мин. 8 символов)
- LoginDto: `email` (строка, email), `password` (строка, мин. 1 символ)
- UpdateOnboardingStepDto: `onboardingStep` (enum: welcome, bot, channel, post, done, completed, skipped)

**Возможные ошибки/валидации:**
- 409: Пользователь с таким email уже существует (регистрация)
- 401: Неверные учетные данные (вход/обновление)
- 400: Неверное значение onboardingStep
- 404: Пользователь не найден

## 2. Посты

**Назначение:** CRUD-операции для запланированных постов, включая черновики и опубликованные посты.

**Основные конечные точки:**
- `POST /posts/schedule` - Создание запланированного поста или черновика (auth)
- `POST /posts/draft` - Создание черновика поста (auth)
- `GET /posts` - Получение постов с пагинацией и фильтрами (auth)
- `GET /posts/:id` - Получение поста по ID (auth)
- `GET /posts/:id/details` - Получение поста с деталями и ошибками (auth)
- `PATCH /posts/:id` - Обновление поста (auth)
- `DELETE /posts/:id` - Удаление поста (auth)
- `PATCH /posts/:id/cancel` - Отмена поста (auth)
- `PATCH /posts/:id/move-to-draft` - Перемещение поста обратно в черновик (auth)
- `PATCH /posts/:id/reschedule` - Перенос поста (auth)
- `POST /posts/:id/retry` - Повторная попытка отправки неудавшегося поста (auth)
- `POST /posts/:id/duplicate` - Дублирование поста (auth)
- `PATCH /posts/:id/publish` - Публикация черновика (auth)

**Ключевые поля DTO:**
- CreatePostDto: `botId` (uuid), `channelId` (uuid, опционально), `text` (строка, опционально), `attachments` (массив, макс. 10), `linkPreviewOptions` (объект), `scheduledAt` (дата-время, опционально), `isDraft` (булево, опционально)
- Attachment: `fileId` (uuid), `filename` (строка, опционально), `url` (строка, url, опционально), `mimeType` (строка), `size` (число, положительное), `kind` (enum: photo, video), `storageProvider` (enum: local, s3, опционально), `storageKey` (строка, опционально)

**Возможные ошибки/валидации:**
- 400: Неверный статус поста для обновления (только DRAFT/SCHEDULED), отсутствие обязательных полей, scheduledAt в прошлом
- 404: Пост не найден
- 409: Конфликты валидации

## 3. Загрузки/Вложения

**Назначение:** Управление загрузкой файлов и вложениями для постов.

**Примечание:** Загрузка файлов обрабатывается как часть создания поста. Файлы хранятся локально или в S3.

**Основные конечные точки:**
- Вложения включаются в `POST /posts/schedule` и `POST /posts/draft`

**Ключевые поля DTO:**
- Те же, что и Attachment в CreatePostDto выше

**Возможные ошибки/валидации:**
- 400: Неверный тип вложения, отсутствие обязательных полей (filename/url/storageKey), количество > 10
- Ограничения по размеру файлов: фотографии до 10МБ, видео до 50МБ

## 4. Планирование

**Назначение:** Управление событиями календаря для планирования постов.

**Основные конечные точки:**
- `POST /schedule` - Создание события (auth)
- `GET /schedule` - Получение событий по диапазону дат (auth)
- `GET /schedule/:id` - Получение события по ID (auth)
- `PATCH /schedule/:id` - Обновление события (auth)
- `DELETE /schedule/:id` - Удаление события (auth)

**Ключевые поля DTO:**
- CreateEventDto: `lessonId` (uuid, опционально), `title` (строка), `date` (дата-время)
- UpdateEventDto: `lessonId` (uuid, опционально), `title` (строка), `date` (дата-время)

**Возможные ошибки/валидации:**
- 400: Неверный формат даты
- 404: Событие не найдено

## 5. Предпросмотр/Тестовая отправка

**Назначение:** Отправка тестовых сообщений в приватный чат бота для предпросмотра.

**Основные конечные точки:**
- `POST /posts/test-send` - Отправка тестового сообщения (auth)

**Ключевые поля DTO:**
- TestSendDto: `botId` (uuid), `text` (строка, опционально), `attachments` (массив, макс. 10), `linkPreviewOptions` (объект)

**Возможные ошибки/валидации:**
- 400: Бот не найден, отсутствует lastPrivateChatId
- Те же валидации вложений, что и для постов

## 6. Статусы

**Назначение:** Управление статусами постов и их обновления.

**Основные конечные точки:**
- `PATCH /posts/status` - Обновление статуса поста (конечная точка воркера) (public)
- `GET /posts/schedule` - Получение расписания с фильтрами статусов (auth)
- `GET /posts/upcoming` - Получение предстоящих постов (auth)

**Ключевые поля DTO:**
- UpdatePostStatusDto: `postId` (uuid), `status` (enum: DRAFT, SCHEDULED, PENDING, SENT, FAILED)

**Возможные ошибки/валидации:**
- 404: Пост не найден
- Статусы постов: DRAFT (черновик), SCHEDULED (запланирован), PENDING (в обработке), SENT (отправлен), FAILED (ошибка)

## Минимальные тестовые данные

Для тестирования основного сценария:
1. Зарегистрировать пользователя: `POST /auth/register` с email/паролем
2. Войти: `POST /auth/login` для получения токенов
3. Подключить бота: `POST /bots` с токеном бота
4. Подключить канал: `POST /channels` с botId и username
5. Создать пост: `POST /posts/schedule` с botId, channelId, text, scheduledAt
6. Тестовая отправка: `POST /posts/test-send` с botId и text

## Endpoint Coverage

| Endpoint | Verified |
|----------|----------|
| `POST /auth/register` | Yes |
| `POST /auth/login` | Yes |
| `POST /auth/refresh` | Yes |
| `GET /auth/me` | Yes |
| `PATCH /auth/me` | Yes |
| `POST /auth/logout` | Yes |
| `POST /posts/schedule` | Yes |
| `POST /posts/draft` | Yes |
| `GET /posts` | Yes |
| `GET /posts/:id` | Yes |
| `GET /posts/:id/details` | Yes |
| `PATCH /posts/:id` | Yes |
| `DELETE /posts/:id` | Yes |
| `PATCH /posts/:id/cancel` | Yes |
| `PATCH /posts/:id/move-to-draft` | Yes |
| `PATCH /posts/:id/reschedule` | Yes |
| `POST /posts/:id/retry` | Yes |
| `POST /posts/:id/duplicate` | Yes |
| `PATCH /posts/:id/publish` | Yes |
| `POST /posts/test-send` | Yes |
| `PATCH /posts/status` | Yes |
| `GET /posts/schedule` | Yes |
| `GET /posts/upcoming` | Yes |
| `POST /schedule` | Yes |
| `GET /schedule` | Yes |
| `GET /schedule/:id` | Yes |
| `PATCH /schedule/:id` | Yes |
| `DELETE /schedule/:id` | Yes |
| `POST /bots` | Yes |
| `GET /bots` | Yes |
| `GET /bots/:id` | Yes |
| `PATCH /bots/:id` | Yes |
| `PATCH /bots/:id/toggle-active` | Yes |
| `DELETE /bots/:id` | Yes |
| `GET /bots/:id/webhook-info` | Yes |
| `POST /bots/:id/webhook-reregister` | Yes |
| `POST /channels` | Yes |
| `GET /channels` | Yes |
| `GET /channels/discover` | Yes |
| `GET /channels/:id` | Yes |
| `PATCH /channels/:id` | Yes |
| `PATCH /channels/:id/toggle-active` | Yes |
| `DELETE /channels/:id` | Yes |
| `POST /channels/:id/connect` | Yes |
| `POST /channels/:id/update-photo` | Yes |