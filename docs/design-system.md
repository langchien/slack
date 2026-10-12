# Phân Tích Thiết Kế Hệ Thống — Slack Clone

> Mục tiêu: Xây dựng ứng dụng nhắn tin nhóm real-time đầy đủ chức năng như Slack, quy mô tối đa 100 người dùng đồng thời. Không cần xử lý horizontal scaling hay distributed systems.

---

## 1. Tổng Quan Kiến Trúc

```
┌─────────────────────────────────────────────────────────┐
│                      CLIENT (React)                      │
│  TanStack Router │ TanStack Query │ Zustand │ Socket.io  │
└──────────────────────────┬──────────────────────────────┘
                           │ HTTP REST + WebSocket
┌──────────────────────────▼──────────────────────────────┐
│                    SERVER (NestJS)                        │
│   REST Controllers │ WebSocket Gateway │ Guards/Pipes    │
└──────────┬──────────────────────────────┬───────────────┘
           │ Prisma ORM                   │ JWT / bcryptjs
┌──────────▼──────────┐        ┌──────────▼───────────────┐
│   PostgreSQL DB      │        │   uploads/ (local files)  │
└─────────────────────┘        └──────────────────────────┘
```

### Nguyên tắc thiết kế
- **Monorepo** với 2 thư mục: `client/` (Vite + React) và `server/` (NestJS)
- REST API cho các tác vụ CRUD, WebSocket cho real-time events
- JWT Access Token (15 phút) + Refresh Token (7 ngày) trong httpOnly cookie
- Tất cả entity đều có `createdAt`, `updatedAt`, `deletedAt` (soft delete)

---

## 2. Domain Model & Database Schema

### 2.1 Core Entities

```
User
├── id (uuid)
├── email (unique)
├── username (unique)
├── displayName
├── passwordHash
├── avatarUrl
├── statusMessage   -- "On vacation" etc.
├── presence        -- ONLINE | AWAY | DND | OFFLINE
├── role            -- ADMIN | MEMBER (system-wide)
├── createdAt / updatedAt / deletedAt

Workspace
├── id (uuid)
├── name
├── slug (unique)   -- URL identifier
├── iconUrl
├── description
├── ownerId (-> User)
├── createdAt / updatedAt

WorkspaceMember
├── id
├── workspaceId (-> Workspace)
├── userId (-> User)
├── role            -- OWNER | ADMIN | MEMBER | GUEST
├── displayName     -- override per workspace
├── joinedAt

Channel
├── id (uuid)
├── workspaceId (-> Workspace)
├── name
├── description
├── topic
├── type            -- PUBLIC | PRIVATE | DM | GROUP_DM
├── createdById (-> User)
├── isArchived
├── createdAt / updatedAt

ChannelMember
├── id
├── channelId (-> Channel)
├── userId (-> User)
├── role            -- OWNER | ADMIN | MEMBER
├── lastReadAt      -- de tinh unread count
├── notificationPreference -- ALL | MENTIONS | NOTHING
├── joinedAt

Message
├── id (uuid)
├── channelId (-> Channel)
├── userId (-> User)
├── content         -- raw text (markdown-like)
├── type            -- TEXT | SYSTEM | BOT
├── parentId (-> Message, nullable)  -- thread reply
├── editedAt
├── createdAt / updatedAt / deletedAt

MessageAttachment
├── id
├── messageId (-> Message)
├── fileUrl
├── fileName
├── fileSize
├── mimeType

Reaction
├── id
├── messageId (-> Message)
├── userId (-> User)
├── emoji           -- unicode emoji string
├── createdAt
-- unique(messageId, userId, emoji)

Notification
├── id
├── userId (-> User)
├── type            -- MENTION | REPLY | CHANNEL_INVITE | DM
├── referenceId     -- messageId hoac channelId
├── isRead
├── createdAt

SavedItem (Bookmark)
├── id
├── userId (-> User)
├── messageId (-> Message)
├── createdAt

UserWorkspaceStatus (per-workspace settings)
├── userId
├── workspaceId
├── sidebarOrder    -- JSON array of channel IDs
├── theme           -- override theme per workspace

SearchIndex (virtual - dung PostgreSQL full-text search)
-- Khong can bang rieng, dung tsvector tren Message.content
```

