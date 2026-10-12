# UI/UX Design Specification — Slack Clone
> **Audience**: AI design tools (Google Stitch, v0, Lovable, etc.) và frontend developers.
> **Purpose**: Mô tả chi tiết layout, visual hierarchy, màu sắc, typography, spacing, interactions cho từng màn hình. Đủ chi tiết để sinh ra code giao diện chính xác mà không cần hỏi thêm.

---

## 1. Design System Foundation

### 1.1 Color Palette

```
PRIMARY DARK THEME (default):
  --bg-base:          #1A1D21   /* Nền ứng dụng toàn cục - gần đen */
  --bg-sidebar:       #19171D   /* Sidebar nền - tím đậm rất nhẹ */
  --bg-main:          #222529   /* Main content area */
  --bg-elevated:      #2B2D31   /* Card, modal, hover states */
  --bg-hover:         #35373B   /* Hover trên items */
  --bg-active:        #3D3F44   /* Active/selected item */
  --bg-input:         #1A1D21   /* Input backgrounds */
  --bg-overlay:       rgba(0,0,0,0.7)  /* Modal backdrop */

  --text-primary:     #D1D2D3   /* Main text */
  --text-secondary:   #ABABAD   /* Secondary labels, timestamps */
  --text-muted:       #7B7D82   /* Placeholder, disabled */
  --text-on-dark:     #FFFFFF   /* Text on colored backgrounds */
  --text-link:        #5CAAFF   /* Links, mentions, channel refs */
  --text-mention-bg:  rgba(92,170,255,0.2) /* @mention highlight bg */

  --accent-purple:    #7C5CBF   /* Workspace brand color (Slack purple) */
  --accent-purple-hover: #8B6BD0
  --accent-green:     #3DB077   /* Online presence, success */
  --accent-yellow:    #E8B430   /* Away presence, warning */
  --accent-red:       #E53935   /* DND, error, delete */
  --accent-blue:      #1D9BD1   /* Informational, links */

  --unread-badge:     #E53935   /* Red badge cho unread count */
  --mention-badge:    #E53935   /* Red badge cho @mention */

  --border-subtle:    #3D3F44   /* Dividers, input borders */
  --border-focus:     #5CAAFF   /* Focus ring */

LIGHT THEME (secondary):
  --bg-base:          #FFFFFF
  --bg-sidebar:       #4A154B   /* Slack purple sidebar */
  --bg-main:          #F8F8F8
  --bg-elevated:      #FFFFFF
  --bg-hover:         #F0F0F0
  --bg-active:        #E8E8E8
  --text-primary:     #1D1C1D
  --text-secondary:   #616061
  --text-on-sidebar:  #CFC3CF   /* Text trên purple sidebar */
  --text-on-sidebar-active: #FFFFFF
  --accent-purple:    #4A154B
```

### 1.2 Typography

```
Font Stack: 'Lato', 'Inter', -apple-system, BlinkMacSystemFont, sans-serif
Load từ Google Fonts: Lato (400, 700, 900)

Scale:
  --text-xs:    11px / line-height 1.4  /* Timestamps, badges */
  --text-sm:    13px / line-height 1.4  /* Secondary labels, metadata */
  --text-base:  15px / line-height 1.5  /* Message content, body */
  --text-md:    16px / line-height 1.4  /* Sidebar items, buttons */
  --text-lg:    18px / line-height 1.3  /* Section headers, channel name */
  --text-xl:    22px / line-height 1.2  /* Modal titles, page titles */
  --text-2xl:   28px / line-height 1.1  /* Auth page headings */

Weight:
  Regular: 400
  Medium:  600 (dùng cho labels, channel names)
  Bold:    700 (dùng cho unread, headings)
  Black:   900 (dùng cho workspace name)
```

### 1.3 Spacing System

```
Base unit: 4px
  --space-1: 4px
  --space-2: 8px
  --space-3: 12px
  --space-4: 16px
  --space-5: 20px
  --space-6: 24px
  --space-8: 32px
  --space-10: 40px
  --space-12: 48px
```

### 1.4 Border Radius

```
  --radius-sm:   4px   /* Badges, tooltips, small elements */
  --radius-md:   8px   /* Buttons, inputs, cards */
  --radius-lg:   12px  /* Modals, panels */
  --radius-xl:   16px  /* Large modals */
  --radius-full: 9999px /* Pills, avatars, badges */
```

### 1.5 Shadows

```
  --shadow-sm:  0 1px 3px rgba(0,0,0,0.3)
  --shadow-md:  0 4px 12px rgba(0,0,0,0.4)
  --shadow-lg:  0 8px 24px rgba(0,0,0,0.5)
  --shadow-glow-purple: 0 0 0 2px rgba(124,92,191,0.5)  /* Focus ring */
```

### 1.6 Breakpoints

```
  sm: 640px   (mobile landscape)
  md: 768px   (tablet)
  lg: 1024px  (desktop small)
  xl: 1280px  (desktop)
  2xl: 1536px (desktop large)
```

---

## 2. Màn Hình Đăng Nhập (Login)

### Layout

```
┌─────────────────────────────────────────────────────┐
│  FULL SCREEN: bg gradient linear(135deg,            │
│  #1A1D21 0%, #2D1B69 50%, #1A1D21 100%)             │
│                                                     │
│              ┌──────────────────────┐               │
│              │   CARD (480px wide)  │               │
│              │   bg: #2B2D31        │               │
│              │   radius: 16px       │               │
│              │   padding: 40px      │               │
│              │   shadow-lg          │               │
│              │                      │               │
│              │  [Logo 48px + Name]  │               │
│              │  "Đăng nhập vào"     │               │
│              │  "Workspace của bạn" │               │
│              │                      │               │
│              │  [Email Input]       │               │
│              │  [Password Input]    │               │
│              │  [Forgot password?]  │               │
│              │                      │               │
│              │  [BTN: Đăng nhập]   │               │
│              │                      │               │
│              │  ── hoặc ──          │               │
│              │                      │               │
│              │  [BTN: Google OAuth] │               │
│              │                      │               │
│              │  Chưa có tài khoản?  │               │
│              │  [Đăng ký ngay]      │               │
│              └──────────────────────┘               │
└─────────────────────────────────────────────────────┘
```

### Chi tiết Components

