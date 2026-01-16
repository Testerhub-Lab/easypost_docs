# API Overview for QA

This document provides an overview of the key API groups involved in the main business process of the bot-admin-backend system. It focuses on endpoints and DTOs used in onboarding, posting, scheduling, and status management.

## 1. Onboarding/Auth

**Purpose:** User registration, authentication, and onboarding step management.

**Main Endpoints:**
- `POST /auth/register` - Register new user
- `POST /auth/login` - User login
- `POST /auth/refresh` - Refresh access token
- `GET /auth/me` - Get current user info
- `PATCH /auth/me` - Update onboarding step
- `POST /auth/logout` - Logout user

**Key DTO Fields:**
- RegisterDto: `email` (string, email), `password` (string, min 8)
- LoginDto: `email` (string, email), `password` (string, min 1)
- UpdateOnboardingStepDto: `onboardingStep` (enum: welcome, bot, channel, post, done, completed, skipped)

**Possible Errors/Validations:**
- 409: User with email already exists (register)
- 401: Invalid credentials (login/refresh)
- 400: Invalid onboardingStep value
- 404: User not found

## 2. Posts

**Purpose:** CRUD operations for scheduled posts, including drafts and published posts.

**Main Endpoints:**
- `POST /posts/schedule` - Create scheduled post or draft
- `POST /posts/draft` - Create draft post explicitly
- `GET /posts` - Get paginated posts with filters
- `GET /posts/:id` - Get post by ID
- `PATCH /posts/:id` - Update post
- `DELETE /posts/:id` - Delete post
- `PATCH /posts/:id/cancel` - Cancel post
- `PATCH /posts/:id/move-to-draft` - Move post back to draft
- `PATCH /posts/:id/reschedule` - Reschedule post
- `POST /posts/:id/retry` - Retry failed post
- `POST /posts/:id/duplicate` - Duplicate post
- `PATCH /posts/:id/publish` - Publish draft post

**Key DTO Fields:**
- CreatePostDto: `botId` (uuid), `channelId` (uuid, optional), `text` (string, optional), `attachments` (array, max 10), `linkPreviewOptions` (object), `scheduledAt` (datetime, optional), `isDraft` (boolean, optional)
- Attachment: `fileId` (uuid), `filename` (string, optional), `url` (string, url, optional), `mimeType` (string), `size` (number, positive), `kind` (enum: photo, video), `storageProvider` (enum: local, s3, optional), `storageKey` (string, optional)

**Possible Errors/Validations:**
- 400: Invalid post status for update (only DRAFT/SCHEDULED), missing required fields, scheduledAt in past
- 404: Post not found
- 409: Validation conflicts

## 3. Uploads/Attachments

**Purpose:** File upload and attachment management for posts.

**Note:** File uploads are handled as part of post creation. Files are stored locally or in S3.

**Main Endpoints:**
- Attachments are included in `POST /posts/schedule` and `POST /posts/draft`

**Key DTO Fields:**
- Same as Attachment in CreatePostDto above

**Possible Errors/Validations:**
- 400: Invalid attachment kind, missing required fields (filename/url/storageKey), count > 10
- File size limits: photos up to 10MB, videos up to 50MB

## 4. Scheduling

**Purpose:** Calendar event management for scheduling posts.

**Main Endpoints:**
- `POST /schedule` - Create event
- `GET /schedule` - Get events by date range
- `GET /schedule/:id` - Get event by ID
- `PATCH /schedule/:id` - Update event
- `DELETE /schedule/:id` - Delete event

**Key DTO Fields:**
- CreateEventDto: `lessonId` (uuid, optional), `title` (string), `date` (datetime)
- UpdateEventDto: `lessonId` (uuid, optional), `title` (string), `date` (datetime)

**Possible Errors/Validations:**
- 400: Invalid date format
- 404: Event not found

## 5. Preview/Test-Send

**Purpose:** Send test messages to bot's private chat for preview.

**Main Endpoints:**
- `POST /posts/test-send` - Send test message

**Key DTO Fields:**
- TestSendDto: `botId` (uuid), `text` (string, optional), `attachments` (array, max 10), `linkPreviewOptions` (object)

**Possible Errors/Validations:**
- 400: Bot not found, lastPrivateChatId missing
- Same attachment validations as posts

## 6. Statuses

**Purpose:** Post status management and updates.

**Main Endpoints:**
- `PATCH /posts/status` - Update post status (worker endpoint)
- `GET /posts/schedule` - Get schedule with status filters
- `GET /posts/upcoming` - Get upcoming posts

**Key DTO Fields:**
- UpdatePostStatusDto: `postId` (uuid), `status` (enum: DRAFT, SCHEDULED, PENDING, SENT, FAILED)

**Possible Errors/Validations:**
- 404: Post not found
- Post statuses: DRAFT (черновик), SCHEDULED (запланирован), PENDING (в обработке), SENT (отправлен), FAILED (ошибка)

## Minimal Test Data

To test the happy path:
1. Register user: `POST /auth/register` with email/password
2. Login: `POST /auth/login` to get tokens
3. Connect bot: `POST /bots` with bot token
4. Connect channel: `POST /channels` with botId and username
5. Create post: `POST /posts/schedule` with botId, channelId, text, scheduledAt
6. Test send: `POST /posts/test-send` with botId and text