# Task Breakdown — Slack Clone

> Cách đọc: Mỗi task được viết đủ chi tiết để AI biết chính xác cần làm gì. Con người chỉ cần đọc tiêu đề và bullet chức năng chính.
> Convention: `[BE]` = Backend NestJS, `[FE]` = Frontend React, `[DB]` = Database/Prisma.

---

## PHASE 0 — Project Setup & Infrastructure

### TASK-001 [BE] Khởi tạo NestJS project
- Chạy `npx @nestjs/cli new server --package-manager npm --skip-git`
- Cài dependencies: `@nestjs/config @nestjs/jwt @nestjs/throttler @nestjs/websockets @nestjs/platform-socket.io socket.io @prisma/client @prisma/adapter-pg pg bcryptjs zod multer @nestjs/platform-express file-type class-transformer`
- Dev deps: `prisma ts-node @types/multer @types/bcryptjs`
- Cấu hình `tsconfig.json`: `"experimentalDecorators": true, "emitDecoratorMetadata": true`
- Setup `ConfigModule.forRoot({ isGlobal: true })` trong AppModule
- Setup global `ValidationPipe` trong `main.ts` với custom Zod-based pipe
- Setup CORS: `app.enableCors({ origin: process.env.CLIENT_URL, credentials: true })`
- Setup cookie-parser: `app.use(cookieParser())`
- Static serve cho uploads: `app.useStaticAssets(join(__dirname, '..', 'uploads'), { prefix: '/uploads' })`

### TASK-002 [DB] Prisma Setup & Schema
- Chạy `npx prisma init`
- Viết toàn bộ `prisma/schema.prisma` với các model:
  - `User` (id uuid, email unique, username unique, displayName, passwordHash, avatarUrl, statusMessage, presence enum[ONLINE/AWAY/DND/OFFLINE], role enum[ADMIN/MEMBER], createdAt, updatedAt, deletedAt nullable)
  - `Workspace` (id uuid, name, slug unique, iconUrl, description, ownerId -> User, createdAt, updatedAt)
  - `WorkspaceMember` (id, workspaceId -> Workspace, userId -> User, role enum[OWNER/ADMIN/MEMBER/GUEST], displayName, joinedAt) — unique(workspaceId, userId)
  - `Channel` (id uuid, workspaceId -> Workspace, name, description, topic, type enum[PUBLIC/PRIVATE/DM/GROUP_DM], createdById -> User, isArchived bool default false, createdAt, updatedAt)
  - `ChannelMember` (id, channelId -> Channel, userId -> User, role enum[OWNER/ADMIN/MEMBER], lastReadAt, notificationPreference enum[ALL/MENTIONS/NOTHING] default ALL, joinedAt) — unique(channelId, userId)
  - `Message` (id uuid, channelId -> Channel, userId -> User, content text, type enum[TEXT/SYSTEM/BOT] default TEXT, parentId -> Message nullable (self-relation), editedAt nullable, createdAt, updatedAt, deletedAt nullable)
  - `MessageAttachment` (id uuid, messageId -> Message, fileUrl, fileName, fileSize int, mimeType, createdAt)
  - `Reaction` (id, messageId -> Message, userId -> User, emoji string, createdAt) — unique(messageId, userId, emoji)
  - `Notification` (id uuid, userId -> User, type enum[MENTION/REPLY/CHANNEL_INVITE/DM], referenceId string, isRead bool default false, createdAt)
  - `SavedItem` (id, userId -> User, messageId -> Message, createdAt) — unique(userId, messageId)
  - `RefreshToken` (id, userId -> User, token string, expiresAt, createdAt)
  - `ChannelPin` (id, channelId -> Channel, messageId -> Message, pinnedById -> User, pinnedAt) — unique(channelId, messageId)
- Chạy `npx prisma migrate dev --name init`
- Sau migrate, chạy raw SQL để thêm full-text search index:
  ```sql
  ALTER TABLE "Message" ADD COLUMN IF NOT EXISTS search_vector tsvector
    GENERATED ALWAYS AS (to_tsvector('english', COALESCE(content, ''))) STORED;
  CREATE INDEX IF NOT EXISTS idx_message_fts ON "Message" USING GIN(search_vector);
  ```
- Tạo `PrismaModule` global (PrismaService extends PrismaClient, onModuleInit gọi $connect)
- Tạo `prisma/seed.ts` với 1 admin user mặc định và 1 workspace demo

