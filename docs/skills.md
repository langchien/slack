# Danh sách Agent Skills & Cú pháp cài đặt

Tài liệu quản lý các kỹ năng (Skills) của AI Agent trong dự án, được phân chia theo cấu trúc **Backend** và **Frontend (React SPA)**.

Quản lý cài đặt thông qua CLI [skills.sh](https://skills.sh/):
```powershell
npx skills add <source> [options]
```

---

## 1. Nhóm Backend

### A. NestJS Architecture & Best Practices
* **Nguồn:** `kadajett/agent-nestjs-skills` (GitHub: [kadajett/agent-nestjs-skills](https://github.com/kadajett/agent-nestjs-skills))
* **Skill:** `nestjs-best-practices` (~30K installs)
* **Mô tả:** Hướng dẫn kiến trúc chuẩn cho NestJS enterprise: phân tách Modular, Dependency Injection, Guards (kiểm soát vai trò RBAC Admin/Member/Guest), Pipes (Validation), Interceptors và Exception Filters.
* **Cú pháp cài đặt:**
  ```powershell
  npx skills add kadajett/agent-nestjs-skills --skill nestjs-best-practices -y
  ```

### B. Realtime Communication (Socket.io & WebSockets)
* **Nguồn:** `jeffallan/claude-skills` (GitHub: [jeffallan/claude-skills](https://github.com/jeffallan/claude-skills))
* **Skill:** `websocket-engineer` (>6.5K installs)
* **Mô tả:** Chuyên sâu về hệ thống giao tiếp thời gian thực 2 chiều bằng Socket.io / WebSockets trong NestJS Gateways. Xử lý quản lý Rooms theo Workspace/Channel, Presence tracking (Online/Away/Offline), Typing indicator, và mở rộng quy mô (Horizontal scaling) với Redis Adapter.
* **Cú pháp cài đặt:**
  ```powershell
  npx skills add jeffallan/claude-skills --skill websocket-engineer -y
  ```

### C. Prisma ORM (Database & Migrations)
* **Nguồn official:** `prisma/skills` (GitHub: [prisma/skills](https://github.com/prisma/skills))
* **Các skills:**
  * `prisma-cli`: Lệnh CLI (migrate, generate, db pull/push, studio).
  * `prisma-client-api`: Cú pháp truy vấn CRUD, filter, relations, transactions.
  * `prisma-orm-setup`: Thiết lập kết nối cơ sở dữ liệu.
  * `prisma-upgrade-v7`: Hướng dẫn nâng cấp và migration giữa các phiên bản.
* **Cú pháp cài đặt:**
  ```powershell
  npx skills add prisma/skills -y
  ```

---

## 2. Nhóm Frontend (React SPA)

### A. Routing & Server State (TanStack)
* **Nguồn official:** `tanstack-skills/tanstack-skills` (GitHub: [tanstack-skills/tanstack-skills](https://github.com/tanstack-skills/tanstack-skills))
* **Các skills:**
  * `tanstack-router`: Định tuyến 100% type-safe, search params, route loaders, nested layouts.
  * `tanstack-query`: Quản lý server state, async caching, mutations, optimistic updates.
  * `tanstack-table`: Headless table/datagrid cho quản lý dữ liệu bảng biểu.
* **Cú pháp cài đặt:**
  ```powershell
  npx skills add tanstack-skills/tanstack-skills --skill tanstack-query tanstack-router tanstack-table -y
  ```

### B. UI Components & Styling
* **shadcn/ui**
  * **Nguồn official:** `shadcn/ui` (GitHub: [shadcn-ui/ui](https://github.com/shadcn-ui/ui))
  * **Skill:** `shadcn`
  * **Mô tả:** Thư viện component mẫu, CLI add components, tích hợp radix-ui và Tailwind.
  * **Cú pháp:**
    ```powershell
    npx skills add shadcn/ui --skill shadcn -y
    ```

* **Tailwind CSS (v4)**
  * **Nguồn:** `lombiq/tailwind-agent-skills`
  * **Skill:** `tailwind-4-docs`
  * **Mô tả:** Tra cứu cú pháp, utility classes, cấu hình CSS-first theo Tailwind v4.
  * **Cú pháp:**
    ```powershell
    npx skills add lombiq/tailwind-agent-skills --skill tailwind-4-docs -y
    ```

### C. Client State Management (Zustand)
* **Nguồn:** `lobehub/lobehub`
* **Skill:** `zustand`
* **Mô tả:** Quản lý state toàn cục phía client (UI states, toggles, active modals/tabs). Hướng dẫn store slices, selector optimization tránh re-render thừa và persist middleware.
* **Cú pháp cài đặt:**
  ```powershell
  npx skills add lobehub/lobehub --skill zustand -y
  ```

### D. Form Handling & Validation
* **Nguồn:** `pproenca/dot-skills`
* **Skills:** `react-hook-form`, `zod`
* **Mô tả:** Xử lý form hiệu năng cao (uncontrolled input, re-render tối thiểu) tương thích trực tiếp với Form component của `shadcn/ui`, kết hợp xác thực schema bằng Zod.
* **Cú pháp cài đặt:**
  ```powershell
  npx skills add pproenca/dot-skills --skill react-hook-form zod -y
  ```

### E. Design Guidelines & UX Audit
* **Nguồn official:** `vercel-labs/agent-skills` (GitHub: [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills))
* **Skill:** `web-design-guidelines`
* **Mô tả:** Đánh giá tuân thủ chuẩn Web Interface Guidelines: kiểm tra tương phản màu sắc, spacing, typography, accessibility (a11y), responsive và micro-interactions.
* **Cú pháp cài đặt:**
  ```powershell
  npx skills add vercel-labs/agent-skills --skill web-design-guidelines -y
  ```

---

## 3. Lệnh khôi phục hoặc cài đặt nhanh toàn bộ dự án

Nếu cần khôi phục lại các skill trên một máy mới dựa trên file `skills-lock.json`:
```powershell
npx skills experimental_install
```

Hoặc cài đặt thủ công từng nhóm:

* **Cài đặt toàn bộ Backend:**
  ```powershell
  npx skills add kadajett/agent-nestjs-skills --skill nestjs-best-practices -y
  npx skills add jeffallan/claude-skills --skill websocket-engineer -y
  npx skills add prisma/skills -y
  ```

* **Cài đặt toàn bộ Frontend (SPA):**
  ```powershell
  npx skills add tanstack-skills/tanstack-skills --skill tanstack-query tanstack-router tanstack-table -y
  npx skills add shadcn/ui --skill shadcn -y
  npx skills add lombiq/tailwind-agent-skills --skill tailwind-4-docs -y
  npx skills add lobehub/lobehub --skill zustand -y
  npx skills add pproenca/dot-skills --skill react-hook-form zod -y
  npx skills add vercel-labs/agent-skills --skill web-design-guidelines -y
  ```