### 2.2 Relationships Map
```
Workspace --< WorkspaceMember >-- User
Workspace --< Channel
Channel   --< ChannelMember >-- User
Channel   --< Message
Message   --< Message (self-ref: thread)
Message   --< MessageAttachment
Message   --< Reaction >-- User
User      --< Notification
User      --< SavedItem >-- Message
```

---

## 3. API Design

### 3.1 REST Endpoints

**Auth** (`/api/auth`)
```
POST   /register          -- dang ky tai khoan
POST   /login             -- tra ve accessToken + set refreshToken cookie
POST   /refresh           -- lay accessToken moi tu refreshToken cookie
POST   /logout            -- xoa refreshToken
GET    /me                -- thong tin user hien tai
PATCH  /me                -- cap nhat profile (displayName, statusMessage)
POST   /me/avatar         -- upload avatar (multipart/form-data)
```

**Workspace** (`/api/workspaces`)
```
GET    /                  -- danh sach workspaces cua user
POST   /                  -- tao workspace moi
GET    /:id               -- chi tiet workspace
PATCH  /:id               -- cap nhat workspace (admin)
DELETE /:id               -- xoa workspace (owner)

GET    /:id/members       -- danh sach members
POST   /:id/members/invite -- moi member qua email
PATCH  /:id/members/:uid  -- doi role
DELETE /:id/members/:uid  -- kick member

GET    /:id/channels      -- danh sach channels user co the thay
```

**Channel** (`/api/workspaces/:wid/channels`)
```
GET    /                  -- list channels (public + joined private)
POST   /                  -- tao channel
GET    /:id               -- chi tiet channel
PATCH  /:id               -- cap nhat channel (name, topic, description)
DELETE /:id               -- archive/xoa channel

POST   /:id/members       -- join channel / invite member
DELETE /:id/members/:uid  -- leave / kick khoi channel

POST   /dm                -- tao hoac tim DM channel voi 1 user
POST   /group-dm          -- tao group DM
```

**Message** (`/api/channels/:cid/messages`)
```
GET    /                  -- paginate messages (cursor-based, limit 50)
POST   /                  -- gui message (text + optional attachments)
PATCH  /:id               -- edit message (chi author)
DELETE /:id               -- soft delete message (author hoac admin)

GET    /:id/thread        -- lay thread replies cua message
POST   /:id/reactions     -- toggle reaction
GET    /:id/reactions     -- list reactions voi users

POST   /:id/attachments   -- upload file dinh kem
```

**Search** (`/api/workspaces/:wid/search`)
```
GET    /messages?q=&channel=&from=&before=&after=
GET    /files?q=
GET    /channels?q=
GET    /people?q=
```

**Notification** (`/api/notifications`)
```
GET    /                  -- danh sach notifications (unread first)
PATCH  /read-all          -- danh dau tat ca da doc
PATCH  /:id/read          -- danh dau 1 notification da doc
```

**Saved Items** (`/api/saved-items`)
```
GET    /                  -- danh sach saved items cua user
POST   /                  -- bookmark message
DELETE /:id               -- unbookmark
```

### 3.2 WebSocket Events (Socket.io)

**Rooms**: Moi user join room `user:{userId}`, va join room `channel:{channelId}` cho tung channel ho la thanh vien.

**Server -> Client events**
```
message:new             -- { message: MessageDto }
message:updated         -- { messageId, content, editedAt }
message:deleted         -- { messageId, channelId }
reaction:updated        -- { messageId, reactions: ReactionGroupDto[] }
channel:new             -- { channel: ChannelDto } (khi duoc invite)
channel:updated         -- { channel: ChannelDto }
channel:deleted         -- { channelId }
member:joined           -- { channelId, user: UserDto }
member:left             -- { channelId, userId }
member:updated          -- { channelId, userId, role }
workspace:member:joined -- { workspaceId, user: UserDto }
workspace:member:left   -- { workspaceId, userId }
user:presence:updated   -- { userId, presence }
typing:start            -- { channelId, userId, displayName }
typing:stop             -- { channelId, userId }
notification:new        -- { notification: NotificationDto }
```

**Client -> Server events**
```
typing:start            -- { channelId }
typing:stop             -- { channelId }
presence:update         -- { presence: 'ONLINE'|'AWAY'|'DND' }
channel:join            -- { channelId }   (join socket room)
channel:leave           -- { channelId }
```

---

## 4. Authentication & Authorization