### TASK-003 [FE] Khởi tạo React + Vite project
- Chạy `npm create vite@latest client -- --template react-ts`
- Cài dependencies: `@tanstack/react-router @tanstack/react-query @tanstack/react-router-devtools zustand axios socket.io-client react-hook-form zod @hookform/resolvers react-markdown remark-gfm emoji-mart @emoji-mart/react @emoji-mart/data date-fns clsx tailwind-merge lucide-react`
- Cài Tailwind CSS v4: theo hướng dẫn official (vite plugin)
- Cài Shadcn UI: `npx shadcn@latest init` — chọn style "New York", base color "Zinc", CSS variables yes
- Cài các Shadcn components cần: `button input textarea avatar badge dropdown-menu dialog sheet tooltip popover command scroll-area separator skeleton tabs`
- Cấu hình TanStack Router: thêm `@tanstack/router-vite-plugin` vào `vite.config.ts`, tạo `src/routeTree.gen.ts` (auto-generated)
- Tạo `src/lib/axios.ts`: axios instance với baseURL từ `VITE_API_URL`, interceptor tự động thêm Authorization header từ authStore, interceptor response bắt 401 -> gọi refresh -> retry
- Tạo `src/lib/queryClient.ts`: QueryClient với defaultOptions staleTime 60s

---

## PHASE 1 — Authentication

### TASK-011 [BE] Auth Module
- Tạo `AuthModule` với: `AuthController`, `AuthService`, `JwtStrategy`, `RefreshTokenStrategy`
- `POST /auth/register`:
  - Zod schema validate: email, username (3-20 chars, alphanumeric+underscore), password (min 8)
  - Check unique email và username
  - bcryptjs hash password với salt 12
  - Insert User, tạo accessToken + refreshToken, lưu refreshToken hash vào RefreshToken table
  - Set refreshToken vào httpOnly cookie (sameSite: 'lax', secure: false cho dev)
  - Return: `{ user: UserDto, accessToken: string }`
- `POST /auth/login`:
  - Tìm user bằng email, verify password bằng bcryptjs.compare
  - Tạo accessToken (JWT, payload: userId + email, expires 15m)
  - Tạo refreshToken (JWT, payload: userId, expires 7d), lưu hash vào DB
  - Return: `{ user: UserDto, accessToken: string }` + set cookie
- `POST /auth/refresh`:
  - Đọc refreshToken từ cookie
  - Verify JWT, tìm trong DB (hash match)
  - Rotate: xóa old refresh token, tạo cặp mới
  - Return: `{ accessToken: string }`
- `POST /auth/logout`:
  - Xóa RefreshToken record trong DB
  - Clear cookie
  - Return 200
- `GET /auth/me` (JwtAuthGuard): Return current user từ JWT payload
- `PATCH /auth/me` (JwtAuthGuard): Update displayName, statusMessage
- `POST /auth/me/avatar` (JwtAuthGuard): Multer single file upload, update avatarUrl
- **JwtAuthGuard**: implement `canActivate` verify accessToken từ Authorization header
- **UserDto**: `{ id, email, username, displayName, avatarUrl, statusMessage, presence, role, createdAt }`

### TASK-012 [FE] Auth Pages & Store
- **Zustand authStore** (`src/stores/auth.store.ts`):
  ```typescript
  interface AuthState {
    user: UserDto | null
    accessToken: string | null
    isAuthenticated: boolean
    setAuth: (user: UserDto, token: string) => void
    updateUser: (user: Partial<UserDto>) => void
    clearAuth: () => void
  }
  ```
  Dùng `persist` middleware của Zustand lưu vào sessionStorage (không localStorage vì security)
- **Axios interceptor**: trước mỗi request lấy `authStore.getState().accessToken` gắn vào header. Response interceptor: nếu 401 và chưa đang refresh -> gọi POST /auth/refresh -> update token -> retry request gốc
- **Route `/login`** (`src/routes/_auth/login.tsx`):
  - Form: email + password với React Hook Form + Zod
  - Gọi POST /auth/login, lưu vào authStore, navigate đến `/workspaces`
  - Link "Chưa có tài khoản? Đăng ký"
  - UI: card centered, gradient background, logo placeholder
- **Route `/register`** (`src/routes/_auth/register.tsx`):
  - Form: email + username + password + confirmPassword
  - Validation với Zod, gọi POST /auth/register
  - Auto login sau register
- **Protected Route layout** (`src/routes/_protected.tsx`):
  - beforeLoad: check `authStore.isAuthenticated`, nếu không -> redirect `/login`
  - Nếu có token nhưng user null -> gọi GET /auth/me để hydrate user
- **Root layout** (`src/routes/__root.tsx`): Wrap QueryClientProvider + Toaster (Shadcn Sonner)

---

## PHASE 2 — Workspace