**Logo**: SVG icon (lightning bolt hoặc speech bubble) 48x48px + text "Nexus" (brand name) font-size 24px font-weight 900 color #FFFFFF. Đặt ở trên cùng center.

**Heading**: 
- Line 1: "Đăng nhập vào" — font-size 22px, color #D1D2D3, font-weight 400
- Line 2: "Workspace của bạn" — font-size 28px, color #FFFFFF, font-weight 700
- Margin-top: 24px từ logo

**Email Input**:
- Label: "Email" — 13px, #ABABAD, margin-bottom 6px
- Input: full-width, height 44px, bg #1A1D21, border 1.5px solid #3D3F44, radius 8px, padding 0 14px, color #D1D2D3, font-size 15px
- Placeholder: "ban@congty.com" — color #7B7D82
- Focus: border-color #7C5CBF, box-shadow 0 0 0 2px rgba(124,92,191,0.3)
- Error state: border-color #E53935, show error text 12px red bên dưới

**Password Input**: Tương tự email + icon eye bên phải (toggle show/hide, 20px, color #7B7D82)

**Forgot password link**: "Quên mật khẩu?" — text-align right, font-size 13px, color #5CAAFF, margin-top 8px

**Primary Button ("Đăng nhập")**:
- Full width, height 44px, bg #7C5CBF, radius 8px
- Text: "Đăng nhập" — 16px, font-weight 700, color white
- Hover: bg #8B6BD0, transform translateY(-1px)
- Active: transform translateY(0)
- Loading state: spinner icon (24px, white) thay text, disabled
- Margin-top: 24px

**Divider**: "── hoặc ──" — flex row với 2 đường kẻ (bg #3D3F44, height 1px) và text 13px #7B7D82 ở giữa

**Google OAuth Button**: Full width, height 44px, bg transparent, border 1.5px solid #3D3F44, radius 8px. Google "G" logo 20px bên trái + "Tiếp tục với Google" 15px color #D1D2D3

**Footer link**: "Chưa có tài khoản? " (muted) + "Đăng ký ngay" (link color #5CAAFF, hover underline)

**Animations**:
- Card: fade-in + slide-up khi mount (translateY 20px → 0, opacity 0 → 1, duration 400ms ease-out)
- Background: animated gradient (keyframes shift background-position, duration 8s infinite)
- Floating particles nhỏ trong background (optional: 20 dots animated với CSS)

---

## 3. Màn Hình Đăng Ký (Register)

### Layout
Tương tự Login card nhưng có thêm fields:
- Username input (bên dưới email)
- Confirm password (bên dưới password)
- Heading: "Tạo tài khoản mới"

**Username input**: Prefix "@" trong input (color #7B7D82), placeholder "ten_nguoi_dung"

**Validation inline**: Dưới mỗi field hiện validation hint khi user đang nhập:
- Username: "3-20 ký tự, chỉ chữ cái, số và dấu gạch dưới" — 12px, #7B7D82
- Password: progress bar strength (4 segments: weak=red, fair=yellow, good=blue, strong=green)

---

## 4. Màn Hình Chọn Workspace

### Layout

```
┌─────────────────────────────────────────────────────┐
│  Topbar (full width, height 52px, bg #19171D)        │
│  [Logo] Nexus              [User avatar] ▼           │
├─────────────────────────────────────────────────────┤
│                                                     │
│        "Chào mừng trở lại, [Name] 👋"              │
│        "Chọn workspace để tiếp tục"                 │
│                                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │ WS Card  │ │ WS Card  │ │ + Tạo mới│            │
│  │ [Icon]   │ │ [Icon]   │ │ [+]      │            │
│  │ Name     │ │ Name     │ │ "Tạo     │            │
│  │ 12 members│ │ 8 members│ │ workspace│            │
│  │          │ │ ▸ Active │ │ mới"     │            │
│  └──────────┘ └──────────┘ └──────────┘            │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Workspace Card**: 180x160px, bg #2B2D31, radius 12px, padding 20px, border 1.5px solid transparent
- Hover: border-color #3D3F44, bg #35373B, transform translateY(-2px)
- Active workspace: border-color #7C5CBF, badge "Đang hoạt động" màu green
- Icon: 56px, radius 12px, bg gradient purple nếu không có ảnh, letter avatar
- Name: 16px, 600, color #FFFFFF, margin-top 12px, ellipsis nếu quá dài
- Members: 13px, #ABABAD

**Tạo workspace card**: Dashed border #3D3F44, center dọc "+ icon 32px #7B7D82" + text "Tạo workspace mới"

---

## 5. Main App Layout (Workspace View)

### Layout Tổng Thể

```
┌────────┬──────────────────────────────────────────────────────────┐
│SIDEBAR │                    MAIN AREA                              │
│260px   │                                                          │
│fixed   │  ┌────────────────────────────────────────────────────┐  │
│        │  │ CHANNEL HEADER (56px height)                        │  │
│        │  └────────────────────────────────────────────────────┘  │
│        │  ┌────────────────────────────────────────────────────┐  │
│        │  │                                                    │  │
│        │  │  MESSAGE LIST (flex-1, scrollable)                 │  │
│        │  │                                                    │  │
│        │  └────────────────────────────────────────────────────┘  │
│        │  ┌────────────────────────────────────────────────────┐  │
│        │  │ MESSAGE INPUT (auto-height, min 44px, max 300px)    │  │
│        │  └────────────────────────────────────────────────────┘  │
└────────┴──────────────────────────────────────────────────────────┘
```

Khi Thread panel mở:
```
┌────────┬─────────────────────────────────┬───────────────────────┐
│SIDEBAR │        MAIN AREA                │   THREAD PANEL        │
│260px   │        (flex-1)                 │   380px fixed         │
└────────┴─────────────────────────────────┴───────────────────────┘
```

---

## 6. Sidebar

### Layout Chi Tiết

```
┌──────────────────────────────┐
│ WORKSPACE HEADER (52px)      │
│ ┌──────────────────────────┐ │
│ │ [WS Icon 24px] Name    ▼ │ │ ← dropdown trigger
│ └──────────────────────────┘ │
├──────────────────────────────┤
│ QUICK ACTIONS (36px each)    │
│ 🏠 Home                      │
│ 💬 DMs                        │
│ 🔔 Activity (badge unread)   │
│ 🔖 Saved                      │
├──────────────────────────────┤
│ [+] Add teammates            │ ← accent link color
├──────────────────────────────┤
│ SECTION: Channels          ▾ │ ← collapsible
│   # general            (3)   │ ← unread badge
│   # design                   │
│   # engineering              │ ← active: highlight bg
│   # random          @2       │ ← mention badge
│   [+] Add channel            │
├──────────────────────────────┤
│ SECTION: Direct Messages   ▾ │
│   ● Alice Nguyen             │ ← presence dot
│   ◌ Bob Tran           (1)   │ ← offline dot
│   ● Me (you)                 │
│   [+] Add DM                 │
├──────────────────────────────┤
│              FLEX SPACER     │
├──────────────────────────────┤
│ USER FOOTER (52px)           │
│ [Avatar][Name][status]  ⚙️   │
└──────────────────────────────┘
```

### Chi Tiết Visual

**Sidebar background**: `#19171D` — tối hơn main area 1 shade

**Workspace Header**:
- Height: 52px, padding 0 16px
- WS Icon: 24x24px, radius 6px
- Workspace name: 16px, font-weight 700, color #FFFFFF, max-width 160px, ellipsis
- Dropdown chevron: ▼ 14px, color #D1D2D3
- Hover: bg rgba(255,255,255,0.1) trên toàn header
- Bottom border: 1px solid rgba(255,255,255,0.1)

**Quick Actions**:
- Height: 32px per item, padding 0 16px
- Icon: 20px emoji/lucide icon, color #ABABAD
- Label: 15px, color #ABABAD, margin-left 8px
- Hover: bg rgba(255,255,255,0.1), color #FFFFFF
- Activity badge: đỏ số tròn 16px, font-size 11px, absolute top-right của icon

**Section Header** ("Channels", "Direct Messages"):
- Height: 28px, padding 0 12px
- Text: 13px, font-weight 700, color #ABABAD, uppercase letter-spacing 0.5px
- Collapse chevron: bên phải, rotate 0° (open) hoặc -90° (closed), transition 200ms
- Hover: color #FFFFFF, chevron visible

**Channel Item** (trong Channels section):
- Height: 28px, padding 0 8px 0 16px, border-radius 4px, margin 0 8px
- Hash icon (#): 16px, color #ABABAD, margin-right 6px
- Channel name: 15px, color #ABABAD
- Unread state: name color #FFFFFF, font-weight 700
- Active state: bg #3D3F44, name color #FFFFFF
- Hover (inactive): bg rgba(255,255,255,0.1)
- Unread badge (số): bên phải, bg #E53935, radius 9999px, min-width 18px, height 18px, font-size 11px, font-weight 700, padding 0 4px
- Mention badge (@N): bên phải, bg #E53935 với "@" prefix, tương tự trên
- Hover overflow: icon "..." xuất hiện bên phải thay badge (nếu có badge thì badge vẫn show)

**DM Item** (trong Direct Messages section):
- Height: 28px, tương tự channel item
- Avatar: 20x20px tròn, margin-right 8px
- Presence dot: 8x8px tròn, positioned absolute bottom-right của avatar
  - ONLINE: #3DB077 (green)
  - AWAY: #E8B430 (yellow) 
  - DND: #E53935 (red) với icon "−" trong dot
  - OFFLINE: #4A4C51 (grey) hoặc không có dot

**User Footer**:
- Height: 52px, padding 0 12px, border-top 1px solid rgba(255,255,255,0.1)
- Avatar: 36px tròn, với presence dot 10px
- Name: 15px, font-weight 600, color #D1D2D3, max-width 120px, ellipsis
- Status: 13px, color #ABABAD, margin-top 2px, ellipsis (":coffee: Be right back")
- Settings cog icon: 20px, color #ABABAD, hover color #FFFFFF, absolute right 12px

**Workspace Dropdown** (khi click header):
- Popover below header, width = sidebar width - 16px
- bg #2B2D31, border 1px solid #3D3F44, radius 8px, shadow-md
- Items: 36px height, padding 0 12px, radius 4px
  - "Workspace settings" (admin only)
  - "Invite people"
  - "Create channel"
  - divider
  - "Switch workspace" → list workspaces
  - divider
  - "Sign out" (color #E53935)

---

## 7. Channel Header

### Layout

```
┌────────────────────────────────────────────────────────────────┐
│ [#] general    [topic text truncated...]    [👤12] [🔍] [📌] [⋮]│
│   56px height, padding 0 20px, border-bottom 1px #3D3F44       │
└────────────────────────────────────────────────────────────────┘
```

### Chi Tiết

**Left side** (flex-1):
- Hash icon "#" — 20px, color #ABABAD, margin-right 6px
- Channel name — 16px, font-weight 700, color #FFFFFF
- Divider "|" — 1px solid #3D3F44, height 16px, margin 0 10px
- Topic text — 15px, color #ABABAD, ellipsis sau 200px, click → edit topic

**Right side** (flex, gap 4px):
- Member count button: icon 👤 (18px) + "12" (14px) — padding 4px 8px, radius 6px, hover bg #35373B
- Search icon: 🔍 20px — 28x28px button, radius 6px, hover bg #35373B
- Pins icon: 📌 20px — tương tự
- More options "⋮": tương tự, click → dropdown (Channel info, Notification prefs, Leave channel)

**DM Header** (thay channel name bằng):
- Avatar 24px + Presence dot + DM partner name + "(Direct message)" label nhỏ

---

## 8. Message List

### Layout & Grouping

```
┌─────────────────────────────────────────┐
│                                         │
│  ────────── 12 tháng 10, 2026 ──────── │ ← date divider
│                                         │
│  [Avatar] Alice Nguyen  10:30 SA        │ ← message group start
│           Chào mọi người! 🎉            │
│           Hôm nay chúng ta bắt đầu...  │ ← same author, <5min
│                                         │
│  👍 3  😂 2  [+ Add Reaction]           │ ← reaction bar
│  💬 2 replies  →                        │ ← thread preview
│                                         │
│           10:32 SA                      │ ← timestamp only (grouped)
│           Ai có câu hỏi không?          │
│                                         │
│  ────────────── Hôm nay ─────────────  │
│                                         │
│  [Avatar] Bob Tran  2:15 CH             │
│           @Alice xem file này nhé      │ ← mention highlighted
│           [📄 report.pdf  128 KB  ↓]   │ ← file attachment
│                                         │
│  [Avatar] Me (You)  2:20 CH            │
│           ```                           │ ← code block
│           const x = 1                  │
│           ```                           │
│                                         │
└─────────────────────────────────────────┘
```

### Date Divider

```
─── 12 tháng 10, 2026 ───
```
- Flex row: line (flex-1, height 1px, bg #3D3F44) + text 13px #7B7D82 + line
- Margin: 20px 0
- Text padding: 0 12px

### MessageItem

**Ungrouped (đầu nhóm)**:
- Padding: 8px 20px 2px 20px
- Avatar: 36x36px tròn, float left, margin-right 8px
- Author name: 15px, font-weight 700, color #FFFFFF, hover underline, cursor pointer
- Timestamp: 13px, color #7B7D82, margin-left 8px
- Content: 15px, color #D1D2D3, line-height 1.5

**Grouped (cùng author trong 5 phút)**:
- Padding: 2px 20px 2px 20px
- Bên trái (chỗ avatar): 44px wide → hover hiện timestamp nhỏ 11px #7B7D82 (right-aligned trong vùng 44px)
- Không hiện avatar, không hiện tên

**Hover state (toàn MessageItem)**:
- bg: rgba(255,255,255,0.03)
- Hiện MessageActions toolbar (absolute, top -18px, right 20px)

**Deleted message**:
- Content: "Tin nhắn này đã bị xóa" — italic, color #7B7D82
- Không hiện reactions, actions, attachments

**Edited message**:
- Sau content: "(đã sửa)" — 12px, color #7B7D82, italic

### MessageActions Toolbar (hover)

```
┌─────────────────────────────────────────────────┐
│  [😊] [↩ Reply] [✏️ Edit] [🔖] [📌] [⋮ More]   │
└─────────────────────────────────────────────────┘
```
- Absolute positioned: top -18px, right 20px
- bg #2B2D31, border 1px solid #3D3F44, radius 8px, shadow-md, padding 4px
- Buttons: 28x28px, radius 6px, icon 16px color #ABABAD
- Hover button: bg #3D3F44, color #FFFFFF
- "Edit" và "Delete" chỉ show cho author (hoặc admin cho delete)
- Tooltip bên dưới mỗi icon khi hover (8px offset)

### Reaction Bar

```
[👍 3] [😂 2] [❤️ 1] [+ Add]
```
- Display: flex, flex-wrap, gap 4px, margin-top 4px
- Reaction pill: height 22px, padding 0 6px, bg rgba(255,255,255,0.08), border 1px solid transparent, radius 9999px
  - Reacted by current user: bg rgba(124,92,191,0.3), border-color rgba(124,92,191,0.6)
  - Emoji: 14px, margin-right 4px
  - Count: 13px, color #D1D2D3
  - Hover: bg rgba(255,255,255,0.15)
- Add reaction button: "+" 14px + emoji icon 16px, color #7B7D82, hover color #D1D2D3

### Thread Preview (bên dưới message)

```
[avatar1][avatar2]  2 replies   Last reply 2 giờ trước   →
```
- Flex row, margin-top 4px, padding 4px 8px, radius 6px, cursor pointer
- Avatars: 18px tròn, overlap -4px, z-index stack
- "X replies": 13px, font-weight 600, color #5CAAFF
- "Last reply ...": 13px, color #ABABAD
- Arrow "→": 13px, hidden, show khi hover
- Hover toàn preview: bg rgba(255,255,255,0.05)

### Attachments

**Image attachment**:
- Max-width 360px, max-height 240px, border-radius 8px, object-fit cover
- Hover: overlay tối 20% + download icon center
- Click: mở lightbox modal (full screen image viewer)

**File attachment**:
```
┌───────────────────────────────────┐
│  📄  report.pdf                   │
│      128 KB · PDF Document    [↓] │
└───────────────────────────────────┘
```
- bg rgba(255,255,255,0.05), border 1px solid #3D3F44, radius 8px, padding 10px 14px
- Icon: 32px file type icon (pdf=red, zip=yellow, doc=blue, etc.)
- Name: 14px, font-weight 600, color #D1D2D3
- Meta: 13px, color #ABABAD
- Download button: 28x28px, icon ↓, hover bg #35373B

### Message Content — Markdown Rendering

```
**bold**        → <strong> font-weight 700, color #FFFFFF
_italic_        → <em> italic
`code`          → inline code: bg #1A1D21, border 1px solid #3D3F44, 
                  radius 4px, padding 1px 5px, font-family monospace,
                  color #E6C07B (amber/yellow code color), font-size 13px
```code block``` → pre: bg #1A1D21, border 1px solid #3D3F44, radius 8px,
                  padding 12px 16px, overflow-x auto, font-size 13px,
                  color #ABB2BF (light grey code), line-height 1.6
> blockquote    → border-left 4px solid #7C5CBF, padding-left 12px, 
                  color #ABABAD, font-style italic
~strikethrough~ → text-decoration line-through, color #7B7D82
@username       → bg rgba(92,170,255,0.2), color #5CAAFF, radius 4px, 
                  padding 1px 4px, font-weight 600, cursor pointer
#channel        → color #5CAAFF, font-weight 600, cursor pointer
:emoji:         → rendered as unicode emoji, font-size 18px
URLs            → color #5CAAFF, underline on hover
```

---

## 9. Message Input

### Layout

```
┌────────────────────────────────────────────────────────┐
│ FORMATTING TOOLBAR (28px height, padding 4px 12px)     │
│ [B] [I] [S] [Code] [|] [Link] [•] [1.] [>] [Emoji]   │
├────────────────────────────────────────────────────────┤
│ CONTENT AREA (min 44px, max 300px)                     │
│  Type a message to #general...                         │
│                                                        │
│  [📎 Attach] ──────────────────── [Send ▶]            │
└────────────────────────────────────────────────────────┘
```

**Outer container**: bg #2B2D31, border 1.5px solid #3D3F44, radius 12px, margin 0 20px 20px 20px
Focus-within: border-color #5CAAFF, box-shadow 0 0 0 1px rgba(92,170,255,0.2)

**Formatting Toolbar**:
- Height: 32px, padding 0 8px, border-bottom 1px solid #3D3F44
- Buttons: 24x24px, radius 4px, icon 14px, color #7B7D82
- Hover: bg #35373B, color #D1D2D3
- Active (toggled): bg rgba(124,92,191,0.3), color #7C5CBF
- Divider between groups: 1px solid #3D3F44, height 16px, margin 0 4px

**Text Area** (contenteditable div):
- Padding: 10px 14px
- Font: 15px, color #D1D2D3, line-height 1.5
- Placeholder: color #7B7D82

**Bottom bar** (height 32px, padding 0 8px):
- Left: Attach icon 📎 — 24x24px button, tooltip "Đính kèm file"
- Right: Send button — 28x28px, bg #7C5CBF, radius 6px, icon ▶ 14px white
  - Disabled (empty message): bg #3D3F44, icon color #7B7D82
  - Hover enabled: bg #8B6BD0
  - Active: transform scale(0.95)

**Attachment Preview** (hiển thị giữa content area và bottom bar):
```
┌─────────────────────────────────────────────┐
│  ┌────────────────┐  ┌─────────────────┐   │
│  │  [Image thumb] │  │ 📄 doc.pdf    X │   │
│  │  [X remove]    │  │ 128 KB          │   │
│  └────────────────┘  └─────────────────┘   │
└─────────────────────────────────────────────┘
```
- Flex row, padding 8px 14px, gap 8px
- Image: 60x60px, radius 6px, object-fit cover, X button overlay top-right
- File: bg #1A1D21, border 1px solid #3D3F44, radius 6px, padding 6px 10px, flex row

**@mention Autocomplete Dropdown**:
- Position: absolute, bottom 100% + 4px, left 0, width 280px
- bg #2B2D31, border 1px solid #3D3F44, radius 8px, shadow-lg
- Max-height 220px, overflow-y auto
- Header: "Members" — 11px, #7B7D82, padding 8px 12px 4px
- Item: 36px, flex row, padding 0 12px, gap 8px, radius 4px
  - Avatar 24px + name 14px #D1D2D3 + username 13px #ABABAD
  - Hover / keyboard-selected: bg #3D3F44

**:emoji: Autocomplete Dropdown**: Tương tự nhưng hiện emoji grid 5 cột, 40px per emoji cell

**Typing Indicator** (hiện phía trên input box, bên ngoài container):
```
● Alice đang nhập...
```
- Flex row, align-items center, padding 4px 20px
- Animated dots "..." (3 dots bounce animation, each 0.3s delay)
- Font: 13px italic, color #ABABAD

---

## 10. Thread Panel

### Layout

```
┌──────────────────────────────────────┐
│ HEADER (52px)                        │
│ "Thread"          [#general]   [✕]  │
├──────────────────────────────────────┤
│                                      │
│ ROOT MESSAGE (full, no grouping)     │
│ [Avatar] Alice  10:30 SA            │
│          Nội dung tin gốc đầy đủ    │
│          [reactions][thread preview] │
│                                      │
├──── 2 REPLIES ───────────────────────┤
│                                      │
│ [Avatar] Bob  10:35 SA              │
│          Reply 1...                  │
│                                      │
│ [Avatar] Me  10:40 SA               │
│          Reply 2...                  │
│                                      │
├──────────────────────────────────────┤
│ MESSAGE INPUT                        │
│ "Trả lời..."                         │
└──────────────────────────────────────┘
```

**Panel container**: width 380px, bg #222529, border-left 1px solid #3D3F44
**Header**: height 52px, padding 0 16px, border-bottom 1px solid #3D3F44
- "Thread": 18px, font-weight 700, color #FFFFFF
- Channel tag: "#general" 13px, bg #3D3F44, radius 4px, padding 2px 8px, color #ABABAD
- Close ✕: 28x28px button, icon 16px, hover bg #35373B

**Replies divider**: "2 Replies" — 12px, color #7B7D82, flex row với lines (tương tự date divider nhưng nhỏ hơn)

**Nếu panel mở trên mobile**: Hiện dưới dạng Sheet (bottom drawer) 80vh

---

## 11. Emoji Picker

```
┌────────────────────────────────┐
│ 🔍 Tìm emoji...                │
├────────────────────────────────┤
│ 😀 😂 😍 🎉 👍 ❤️ 🔥 ...     │ ← Frequently used
├────────────────────────────────┤
│ SMILEYS & PEOPLE              │
│ 😀 😃 😄 😁 😆 😅 😂         │
│ 🤣 🥲 ☺️ 😊 😇 🙂 🙃         │
│ ...                           │
└────────────────────────────────┘
```
- Popover: width 320px, bg #2B2D31, border 1px solid #3D3F44, radius 12px, shadow-lg
- Search input: height 36px, bg #1A1D21, radius 8px, padding 0 12px, border 1px solid #3D3F44
- Category tabs: icon-only row, 8 categories, 28x28px each
- Emoji grid: 8 columns, 38x38px per cell, emoji 22px, hover bg #3D3F44 radius 6px
- Scroll area: max-height 280px

---

## 12. User Profile Popup

Trigger: click avatar hoặc username trong message list, DM list

```
┌────────────────────────────────────┐
│  BANNER (80px, bg gradient purple) │
├──[Avatar 72px, -24px overlap]──────┤
│  Alice Nguyen          [✉️ Message] │
│  @alice.nguyen                     │
│  ──────────────────────────────── │
│  🟢 Active  · "Đang code 💻"      │
│  ──────────────────────────────── │
│  Local time  2:30 CH (GMT+7)      │
│  Member since  Jan 2024            │
│                                    │
│  CHANNELS IN COMMON               │
│  # general  # design               │
└────────────────────────────────────┘
```
- Popover: width 300px, bg #2B2D31, border 1px solid #3D3F44, radius 12px, shadow-lg
- Banner: height 80px, bg linear gradient (random từ username hash)
- Avatar: 72px, border 3px solid #2B2D31, positioned absolute
- Name: 18px, font-weight 700, color #FFFFFF, margin-top 40px (sau avatar)
- Username: 14px, color #ABABAD
- Message button: top-right của card, bg #7C5CBF, radius 6px, padding 6px 12px

---

## 13. Search Modal

Trigger: Ctrl+K hoặc click search icon trong header

```
┌─────────────────────────────────────────────────────┐
│ BACKDROP (blur, dim)                                 │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │ 🔍 Tìm kiếm...                          [⌘K] │   │ ← input
│  ├──────────────────────────────────────────────┤   │
│  │ RECENT SEARCHES                              │   │
│  │  🕐 "design review"                          │   │
│  │  🕐 "@alice"                                 │   │
│  ├──────────────────────────────────────────────┤   │
│  │ [Messages] [Files] [Channels] [People]       │   │ ← tabs
│  ├──────────────────────────────────────────────┤   │
│  │ ┌──────────────────────────────────────────┐ │   │
│  │ │ RESULT ITEM                               │ │   │
│  │ │ [Avatar] Alice Nguyen  #general  2 giờ   │ │   │
│  │ │ ...tìm thấy trong **design** review...   │ │   │
│  │ └──────────────────────────────────────────┘ │   │
│  │ (more results...)                            │   │
│  │                                              │   │
│  │ FILTERS: Channel ▾  From ▾  Date range ▾    │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```
- Modal: width 640px, max-height 70vh, bg #2B2D31, border 1px solid #3D3F44, radius 12px, shadow-lg
- Input: height 52px, padding 0 16px, font-size 16px, no border, border-bottom 1px solid #3D3F44
- Search icon: 20px left-padded
- Shortcut hint [⌘K]: badge 12px bg #3D3F44, absolute right
- Tabs: 40px height, border-bottom 1px solid #3D3F44, tab item 14px, active: color #7C5CBF + bottom border 2px #7C5CBF
- Result item: padding 12px 16px, hover bg #35373B, radius 8px
  - Highlight keyword: bg rgba(92,170,255,0.3), color #5CAAFF
- Filters bar: padding 8px 16px, flex gap 8px, chip style (radius 9999px, border #3D3F44, padding 4px 10px)

---

## 14. Notification Panel

Trigger: click Activity icon hoặc bell icon trong sidebar

```
┌─────────────────────────────────────────────┐
│ PANEL HEADER                                 │
│ "Activity"       [Đánh dấu tất cả đã đọc]  │
├─────────────────────────────────────────────┤
│ TODAY                                        │
│ ┌─────────────────────────────────────────┐ │
│ │ [●][Avatar] Bob đã @mention bạn          │ │ ← unread dot
│ │             trong #general              │ │
│ │             "Chào @alice bạn xem..."    │ │
│ │             5 phút trước               │ │
│ └─────────────────────────────────────────┘ │
│ ┌─────────────────────────────────────────┐ │
│ │ [Avatar] Alice đã reply thread của bạn  │ │ ← read (no dot)
│ │          trong #design                  │ │
│ │          1 giờ trước                   │ │
│ └─────────────────────────────────────────┘ │
│ YESTERDAY                                    │
│ ...                                          │
└─────────────────────────────────────────────┘
```
- Panel: width 380px, hiện dưới dạng Sheet (slide từ phải), bg #222529
- Unread item: bg rgba(124,92,191,0.08), border-left 3px solid #7C5CBF
- Unread dot: 8px circle #7C5CBF, absolute left
- Message preview: 14px, color #ABABAD, max 2 lines, ellipsis

---

## 15. Saved Items Page

Route: `/saved`

```
┌─────────────────────────────────────────────────────────┐
│ HEADER                                                   │
│ 🔖 "Saved Items"                           [🔍 Filter]  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ ── #general ─────────────────────────────────────────  │
│  ┌────────────────────────────────────────────────┐    │
│  │ [Avatar] Alice Nguyen  Oct 12, 2026        [✕] │    │
│  │          Nội dung message đã saved...           │    │
│  │          [Jump to message →]                    │    │
│  └────────────────────────────────────────────────┘    │
│                                                         │
│ ── #design ───────────────────────────────────────────  │
│  ...                                                    │
└─────────────────────────────────────────────────────────┘
```
- Layout: centered, max-width 720px, padding 24px
- Grouped bằng channel name (divider style section header)
- Saved card: bg #2B2D31, border 1px solid #3D3F44, radius 8px, padding 16px
- Remove button ✕: top-right absolute, 24x24px, hover bg #35373B
- "Jump to message" link: 13px, color #5CAAFF, hover underline

---

## 16. Settings — Profile

```
┌─────────────────────────────────────────────────────────┐
│ LEFT NAV (200px)        │ RIGHT CONTENT (flex-1)         │
│ ─────────────────       │                               │
│ Tài khoản của bạn  ◀  │ PROFILE SETTINGS              │
│   Hồ sơ                │                               │
│   Thông báo            │ ┌─────────────────────────┐   │
│   Giao diện            │ │   [Avatar 80px]  [Đổi]  │   │
│                         │ └─────────────────────────┘   │
│ Workspace Settings      │                               │
│   Tổng quan            │ Display Name                  │
│   Members              │ [Input: Alice Nguyen         ] │
│   Channels             │                               │
│                         │ Username                      │
│                         │ [@][Input: alice.nguyen      ]│
│                         │                               │
│                         │ Status                        │
│                         │ [😀][Input: Đang code 💻    ]│
│                         │                               │
│                         │ [BTN: Lưu thay đổi]          │
└─────────────────────────────────────────────────────────┘
```
- Page bg #1A1D21
- Left nav: bg #19171D, fixed, padding 16px 0
- Nav item: 36px, padding 0 20px, 14px, color #ABABAD, hover bg rgba(255,255,255,0.1), active color #FFFFFF font-weight 600
- Section header in nav: 11px, uppercase, #7B7D82, padding 12px 20px 4px
- Content area: max-width 560px, padding 32px

**Avatar section**: circular 80px, border 3px solid #3D3F44, hover: overlay "Đổi ảnh" center
**"Lưu thay đổi" button**: bg #7C5CBF, width auto, padding 0 20px, height 40px

---

## 17. Settings — Notifications

```
NOTIFICATION PREFERENCES

[Toggle] Bật thông báo desktop
  └── Khi bật: thêm options bên dưới
      [Toggle] Âm thanh thông báo
      [Toggle] Hiện preview nội dung

CHANNEL NOTIFICATIONS

  # general          [All ▾]
  # design           [Mentions only ▾]
  # engineering      [Nothing ▾]
  [+ Add exception]

DO NOT DISTURB

  [Toggle] Tắt thông báo theo lịch
  From [21:00 ▾] to [08:00 ▾]
```

- Toggle switch: width 36px, height 20px, bg #3D3F44 (off) / #7C5CBF (on), circle 16px slide animation
- Section divider: 1px solid #3D3F44, padding 24px 0
- Dropdown select: bg #2B2D31, border 1px solid #3D3F44, radius 6px, 34px height

---

## 18. Settings — Workspace (Admin)

### Tab: General

```
Workspace Icon: [Icon 72px] [Upload new] [Remove]
Name: [Input]
Description: [Textarea 3 rows]
Slug: nexus.app/[slug-input]
[Lưu thay đổi]

DANGER ZONE (red border section)
[Delete Workspace] ← red button
```

### Tab: Members

```
[Invite member] button  top-right

SEARCH: [🔍 Tìm member...]

┌────────────────────────────────────────────────────────┐
│ [Avatar] Alice Nguyen  @alice  alice@co.com  [Owner ▾] │ [✕]
│ [Avatar] Bob Tran      @bob    bob@co.com   [Admin ▾]  │ [✕]
└────────────────────────────────────────────────────────┘
```
- Table-like list: row 52px, border-bottom 1px solid #3D3F44
- Role dropdown: chip style, bg #3D3F44, radius 4px, width 100px

### Tab: Channels

```
[+ Create Channel]  [🔍 Tìm channel...]

# general    PUBLIC   42 members  Active   [Archive]
# design     PRIVATE  8 members   Active   [Archive]
# old-chat   PUBLIC   5 members   Archived [Unarchive] [Delete]
```
- Archived channels: name color #7B7D82, italic, row bg rgba(255,255,255,0.03)
- Archive button: outline style, border #3D3F44, text #ABABAD, hover border #E53935, text #E53935

---

## 19. Create Channel Modal

```
┌─────────────────────────────────────────┐
│ Tạo Channel              [✕]            │
│                                         │
│ Tên Channel                             │
│ [# ][ tên-channel...                  ] │
│ Tên kênh phải là chữ thường, viết liền  │
│                                         │
│ Mô tả (tuỳ chọn)                        │
│ [Textarea...]                           │
│                                         │
│ Loại Channel                            │
│ ◉ 🌐 Công khai — Ai cũng có thể join   │
│ ○ 🔒 Riêng tư — Chỉ thành viên được mời│
│                                         │
│            [Huỷ]  [Tạo Channel →]       │
└─────────────────────────────────────────┘
```
- Modal: width 480px, bg #2B2D31, radius 16px, shadow-lg
- Header: 52px, border-bottom 1px solid #3D3F44, title 18px 700
- Body padding: 20px 24px
- Radio option: flex row, gap 12px, padding 12px, radius 8px, border 1.5px solid #3D3F44
  - Selected: border-color #7C5CBF, bg rgba(124,92,191,0.1)
- Confirm button: bg #7C5CBF → disabled (#3D3F44) nếu name rỗng

---

## 20. Invite Member Modal

```
┌─────────────────────────────────────────┐
│ Mời người vào workspace      [✕]        │
│                                         │
│ Email address                           │
│ [alice@company.com              ] [Mời] │
│                                         │
│ Hoặc chia sẻ invite link               │
│ [https://nexus.app/invite/abc123] [📋]  │
│                                         │
│ Thành viên mới sẽ được join với role:   │
│ [Member ▾]                              │
└─────────────────────────────────────────┘
```

---

## 21. Channel Browse Modal

```
┌─────────────────────────────────────────────────────┐
│ Duyệt Channels                         [✕]          │
│ [🔍 Tìm channel...                              ]    │
├─────────────────────────────────────────────────────┤
│ 3 channels (sorted by member count)                 │
│                                                     │
│ # general         Mô tả ngắn...   42 members [Join] │
│ # design          UI/UX team       8 members [Join] │
│ # engineering     Tech team        15 members       │
│                                    (đã join) [Open] │
└─────────────────────────────────────────────────────┘
```
- Modal: width 560px, height 60vh
- Row: 60px, border-bottom 1px solid #3D3F44, padding 0 20px
- Join button: outline, border #7C5CBF, color #7C5CBF → hover bg #7C5CBF color white
- Already joined → "Open" button: text-only, color #5CAAFF

---

## 22. Image Lightbox

Trigger: click image attachment

```
┌─────────────────────────────────────────────────────┐
│ OVERLAY (bg black 90%, backdrop-blur 4px)            │
│                                                      │
│  [✕ Đóng]                               [↓ Tải về]  │
│                                                      │
│           ┌──────────────────────────┐              │
│    [‹]    │     IMAGE (max 90vw,     │    [›]       │
│           │         80vh)            │              │
│           └──────────────────────────┘              │
│                                                      │
│    Alice Nguyen • #general • Oct 12, 2026            │
└─────────────────────────────────────────────────────┘
```
- Click outside image → close
- Keyboard: Escape=close, ←→=prev/next (nếu multiple images)
- Pinch-to-zoom support

---

## 23. Micro-animations & Transitions

```
GLOBAL TRANSITIONS:
  All interactive elements: transition 150ms ease

SIDEBAR:
  Channel item hover bg: 100ms
  Unread badge appear: scale 0 → 1, 200ms spring

MESSAGE:
  New message appear: translateY(8px) opacity(0) → normal, 250ms ease-out
  Reaction add: scale 1.3 → 1, 200ms spring
  Reaction remove: scale 0, opacity 0, 150ms
  Actions toolbar appear: opacity 0 → 1, translateY(-4px) → 0, 150ms
  Deleted message: content fade to grey, height collapse, 300ms

THREAD:
  Panel slide in: translateX(100%) → 0, 300ms ease-out
  Panel slide out: reverse, 250ms ease-in

MODAL:
  Backdrop: opacity 0 → 1, 200ms
  Modal content: scale(0.95) opacity(0) → normal, 250ms ease-out
  Close: reverse, 150ms

TYPING INDICATOR:
  Dots bounce: keyframe translateY 0 → -4px → 0, 800ms infinite
  Each dot offset: 0ms, 150ms, 300ms delay

PRESENCE DOT:
  Online → offline: color transition 500ms
  New online: pulse ring animation (scale 1 → 1.5, opacity 1 → 0, 1s infinite)

NOTIFICATION BADGE:
  First appear: scale 0 → 1.2 → 1, 300ms spring
  Count increment: scale 1 → 1.2 → 1, 200ms

SEND BUTTON:
  Active/enabled: bg transition 200ms
  Click: scale 0.9 → 1, 150ms

SKELETON LOADING:
  Shimmer: linear-gradient slide từ left sang right, 1.5s infinite
  Color: #2B2D31 → #35373B → #2B2D31
```

---

## 24. Mobile Layout (< 768px)

### Main Layout
```
Sidebar: ẩn hoàn toàn (display: none)
Hamburger button: top-left, 44x44px, bên cạnh channel name
Click hamburger: Sidebar hiện dưới dạng Drawer (left slide, 85vw, overlay backdrop)
```

### Message Input (Mobile)
```
Formatting toolbar: ẩn (chỉ hiện Attach + Emoji + Send)
Input: tăng padding cho touch targets (min-height 48px)
```

### Thread Panel (Mobile)
```
Hiện dưới dạng bottom sheet (80vh, có handle bar kéo)
Hoặc navigate đến full screen thread route
```

---

## 25. Loading & Empty States

### Channel Loading (Skeleton)
```
MessageList skeleton:
- 3-5 message skeletons, mỗi cái:
  - Circle 36px (avatar) + line 120px (name) + line 60px (timestamp)
  - Lines 100%, 80%, 60% (content)
- Shimmer animation

Sidebar skeleton:
- 5 channel items: rect 60% width shimmer
```

### Empty States

**Channel mới không có tin nhắn**:
```
[Channel icon 64px, gradient bg, center]
# general
"Đây là điểm khởi đầu của kênh #general"
"Kênh này dành cho... [mô tả nếu có]"
[🎉 Mời thành viên vào kênh này]
```

**DM mới**:
```
[Avatar 64px partner]
"Đây là kênh nhắn tin riêng với Alice Nguyen"
"@alice.nguyen • Active 2 giờ trước"
[Send a message button]
```

**No search results**:
```
[🔍 icon 48px, grey]
"Không tìm thấy kết quả cho "xyz""
"Thử tìm với từ khoá khác"
```

**No saved items**:
```
[🔖 icon 48px]
"Chưa có tin nhắn được lưu"
"Hover lên bất kỳ tin nhắn nào và click 🔖 để lưu"
```

---

## 26. Toast Notifications (Bottom-right corner)

```
┌────────────────────────────────┐
│ ✅ Message đã được gửi          │ [✕]
└────────────────────────────────┘

┌────────────────────────────────┐
│ ❌ Gửi thất bại. [Thử lại]     │ [✕]
└────────────────────────────────┘

┌────────────────────────────────┐
│ 🔔 @alice đề cập đến bạn       │
│    trong #general               │ [✕]
└────────────────────────────────┘
```
- Stack từ bottom-right, gap 8px
- Width: 320px
- bg: success=#1E3A2E, error=#3A1E1E, info=#1E2A3A — tất cả với left border 4px màu tương ứng
- Appear: slide từ right + fade in, 250ms
- Auto-dismiss: 5s (success), 8s (error), không auto-dismiss (mention)
- Progress bar: thin line at bottom, shrinks over duration

---

## 27. Presence Status Picker (Click status trong User Footer)

```
┌──────────────────────────────┐
│ 🟢 Active                ✓  │ ← hiện tại
│ 🌙 Away                      │
│ ⛔ Do not disturb             │
├──────────────────────────────┤
│ Set status message...        │
│ [😀][Clear status "Đang code"]│
└──────────────────────────────┘
```
- Popover: width 220px, above user footer, bg #2B2D31, border 1px solid #3D3F44, radius 8px, shadow-md
- Items: 36px, padding 0 12px, hover bg #35373B, radius 4px
- Status message input (bottom): 13px, placeholder "Bạn đang làm gì?", Enter để lưu

---

## 28. Workspace Switcher Dropdown (Click workspace icon)

```
┌──────────────────────────────────────┐
│ MY WORKSPACES                        │
│ ┌────────────────────────────────┐   │
│ │ [Icon] Công ty ABC     ✓ Active│   │
│ └────────────────────────────────┘   │
│   [Icon] Startup XYZ                 │
├──────────────────────────────────────┤
│ [+] Tạo workspace mới                │
│ [↗] Tìm workspace khác               │
└──────────────────────────────────────┘
```
- Dropdown below workspace icon
- Width: 280px
- Active workspace: bg rgba(124,92,191,0.15), border-left 3px solid #7C5CBF
- Item: 52px, padding 0 16px, flex row, gap 10px
  - Icon: 36px, radius 8px
  - Name: 15px font-weight 600, sub: "X members" 13px #ABABAD
  - Active checkmark ✓: color #7C5CBF

---

## 29. Component Library Summary (Shadcn UI Usage)

```
Button        → Login/register CTA, modal actions, send button
Input         → All form inputs, search bars
Textarea      → Message input fallback, description fields
Avatar        → User avatars throughout
Badge         → Unread counts, role labels (Member/Admin/Owner)
Dialog        → Create channel, invite, confirm delete
Sheet         → Thread panel (mobile), notification panel, member list
Tooltip       → Toolbar button hints, timestamp hover, emoji name
Popover       → User profile, emoji picker, status picker
Command       → Search modal (⌘K), @mention dropdown, emoji dropdown
ScrollArea    → Message list, notification list, sidebar
Separator     → Between sections
Skeleton      → Loading states
Tabs          → Search modal tabs, settings tabs
DropdownMenu  → Workspace header dropdown, message "more" actions
Toggle        → Formatting toolbar buttons, notification toggles
```

---

## 30. Accessibility Requirements

```
- Tất cả interactive elements có focus ring (#5CAAFF, 2px)
- Keyboard navigation: Tab order logic, Escape đóng modal/panel
- ARIA labels: aria-label cho icon-only buttons
- Screen reader: role="alert" cho toast, aria-live="polite" cho typing indicator
- Color contrast: minimum 4.5:1 cho text, 3:1 cho UI elements
- Image alt text: meaningful alt cho avatars và attachments
- Reduced motion: @media (prefers-reduced-motion) → tắt animations
```