### 4.1 JWT Flow
```
Login -> accessToken (15m, in response body) + refreshToken (7d, httpOnly cookie)
Client luu accessToken trong Zustand memory (khong localStorage)
Moi request dinh kem: Authorization: Bearer <accessToken>
Khi 401 -> TanStack Query interceptor goi POST /auth/refresh -> retry request
Logout -> goi POST /auth/logout -> xoa refreshToken cookie phia server
```

### 4.2 Authorization Levels
```
SystemAdmin     -- co the xem/quan ly toan bo
WorkspaceOwner  -- full quyen workspace
WorkspaceAdmin  -- quan ly channels, members
WorkspaceMember -- tham gia channel public, gui tin, v.v.
WorkspaceGuest  -- chi thay channel duoc assign

ChannelOwner  -- xoa channel, quan ly members
ChannelAdmin  -- kick member, pin message
ChannelMember -- gui/sua/xoa tin cua minh

Rule: Sua/xoa message -> chi author HOAC ChannelAdmin+
Rule: Archive channel -> ChannelOwner hoac WorkspaceAdmin+
```

### 4.3 NestJS Guards Order
```
JwtAuthGuard -> WorkspaceMemberGuard -> ChannelMemberGuard -> RoleGuard
```

---

## 5. Real-time Architecture

### 5.1 Socket.io Setup (NestJS)
```typescript
// Namespace: / (default)
// Adapter: khong can Redis adapter vi single-server
// Auth: handshake.auth.token = accessToken
// Khi connect:
//   1. Verify JWT
//   2. Join room user:{userId}
//   3. Query tat ca channelIds user la member
//   4. Join room channel:{channelId} cho tung channel
//   5. Broadcast user:presence:updated -> ONLINE
// Khi disconnect:
//   1. Broadcast user:presence:updated -> OFFLINE (sau 30s debounce)
```

### 5.2 Message Flow (gui tin)
```
Client POST /api/channels/:id/messages
  -> NestJS MessageService.create()
  -> Prisma insert Message
  -> Emit socket event message:new vao room channel:{channelId}
  -> Tao Notifications cho users duoc @mention
  -> Emit notification:new vao room user:{mentionedUserId}
  -> Return MessageDto
```

### 5.3 Typing Indicator
```
Client emit typing:start -> server nhan -> broadcast typing:start trong channel room (tru sender)
Neu sau 3s khong co typing:stop -> server tu clear (debounce Map<channelId, Map<userId, timeout>>)
```

---

## 6. File Upload

```
Multer middleware nhan multipart/form-data
Luu vao server/uploads/<workspaceId>/<channelId>/<uuid>.<ext>
Static serve: GET /uploads/:workspaceId/:channelId/:filename (protected by JwtAuthGuard)
Gioi han: 10MB per file, types: image/*, application/pdf, text/*, .zip
MessageAttachment record luu duong dan tuong doi
```

---

## 7. Search

```
PostgreSQL Full-Text Search tren Message.content
Migration tao:
  ALTER TABLE "Message" ADD COLUMN search_vector tsvector
    GENERATED ALWAYS AS (to_tsvector('english', content)) STORED;
  CREATE INDEX idx_message_fts ON "Message" USING GIN(search_vector);

Query: WHERE search_vector @@ plainto_tsquery('english', :q)
       AND channelId IN (:userChannelIds)   -- security filter
       ORDER BY ts_rank(...) DESC, createdAt DESC
       LIMIT 20 OFFSET :offset
```

---

## 8. Notification System

```
Trigger notification khi:
  1. Message @mention user (parse content tim @username)
  2. Reply vao thread cua user
  3. DM moi
  4. Duoc invite vao channel

NotificationService.create() -> insert DB -> emit socket notification:new
Client TanStack Query invalidate /notifications khi nhan socket event
Badge count = notifications.filter(n => !n.isRead).length
```

---

## 9. Frontend Architecture

### 9.1 Routing Structure (TanStack Router — File-based)
```
routes/
├── __root.tsx               -- root layout (providers, toast, socket init)
├── _auth/
│   ├── login.tsx
│   └── register.tsx
├── _protected/              -- layout check auth
│   ├── workspaces.tsx       -- chon workspace
│   └── $workspaceSlug/
│       ├── __layout.tsx     -- Sidebar + main area layout
│       ├── index.tsx        -- redirect den last channel
│       ├── channel/
│       │   └── $channelId/
│       │       ├── index.tsx       -- message list + input
│       │       └── thread.$msgId.tsx -- thread panel (right sidebar)
│       ├── dm/
│       │   └── $channelId.tsx
│       ├── search.tsx
│       ├── saved.tsx
│       └── settings/
│           ├── profile.tsx
│           ├── notifications.tsx
│           └── workspace.tsx  -- (admin only)
```