### TASK-021 [BE] Workspace Module
- Tạo `WorkspacesModule` với Controller, Service
- **WorkspaceMemberGuard**: middleware check user là thành viên của workspace từ `params.workspaceId` hoặc `params.wid`. Attach vào request `req.workspaceMember` với role.
- `GET /workspaces`: Trả workspaces user là member, join với WorkspaceMember để lấy role
- `POST /workspaces`: Tạo workspace mới
  - Validate: name (2-80 chars), generate slug từ name (slugify + unique suffix nếu trùng)
  - Insert Workspace, insert WorkspaceMember với role OWNER
  - Tự động tạo channel #general (PUBLIC) và add owner vào channel đó
  - Return WorkspaceDto với channels
- `GET /workspaces/:id`: Chi tiết workspace + thông tin member hiện tại
- `PATCH /workspaces/:id` (WorkspaceAdmin+): update name, description, iconUrl
- `DELETE /workspaces/:id` (WorkspaceOwner): soft delete workspace và tất cả resources
- `GET /workspaces/:id/members`: Danh sách members với user info, sort bằng role rồi displayName
- `POST /workspaces/:id/members/invite` (WorkspaceAdmin+): tìm user bằng email -> add vào workspace với role MEMBER -> emit socket `workspace:member:joined`
- `PATCH /workspaces/:id/members/:userId` (WorkspaceAdmin+): đổi role (không thể đổi OWNER)
- `DELETE /workspaces/:id/members/:userId` (WorkspaceAdmin+): xóa member, xóa ChannelMember tương ứng -> emit socket `workspace:member:left`
- **WorkspaceDto**: `{ id, name, slug, iconUrl, description, role, memberCount, createdAt }`

### TASK-022 [FE] Workspace Selection & Layout
- **Route `/workspaces`** (`src/routes/_protected/workspaces.tsx`):
  - Query GET /workspaces (TanStack Query key: `['workspaces']`)
  - Nếu chỉ có 1 workspace -> tự động redirect `/$workspaceSlug`
  - Hiển thị danh sách workspace cards để chọn
  - Button "Tạo workspace mới" -> open Dialog với form tạo workspace
- **WorkspaceLayout** (`src/routes/_protected/$workspaceSlug/__layout.tsx`):
  - beforeLoad: gọi GET /workspaces/:slug, lưu vào authStore.workspace
  - Nếu không phải member -> redirect `/workspaces`
  - Layout: `flex h-screen` với Sidebar (260px cố định) + MainArea (flex-1)
- **Sidebar** (`src/components/layout/Sidebar.tsx`):
  - Header: workspace name + icon + dropdown (settings, invite people, ...)
  - Channels section: danh sách channels đã join, badge unread
  - DMs section: danh sách DMs với presence indicator
  - Footer: avatar user + displayName + status + settings icon
- **WorkspaceSwitcher** (`src/components/layout/WorkspaceSwitcher.tsx`):
  - Dropdown hiển thị tất cả workspaces của user
  - Click -> navigate đến workspace đó

---

## PHASE 3 — Channels

### TASK-031 [BE] Channel Module
- Tạo `ChannelsModule`
- **ChannelMemberGuard**: check user là member của channel từ `params.channelId` hoặc `params.cid`. Attach `req.channelMember`.
- `GET /workspaces/:wid/channels`: Trả channels user có thể thấy:
  - Tất cả PUBLIC channels trong workspace
  - PRIVATE channels user là member
  - DM/GROUP_DM channels user là member
  - Mỗi channel kèm `unreadCount` (count messages WHERE createdAt > user's lastReadAt) và `lastMessage`
- `POST /workspaces/:wid/channels` (WorkspaceMember+):
  - Validate: name (lowercase, hyphen-separated, 2-80 chars), type (PUBLIC/PRIVATE)
  - Insert Channel, insert ChannelMember với role OWNER
  - Emit socket `channel:new` đến workspace room nếu PUBLIC
  - Return ChannelDto
- `GET /workspaces/:wid/channels/:id` (ChannelMemberGuard): Chi tiết channel + member count
- `PATCH /workspaces/:wid/channels/:id` (ChannelAdmin+): update name, description, topic
- `DELETE /workspaces/:wid/channels/:id` (ChannelOwner hoặc WorkspaceAdmin+): archive channel (set isArchived = true), emit `channel:deleted`
- `POST /workspaces/:wid/channels/:id/members` (ChannelMember): invite user vào channel. Check user là workspace member. Insert ChannelMember. Emit `member:joined`.
- `DELETE /workspaces/:wid/channels/:id/members/:uid` (ChannelAdmin+ hoặc self-leave): xóa ChannelMember. Emit `member:left`.
- `PATCH /workspaces/:wid/channels/:id/members/me/read`: Update `lastReadAt = now()` cho current user. Dùng sau khi mở channel.
- `POST /workspaces/:wid/channels/dm` (WorkspaceMember): 
  - Nhận `{ targetUserId }`, tìm existing DM channel giữa 2 user trong workspace
  - Nếu chưa có: tạo Channel type=DM (name = userId1_userId2 sorted), thêm 2 ChannelMember
  - Return ChannelDto
- `POST /workspaces/:wid/channels/group-dm` (WorkspaceMember):
  - Nhận `{ userIds: string[] }` (2-8 users), tạo Channel type=GROUP_DM
  - Add tất cả users + current user vào channel
- **ChannelDto**: `{ id, name, description, topic, type, isArchived, memberCount, unreadCount, lastMessage, createdAt }`

### TASK-032 [FE] Channel Sidebar & Navigation
- **SidebarSection** (`src/components/layout/SidebarSection.tsx`): Collapsible section với title và children list
- **ChannelItem** (`src/components/channel/ChannelItem.tsx`):
  - Hiển thị # icon + channel name + unread badge (số đỏ nếu > 0)
  - Active state khi đang ở channel đó (highlight background)
  - Hover: hiển thị icon "..." cho options (leave channel, mute, ...)
  - Click -> navigate đến channel route
- **DMItem** (`src/components/channel/DMItem.tsx`):
  - Hiển thị Avatar với PresenceIndicator + displayName + unread badge
- **AddChannelButton**: Opens Dialog để tạo channel hoặc browse channels
- **BrowseChannelsDialog**: List tất cả public channels trong workspace, search, join button
- **CreateChannelDialog**: Form tạo channel mới (name, description, public/private toggle)
- **CreateDMDialog**: Search users trong workspace, click để mở DM
- Query `['workspace', workspaceId, 'channels']` -> refetch khi nhận socket `channel:new`, `channel:updated`, `channel:deleted`

### TASK-033 [FE] Channel Header & Info
- **ChannelHeader** (`src/components/channel/ChannelHeader.tsx`):
  - Left: channel name (click -> open channel info sheet), topic nếu có
  - Right: member count icon (click -> open members panel), search icon, pins icon
- **ChannelInfoSheet** (Shadcn Sheet ở right): name, description, topic, created by, member count
- **ChannelMembersPanel** (Sheet): Danh sách members với role, presence indicator, click user -> user profile popup
- **ChannelPinsPanel** (Sheet): Danh sách pinned messages, click -> jump đến message

---

## PHASE 4 — Messages (Core)

### TASK-041 [BE] Message Module - CRUD
- Tạo `MessagesModule`
- `GET /channels/:cid/messages` (ChannelMemberGuard):
  - Cursor-based pagination: nhận query `{ before?: string (messageId), limit: 50 }`
  - Nếu có `before`: `WHERE id < before AND channelId = :cid AND parentId IS NULL AND deletedAt IS NULL`
  - Nếu không có `before`: lấy 50 tin mới nhất
  - ORDER BY createdAt DESC, trả về sorted ASC cho client
  - Include: user info, attachments, reactions (grouped), thread reply count
  - Return: `{ messages: MessageDto[], hasMore: boolean, nextCursor: string | null }`
- `POST /channels/:cid/messages` (ChannelMemberGuard):
  - Body: `{ content: string (1-4000 chars), parentId?: string }`
  - Validate parentId nếu có: phải thuộc cùng channel
  - Insert Message
  - Parse @mentions: regex `/@(\w+)/g` -> tìm users bằng username trong workspace -> tạo Notifications
  - Emit socket `message:new` vào room `channel:{cid}` với full MessageDto
  - Nếu có parentId: emit socket `message:updated` cho root message (update replyCount)
  - Nếu tạo notification: emit `notification:new` vào `user:{userId}`
  - Return MessageDto
- `PATCH /channels/:cid/messages/:id` (ChannelMemberGuard + author only):
  - Validate content
  - Update Message: content, editedAt = now()
  - Emit `message:updated`
  - Return MessageDto
- `DELETE /channels/:cid/messages/:id` (author hoặc ChannelAdmin+):
  - Soft delete: set deletedAt = now(), set content = "[Message deleted]"
  - Emit `message:deleted` với `{ messageId, channelId }`
- `GET /channels/:cid/messages/:id/thread` (ChannelMemberGuard):
  - Lấy root message + tất cả replies WHERE parentId = :id
  - Same cursor pagination, include full info
  - Return `{ rootMessage: MessageDto, replies: MessageDto[], hasMore: boolean }`
- **MessageDto**: 
  ```typescript
  {
    id, channelId, content, type, parentId, editedAt, createdAt, deletedAt,
    user: { id, username, displayName, avatarUrl, presence },
    attachments: AttachmentDto[],
    reactions: { emoji: string, count: number, users: string[], reacted: boolean }[],
    replyCount: number,
    lastReplyAt: Date | null,
    lastReplyUser: { username, displayName, avatarUrl } | null
  }
  ```

### TASK-042 [BE] Message Reactions & Attachments
- `POST /channels/:cid/messages/:id/reactions` (ChannelMemberGuard):
  - Body: `{ emoji: string }` (validate là emoji unicode, max 10 chars)
  - Toggle: nếu tồn tại unique(messageId, userId, emoji) -> delete, nếu không -> insert
  - Query lại tất cả reactions của message, group và compute `reacted` flag
  - Emit `reaction:updated` vào `channel:{cid}` với `{ messageId, reactions }`
- `POST /channels/:cid/messages/:id/attachments` (ChannelMemberGuard + author):
  - Multer: single file, max 10MB, filter mimeType (image/*, application/pdf, text/*, application/zip)
  - Validate file thật bằng `file-type` package (đọc magic bytes)
  - Save đến `uploads/{workspaceId}/{channelId}/{uuid}.{ext}`
  - Insert MessageAttachment, return AttachmentDto
- `POST /channels/:cid/messages` (thêm tính năng): Nhận cả file upload cùng message trong 1 request qua multipart (optional field `file`)

### TASK-043 [BE] Channel Pins
- Thêm vào MessagesModule:
- `POST /channels/:cid/messages/:id/pin` (ChannelAdmin+):
  - Insert ChannelPin, emit `channel:updated` (hoặc custom event `message:pinned`)
- `DELETE /channels/:cid/messages/:id/pin` (ChannelAdmin+):
  - Delete ChannelPin, emit tương tự
- `GET /channels/:cid/pins` (ChannelMemberGuard):
  - Lấy tất cả ChannelPin của channel, join với Message và User
  - Return list MessageDto

### TASK-044 [FE] Message List (Virtualized)
- **MessageList** (`src/components/message/MessageList.tsx`):
  - Dùng `useInfiniteQuery` key `['channel', channelId, 'messages']`
  - Scroll container với `overflow-y: auto`
  - IntersectionObserver ở top element -> khi visible gọi `fetchNextPage()`
  - Khi data load thêm (older messages): giữ nguyên scroll position (tính delta height và set scrollTop)
  - Khi channel mới open: scroll to bottom
  - Khi có tin mới đến (socket): nếu user đang ở bottom (scrollTop + clientHeight >= scrollHeight - 50) -> scroll to bottom; nếu không -> hiện "X new messages" banner
  - **MessageGroup**: Group messages theo ngày, hiển thị date divider ("Today", "Yesterday", "Jan 12, 2024")
  - Messages từ cùng author trong vòng 5 phút được gom nhóm (chỉ hiện avatar + name ở tin đầu)
- **TypingIndicator** (`src/components/message/TypingIndicator.tsx`):
  - Hiển thị "A đang nhập..." hoặc "A, B đang nhập..." hoặc "Nhiều người đang nhập..."
  - Zustand typingStore: `Map<channelId, Set<userId>>`
  - Nhận socket `typing:start` -> add, `typing:stop` -> remove

### TASK-045 [FE] MessageItem Component
- **MessageItem** (`src/components/message/MessageItem.tsx`):
  - Layout: Avatar (left, 36px) + Content (right)
  - Avatar + Name + Timestamp chỉ hiện ở tin đầu của group
  - Content: render MessageContent (markdown)
  - Hover: hiển thị **MessageActions** toolbar (floating, absolute positioned)
  - Nếu `deletedAt` không null: hiển thị italic gray "[Tin nhắn đã bị xóa]", không hiện actions
  - Nếu `editedAt` không null: thêm text " (đã chỉnh sửa)" nhỏ sau content
  - Hiển thị AttachmentList nếu có attachments
  - Hiển thị ReactionBar nếu có reactions
  - Hiển thị ThreadPreview nếu `replyCount > 0` (chỉ trong channel view)
- **MessageContent** (`src/components/message/MessageContent.tsx`):
  - Dùng `react-markdown` với `remark-gfm`
  - Custom renderer cho @mention: wrap bằng `<span class="mention">` có highlight và tooltip
  - Custom renderer cho #channel: clickable navigate
  - Custom renderer cho link: mở tab mới
  - Code block: syntax highlight đơn giản (pre + code styling)
- **MessageActions** (`src/components/message/MessageActions.tsx`):
  - Toolbar icons: 😊 (react), ↩ (reply in thread), ✏️ (edit, only author), 🔖 (bookmark), 📌 (pin, only admin), 🗑️ (delete, author or admin)
  - Clicking react -> mở EmojiPicker popover

### TASK-046 [FE] MessageInput Component
- **MessageInput** (`src/components/message/MessageInput.tsx`):
  - `<div contenteditable>` hoặc `<textarea>` với auto-resize
  - Toolbar trên cùng: Bold, Italic, Code, Link, Ordered List, Unordered List, Quote
  - **@mention autocomplete**: khi gõ `@` -> mở dropdown tìm members, ArrowUp/Down để chọn, Enter để insert
  - **:emoji: autocomplete**: khi gõ `:` + 2 ký tự -> dropdown tìm emoji, same navigation
  - **Shift+Enter**: xuống dòng. **Enter**: submit (nếu không có Shift)
  - Paste hình ảnh: bắt `paste` event, nếu có file image -> preview + auto attach
  - Drag-drop file: khi drag file vào input area -> preview attachment
  - Attachment preview: thumbnail list phía dưới input, nút X để remove
  - Submit: POST /channels/:cid/messages, nếu có file -> POST /channels/:cid/messages/:id/attachments sau đó
  - Trong thread: prop `parentId` -> gửi reply
- **EmojiPicker** (`src/components/common/EmojiPicker.tsx`):
  - Dùng `@emoji-mart/react` wrapped trong Popover
  - Trên click reaction icon trong MessageActions hoặc trong MessageInput toolbar

### TASK-047 [FE] Real-time Message Sync
- **useSocketMessages** hook (`src/hooks/useSocketMessages.ts`):
  - Subscribe socket events khi mount, unsubscribe khi unmount
  - `message:new`: nếu `message.channelId === activeChannelId` -> append vào query cache `['channel', channelId, 'messages']`. Invalidate `['workspace', wid, 'channels']` để cập nhật lastMessage
  - `message:updated`: update message trong cache bằng messageId
  - `message:deleted`: update message trong cache (set deletedAt, content)
  - `reaction:updated`: update reactions của message trong cache
  - `typing:start` / `typing:stop`: update typingStore
  - Sau khi nhận `message:new` trong channel đang active: gọi PATCH /channels/:cid/members/me/read

---

## PHASE 5 — Direct Messages & Real-time

### TASK-051 [BE+FE] Socket Gateway
- **[BE] SocketGateway** (`src/socket/socket.gateway.ts`):
  - `@WebSocketGateway({ cors: { origin: process.env.CLIENT_URL, credentials: true } })`
  - `handleConnection(client: Socket)`:
    1. Đọc `client.handshake.auth.token`
    2. Verify JWT bằng JwtService
    3. Nếu invalid -> `client.disconnect()`
    4. Attach `client.data.userId` = userId từ JWT
    5. Join room `user:{userId}`
    6. Query tất cả channelIds user là member (ChannelMember WHERE userId = userId AND channel isArchived = false)
    7. Join tất cả `channel:{channelId}` rooms
    8. Update User.presence = ONLINE trong DB
    9. Broadcast `user:presence:updated` { userId, presence: 'ONLINE' } đến room `workspace:{workspaceId}` (cần lấy workspaceId của user)
  - `handleDisconnect(client: Socket)`:
    1. Set timeout 30s -> update presence OFFLINE nếu không reconnect
    2. Broadcast `user:presence:updated` { userId, presence: 'OFFLINE' }
  - `@SubscribeMessage('typing:start')`: broadcast đến `channel:{channelId}` (trừ sender) event `typing:start` với `{ channelId, userId, displayName }`; debounce auto-stop sau 3s
  - `@SubscribeMessage('typing:stop')`: broadcast `typing:stop`
  - `@SubscribeMessage('presence:update')`: update DB và broadcast
  - `@SubscribeMessage('channel:join')`: client.join(`channel:{channelId}`) — dùng sau khi join channel mới
  - Tạo `SocketService` (injectable) với method `emitToChannel(channelId, event, data)` và `emitToUser(userId, event, data)` để các service khác inject và dùng
- **[FE] Socket Store** (`src/stores/socket.store.ts`):
  ```typescript
  interface SocketState {
    socket: Socket | null
    isConnected: boolean
    connect: (token: string) => void
    disconnect: () => void
    emit: (event: string, data: any) => void
  }
  ```
  - `connect()`: khởi tạo socket.io-client với `{ auth: { token } }`
  - Setup event listeners: `connect`, `disconnect`, `connect_error`
  - Khi accessToken thay đổi trong authStore -> reconnect socket
- **[FE] SocketProvider** (`src/components/providers/SocketProvider.tsx`):
  - useEffect khi mount + khi accessToken change -> socketStore.connect(token)
  - useEffect cleanup -> socketStore.disconnect()
  - Context cung cấp socket instance cho toàn app

### TASK-052 [FE] Direct Message Flow
- **DM Flow**:
  - Click user trong sidebar hoặc member list -> gọi `POST /workspaces/:wid/channels/dm { targetUserId }` -> nhận channelId -> navigate đến `/channel/{channelId}`
  - DM channel route dùng chung MessageList + MessageInput với regular channel
  - Phân biệt qua `channel.type === 'DM'` để ẩn invite button, hiện partner's name ở header
- **Group DM**: Tương tự, header hiển thị danh sách tên members

---

## PHASE 6 — Thread

### TASK-061 [FE] Thread Panel
- **ThreadPanel** (`src/components/message/ThreadPanel.tsx`):
  - Hiển thị ở right sidebar (Sheet hoặc fixed panel 360px)
  - Header: "Thread" + channel name + close button
  - Root message hiển thị đầy đủ (MessageItem không grouped)
  - Divider "X replies"
  - Danh sách replies (MessageList variant, không grouped, infinite scroll)
  - MessageInput với prop `parentId={rootMessageId}` ở cuối
  - Query: `useInfiniteQuery(['message', messageId, 'thread'], ...)`
- **ThreadPreview** (`src/components/message/ThreadPreview.tsx`):
  - Hiển thị bên dưới message gốc trong channel view: avatars của repliers + "X replies" + "Last reply from Y at Z"
  - Click -> mở ThreadPanel

---

## PHASE 7 — Search

### TASK-071 [BE] Search Module
- Tạo `SearchModule`
- `GET /workspaces/:wid/search/messages?q=&channelId=&userId=&before=&after=&offset=0&limit=20`:
  - Validate user là member của workspace
  - Lấy tất cả channelIds user có access (PUBLIC + PRIVATE user là member)
  - Full-text search: `WHERE search_vector @@ plainto_tsquery('english', :q) AND "channelId" = ANY(:channelIds) AND "deletedAt" IS NULL`
  - Thêm filters: channelId, userId (from), date range nếu có
  - ORDER BY `ts_rank(search_vector, ...) DESC, "createdAt" DESC`
  - Return: `{ results: SearchResultDto[], total: number, offset: number }`
  - `SearchResultDto`: MessageDto với thêm `highlight` (snippet với từ khóa highlighted)
- `GET /workspaces/:wid/search/channels?q=`: Tìm channels theo name
- `GET /workspaces/:wid/search/people?q=`: Tìm members theo username/displayName
- `GET /workspaces/:wid/search/files?q=`: Tìm attachments theo fileName

### TASK-072 [FE] Search UI
- **SearchBar** trong ChannelHeader: focus -> mở SearchModal (Command dialog)
- **SearchModal** (`src/components/search/SearchModal.tsx`):
  - Input với debounce 300ms
  - Tabs: Messages | Files | Channels | People
  - Hiển thị kết quả với highlight từ khóa
  - Click result -> navigate đến channel/message
  - Filters sidebar: channel filter, date range, from user

---

## PHASE 8 — Notifications & Saved Items

### TASK-081 [BE] Notification Module
- Tạo `NotificationsModule`
- `GET /notifications?unreadOnly=true&limit=20&offset=0`:
  - Lấy notifications của current user, sort createdAt DESC
  - Join với referenced message/channel để lấy preview
  - Return `{ notifications: NotificationDto[], unreadCount: number }`
- `PATCH /notifications/read-all`: Update tất cả isRead = true WHERE userId = currentUser
- `PATCH /notifications/:id/read`: Update 1 notification isRead = true
- `NotificationDto`: `{ id, type, isRead, createdAt, referenceId, channel: { id, name }, message: { id, content snippet }, fromUser: UserMinimalDto }`

### TASK-082 [BE] Saved Items Module
- Tạo `SavedItemsModule`
- `GET /saved-items`: Lấy saved items của user, include full message với channel info
- `POST /saved-items` body `{ messageId }`: Check unique, insert SavedItem
- `DELETE /saved-items/:id`: Delete SavedItem

### TASK-083 [FE] Notification Bell & Panel
- **NotificationBell** trong Sidebar footer hoặc header:
  - Badge số unread count (query `['notifications']`)
  - Nhận socket `notification:new` -> invalidate query + hiện toast
  - Click -> mở NotificationPanel
- **NotificationPanel** (Sheet):
  - List notifications, nhóm theo "Hôm nay", "Hôm qua", "Tuần này"
  - Click notification -> navigate đến message (jump to message in channel)
  - "Đánh dấu tất cả đã đọc" button

### TASK-084 [FE] Saved Items Page
- **Route `/saved`**: Danh sách saved items
- Grouped by workspace/channel
- Click -> navigate đến message gốc
- Unsave button (X) trên mỗi item

---

## PHASE 9 — Settings & Admin

### TASK-091 [FE] User Settings
- **Route `/settings/profile`**: 
  - Form cập nhật displayName, statusMessage
  - Avatar upload: click avatar -> file picker -> preview -> submit POST /auth/me/avatar
  - Form cập nhật username, email (cần xác nhận password)
- **Route `/settings/notifications`**:
  - Toggle global notification preference
  - Per-channel overrides: list channels với dropdown preference
  - Desktop notifications: button enable/disable (Notification API)
- **Route `/settings/appearance`**:
  - Theme: Light / Dark / System (dùng next-themes hoặc class-based toggle)
  - Message display: Comfortable / Compact

### TASK-092 [FE+BE] Workspace Admin Settings
- **Route `/settings/workspace`** (chỉ WorkspaceAdmin+):
  - **General tab**: upload workspace icon, edit name/description -> PATCH /workspaces/:id
  - **Members tab**: 
    - Danh sách members với search, sort
    - Dropdown role per member
    - Remove member button -> DELETE /workspaces/:id/members/:uid
    - "Invite member" button -> input email -> POST /workspaces/:id/members/invite
  - **Channels tab**: list tất cả channels (kể cả archived), archive/unarchive button
- **[BE]**: Thêm `GET /workspaces/:id/channels/all` (admin only): trả cả archived channels

---

## PHASE 10 — Polish & Edge Cases

### TASK-101 [FE] UX Polish
- **Unread management**:
  - Khi navigate đến channel: gọi PATCH /channels/:cid/members/me/read
  - Nếu có unread messages: hiện "Jump to first unread" button
  - Sidebar channel item: bold name khi có unread
- **Keyboard shortcuts**:
  - `Ctrl+K`: Mở quick switcher (search channels + DMs)
  - `Escape`: Đóng thread panel, đóng modal
  - `Alt+ArrowUp/Down`: Navigate giữa channels
- **Empty states**: Channel mới tạo hiện empty state với hướng dẫn, DM empty state

### TASK-102 [FE] Performance
- **Message list virtualization**: Nếu list > 200 items, dùng `react-virtual` để virtualize
- **Image lazy loading**: `loading="lazy"` + IntersectionObserver
- **Query prefetching**: khi hover ChannelItem -> prefetch query messages
- **Optimistic updates**: 
  - Gửi message: append message với trạng thái "sending" vào cache ngay, sau khi API thành công replace bằng real message. Nếu lỗi: hiện "Retry" button
  - Toggle reaction: update local cache ngay, sau đó sync với server response

### TASK-103 [BE] Rate Limiting & Security
- Apply `@Throttle()` decorator:
  - Auth endpoints: 5 requests/minute
  - Message create: 30 messages/minute per user
  - File upload: 10 uploads/minute per user
- Sanitize message content: strip HTML tags, chỉ allow markdown syntax
- Validate file magic bytes (không chỉ extension/mimetype từ client)
- Middleware log tất cả requests với IP, userId, endpoint

### TASK-104 [FE] Error Handling & Loading States
- **Skeleton loaders**: MessageListSkeleton, SidebarSkeleton, ChannelHeaderSkeleton
- **Error boundaries**: Wrap channel view, thread panel với React ErrorBoundary
- **Retry logic**: TanStack Query retry 2 lần với exponential backoff (trừ 4xx errors)
- **Toast notifications** (Shadcn Sonner):
  - Thành công: "Message sent", "Channel created", ...
  - Lỗi: "Failed to send message. Retry?", ...
  - Thông báo real-time: "@username mentioned you in #channel"

### TASK-105 [FE] Responsive Layout
- Trên mobile (< 768px): sidebar ẩn, hiện hamburger menu -> Drawer
- Thread panel: trên desktop overlay bên phải; trên tablet -> full sheet
- MessageInput: trên mobile remove toolbar formatting buttons, giữ core

---

## Thứ tự thực hiện (Recommended)

```
Phase 0 (Setup) → Phase 1 (Auth) → Phase 2 (Workspace) → Phase 3 (Channels)
→ Phase 4 (Messages Core) → Phase 5 (DM + Socket) → Phase 6 (Thread)
→ Phase 7 (Search) → Phase 8 (Notifications) → Phase 9 (Settings)
→ Phase 10 (Polish)
```

## Dependencies giữa các tasks

- TASK-002 (DB Schema) phải xong trước tất cả BE tasks
- TASK-011 (Auth BE) phải xong trước mọi FE task cần API
- TASK-051 (Socket Gateway) phải xong trước TASK-047, TASK-052
- TASK-031 (Channel BE) phải xong trước TASK-041 (Message BE)
- TASK-044, 045, 046 có thể làm song song