### 9.2 State Management

**Zustand Store** (`authStore`)
```typescript
{
  user: UserDto | null,
  accessToken: string | null,
  workspace: WorkspaceDto | null,  // current active workspace
  setAuth(user, token): void,
  clearAuth(): void,
  setWorkspace(workspace): void,
}
```

**TanStack Query Keys Convention**
```typescript
['workspaces']
['workspace', workspaceId]
['workspace', workspaceId, 'members']
['workspace', workspaceId, 'channels']
['channel', channelId]
['channel', channelId, 'messages']          -- infinite query
['channel', channelId, 'members']
['message', messageId, 'thread']           -- infinite query
['notifications']
['saved-items']
['search', workspaceId, queryParams]
```

**Socket Context** (React Context + useEffect)
```
socketStore (Zustand):
  socket: Socket | null
  connect(token): void
  disconnect(): void

Khi accessToken thay doi -> reconnect socket
Socket events -> invalidate TanStack Query keys hoac update cache truc tiep
```

### 9.3 Key Component Hierarchy
```
App
└── RouterProvider
    └── RootLayout (Providers: QueryClient, Toaster, SocketProvider)
        ├── AuthLayout (login/register pages)
        └── ProtectedLayout (check auth -> redirect neu chua dang nhap)
            └── WorkspaceLayout
                ├── Sidebar
                │   ├── WorkspaceSwitcher
                │   ├── SidebarSection "Channels"
                │   │   ├── ChannelItem (voi UnreadBadge)
                │   │   └── AddChannelButton
                │   ├── SidebarSection "Direct Messages"
                │   │   └── DMItem (voi PresenceIndicator)
                │   └── UserFooter (avatar, presence, settings)
                └── MainArea
                    ├── ChannelHeader (name, topic, member count, search)
                    ├── MessageList (virtualized, infinite scroll up)
                    │   ├── MessageGroup (grouped by date)
                    │   │   └── MessageItem
                    │   │       ├── MessageContent (markdown render)
                    │   │       ├── ReactionBar
                    │   │       ├── MessageAttachments
                    │   │       └── MessageActions (hover: reply, react, edit, bookmark...)
                    │   └── TypingIndicator
                    └── MessageInput
                        ├── RichTextEditor (contenteditable, @mention, :emoji:)
                        ├── AttachmentPicker
                        └── SendButton
```

### 9.4 Infinite Scroll Messages
```
useInfiniteQuery(['channel', channelId, 'messages'], ({ pageParam }) =>
  fetchMessages(channelId, { before: pageParam, limit: 50 })
)
Dung IntersectionObserver o top cua list -> fetchNextPage()
Khi nhan socket message:new -> append truc tiep vao query cache (khong refetch)
Khi navigate den channel -> scroll to bottom
Khi co unread -> "Jump to unread" button
```

---

## 10. Message Features Detail

### 10.1 Markdown Rendering
```
Ho tro: **bold**, _italic_, `code`, ```code block```, > blockquote, ~strikethrough~
Links auto-detect va render thanh <a>
@username mention -> highlight + clickable -> user profile popup
#channel-name -> clickable -> navigate den channel
:emoji: shortcode -> render emoji unicode
Dung thu vien: react-markdown hoac tu parse don gian voi regex
```

### 10.2 Thread (Reply to Message)
```
Message co parentId -> la thread reply
Thread panel mo o right sidebar khi click "Reply in Thread"
Thread root message hien thi "X replies, last reply from Y"
Socket events: message:new voi parentId -> update thread cache + update root message reply count
```

### 10.3 Reactions
```
Emoji picker (emoji-mart hoac tu build 1 grid nho)
Toggle: neu user da react emoji do -> remove, chua co -> add
Group reactions: { emoji: '👍', count: 3, users: ['A', 'B', 'C'], reacted: true }
Socket: reaction:updated broadcast toan channel voi full reaction state moi
```

### 10.4 Message Actions (hover menu)
```
Reply in Thread | React | Edit (author only) | Bookmark | Pin | Copy link | Delete (author/admin)
Pin message -> luu vao ChannelPin table, hien thi trong Channel Pins popup
```

---

## 11. Sidebar & UX Details

### 11.1 Unread Count
```
lastReadAt (ChannelMember) -> count messages WHERE createdAt > lastReadAt
Khi user mo channel -> PATCH /channels/:id/members/me/read (update lastReadAt = now)
Socket: message:new -> client tu tang unread count trong local state neu channel khong active
```

### 11.2 Presence System
```
Socket connect -> ONLINE
Socket disconnect -> debounce 30s -> OFFLINE
Manual set: emit presence:update voi 'AWAY' hoac 'DND'
Hien thi: green dot (ONLINE), yellow (AWAY), red (DND), grey (OFFLINE)
```

### 11.3 Channel Sidebar Sections
```
Starred / Favorites (user drag-drop de reorder)
Channels (public, joined private)
Direct Messages
```

---

## 12. Settings & Admin

### 12.1 User Settings
```
Profile: displayName, statusMessage, avatar upload
Notifications:
  - Global: All / Mentions / Nothing
  - Per-channel override
  - Desktop notification permission (Notification API)
Appearance: theme (light/dark), message display density
```

### 12.2 Workspace Settings (Admin)
```
General: name, icon, description
Members: list, role change, remove, invite link generation
Channels: list all, archive, delete
```

---

## 13. Security Considerations

```
- httpOnly cookie cho refreshToken -> chong XSS
- CORS chi allow client origin
- Rate limiting: @nestjs/throttler (login: 5 req/min, messages: 60 req/min)
- Input validation: Zod pipes tren moi DTO
- SQL injection: khong the qua Prisma parameterized queries
- File upload: validate mimeType bang file-type package (khong trust extension)
- Authorization check: moi resource access phai verify user la member cua workspace/channel
- Soft delete: khong xoa data that, chi set deletedAt
```

---

## 14. Environment & Config

### 14.1 Server (.env)
```
DATABASE_URL=postgresql://...
JWT_ACCESS_SECRET=...
JWT_REFRESH_SECRET=...
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
CLIENT_URL=http://localhost:5173
UPLOAD_DIR=./uploads
MAX_FILE_SIZE=10485760
PORT=3000
```

### 14.2 Client (.env)
```
VITE_API_URL=http://localhost:3000/api
VITE_SOCKET_URL=http://localhost:3000
```

---

## 15. Project Folder Structure

```
slack/
├── client/                    # Vite + React + TS
│   ├── src/
│   │   ├── routes/            # TanStack Router file-based routes
│   │   ├── components/        # Shared UI components
│   │   │   ├── ui/            # Shadcn generated components
│   │   │   ├── layout/        # Sidebar, Header, MainArea
│   │   │   ├── message/       # MessageItem, MessageInput, Thread...
│   │   │   ├── channel/       # ChannelList, ChannelHeader...
│   │   │   └── common/        # Avatar, Badge, Modal, Tooltip...
│   │   ├── hooks/             # Custom hooks (useSocket, useMessages...)
│   │   ├── stores/            # Zustand stores (auth, socket)
│   │   ├── lib/               # api client (axios instance), queryClient, utils
│   │   ├── types/             # Shared TypeScript types / DTOs
│   │   └── main.tsx
│   ├── index.html
│   └── package.json
│
└── server/                    # NestJS + TS
    ├── src/
    │   ├── auth/              # AuthModule: login, register, refresh, guards
    │   ├── users/             # UsersModule: profile, avatar, presence
    │   ├── workspaces/        # WorkspacesModule: CRUD, members
    │   ├── channels/          # ChannelsModule: CRUD, members, DM
    │   ├── messages/          # MessagesModule: CRUD, threads, reactions, pins
    │   ├── notifications/     # NotificationsModule
    │   ├── search/            # SearchModule: FTS queries
    │   ├── saved-items/       # SavedItemsModule
    │   ├── uploads/           # UploadsModule: multer, static serve
    │   ├── socket/            # SocketModule: Gateway, rooms management
    │   ├── prisma/            # PrismaModule: PrismaService
    │   ├── common/            # Guards, Pipes, Interceptors, Decorators, DTOs
    │   └── main.ts
    ├── prisma/
    │   ├── schema.prisma
    │   └── migrations/
    └── package.json
```
