# CẨM NANG TOÀN DIỆN VỀ TURBOREPO CHO FULLSTACK MONOREPO

> **Kiến trúc ứng dụng**: Client (React / Vite / tanstack router hoặc react-router) + Server (NestJS / Node.js) + Shared Package (`@app/shared`)  
> **Package Manager**: pnpm workspaces  
> **Nguồn tham chiếu chính thức**: [Turborepo Documentation (turbo.build)](https://turbo.build/repo/docs)

---

## MỤC LỤC

1. [Kỹ Năng Đọc & Tự Tra Cứu Tài Liệu Chính Thức Turborepo](#1-kỹ-năng-đọc--tự-tra-cứu-tài-liệu-chính-thức-turborepo)
2. [Các Khái Niệm Cốt Lõi (Core Concepts)](#2-các-khái-niệm-cốt-lõi-core-concepts)
3. [Quy Trình Cài Đặt Chi Tiết Từng Bước (Step-by-Step Setup)](#3-quy-trình-cài-đặt-chi-tiết-từng-bước-step-by-step-setup)
4. [Giải Thích Chi Tiết Cấu Hình `turbo.json`](#4-giải-thích-chi-tiết-cấu-hình-turbojson)
5. [Giải Quyết Bài Toán Package `shared` Giữa NestJS & React](#5-giải-quyết-bài-toán-package-shared-giữa-nestjs--react)
6. [Cẩm Nang Lệnh CLI & Cú Pháp Lọc (`--filter` Cheatsheet)](#6-cẩm-nang-lệnh-cli--cú-pháp-lọc---filter-cheatsheet)
7. [Xử Lý Lỗi Phổ Biến (Troubleshooting & Best Practices)](#7-xử-lý-lỗi-phổ-biến-troubleshooting--best-practices)

---

## 1. KỸ NĂNG ĐỌC & TỰ TRA CỨU TÀI LIỆU CHÍNH THỨC TURBOREPO

Khi công nghệ cập nhật, việc tự đọc hiểu tài liệu gốc trên **[turbo.build/repo/docs](https://turbo.build/repo/docs)** giúp bạn không bị phụ thuộc vào AI hay các hướng dẫn cũ. Dưới đây là "bản đồ định vị" cấu trúc trang tài liệu của Vercel:

```
turbo.build/repo/docs/
├── Getting Started       -> Khởi đầu: Triết lý Monorepo, Cài đặt nhanh
├── Core Concepts         -> BẮT BUỘC ĐỌC: Hiểu cách Turborepo hoạt động
│   ├── Monorepos         -> Workspace hoạt động thế nào
│   ├── Task Graph        -> Cách xác định thứ tự chạy task
│   └── Caching           -> Cách tính toán Hash và lưu kết quả
├── Crafting your turbo.json -> NƠI TRA CỨU CẤU HÌNH PIPELINE
│   ├── Tasks             -> Khai báo các script build, dev, test
│   ├── Task Dependencies -> Cú pháp `dependsOn`, dấu `^`
│   └── Environment Variables -> Quản lý biến môi trường trong cache
├── Reference             -> TỪ ĐIỂN TRA CỨU NHANH
│   ├── Configuration     -> Toàn bộ thuộc tính của turbo.json
│   └── Command Line (CLI)-> Toàn bộ cờ (flags) của lệnh `turbo run`
└── Guides                -> Hướng dẫn theo từng framework (React, NestJS, Docker, CI/CD)
```

### 3 Quy tắc vàng khi đọc Docs Turborepo:

1. **Lưu ý phiên bản (Turborepo v1 vs v2)**:
   - **v1**: Cấu hình pipeline nằm dưới từ khóa `"pipeline": { ... }`.
   - **v2 (Hiện tại)**: Cấu hình đổi thành `"tasks": { ... }`. Luôn tra cứu theo chuẩn `tasks`.
2. **Phân biệt Workspace Tool vs Build System**:
   - `pnpm` / `npm` / `yarn`: Phụ trách tải dependency, liên kết symlink giữa các thư mục (`workspace:*`).
   - `Turborepo`: Phụ trách điều phối thứ tự chạy script và lưu cache kết quả. Turborepo **không** thay thế `pnpm`.
3. **Khi gặp lỗi, tra cứu mục nào trước?**:
   - Lỗi cấu hình file JSON -> Vào mục **Reference > Configuration Reference**.
   - Lỗi lệnh chạy hoặc muốn lọc package -> Vào mục **Reference > CLI Reference**.
   - Lỗi cache bị miss liên tục -> Vào mục **Core Concepts > Caching**.

---

## 2. CÁC KHÁI NIỆM CỐT LÕI (CORE CONCEPTS)

### A. Task Graph (Đồ thị tác vụ)

Turborepo xem các script trong dự án của bạn như một đồ thị có hướng (Directed Acyclic Graph - DAG).

- Nếu `server` và `client` cần code từ `shared`, Turborepo vẽ ra luồng:
  $$\text{shared:build} \longrightarrow \begin{cases} \text{server:build} \\ \text{client:build} \end{cases}$$
- Hai task `server:build` và `client:build` sẽ được chạy song song ngay khi `shared:build` vừa hoàn tất.

### B. Topological Dependency (Dấu `^`)

Trong Turborepo, dấu `^` (caret) mang ý nghĩa **"Phụ thuộc ngược lên các package bên dưới"**:

- `"dependsOn": ["^build"]`: Bắt buộc tất cả các package mà tôi phụ thuộc (ví dụ `shared`) phải chạy lệnh `build` thành công trước khi tôi bắt đầu chạy `build`.

### C. Cơ chế Caching (Smart Hashing)

Trước khi chạy bất kỳ task nào, Turborepo sẽ tính toán một chuỗi băm (Hash) dựa trên:

1. Nội dung tất cả các file mã nguồn của package đó (theo khai báo `inputs`).
2. Mã băm của các package mà nó phụ thuộc vào (`^build`).
3. Các biến môi trường được khai báo liên quan.
4. Phiên bản các dependency trong `package.json`.

> **Kết quả**:
>
> - Nếu chuỗi Hash này **đã từng xuất hiện**, Turborepo khôi phục ngay toàn bộ thư mục `outputs` (ví dụ `dist/`) và in lại log trong ~**10ms - 50ms** (`>>> FULL TURBO`).
> - Không tốn CPU biên dịch lại những gì không đổi!

---

## 3. QUY TRÌNH CÀI ĐẶT CHI TIẾT TỪNG BƯỚC (STEP-BY-STEP SETUP)

Áp dụng cho cấu trúc:

```text
my-monorepo/
├── pnpm-workspace.yaml
├── package.json
├── turbo.json
├── client/          (React / Vite)
├── server/          (NestJS)
└── shared/          (TypeScript / Zod)
```

### Bước 1: Khởi tạo Workspace với pnpm

Tạo file `pnpm-workspace.yaml` tại thư mục gốc:

```yaml
packages:
  - "client"
  - "server"
  - "shared"
```

### Bước 2: Cài đặt Turborepo vào Root

Tại thư mục gốc, cài đặt `turbo` làm `devDependency`:

```bash
pnpm add -Dw turbo
```

### Bước 3: Tạo file cấu hình `turbo.json` ở Root

Tạo file `turbo.json` tại thư mục gốc:

```json
{
  "$schema": "https://turbo.build/schema.json",
  "ui": "tui",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["src/**", "tsconfig*.json", "package.json"],
      "outputs": ["dist/**", "build/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "typecheck": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    "lint": {
      "outputs": []
    },
    "clean": {
      "cache": false
    }
  }
}
```

### Bước 4: Chuẩn hóa Scripts trong từng Package con

Để Turborepo điều phối nhịp nhàng, các package con cần sử dụng tên script thống nhất:

#### 1. Package `shared/package.json`:

```json
{
  "name": "@app/shared",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "default": "./dist/index.js"
    }
  },
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch --preserveWatchOutput",
    "typecheck": "tsc --noEmit",
    "clean": "rimraf dist"
  }
}
```

#### 2. Package `server/package.json` (NestJS):

```json
{
  "name": "server",
  "private": true,
  "dependencies": {
    "@app/shared": "workspace:*"
  },
  "scripts": {
    "build": "nest build",
    "dev": "nest start --watch",
    "typecheck": "tsc --noEmit",
    "clean": "rimraf dist"
  }
}
```

#### 3. Package `client/package.json` (React / Vite):

```json
{
  "name": "client",
  "private": true,
  "dependencies": {
    "@app/shared": "workspace:*"
  },
  "scripts": {
    "build": "vite build",
    "dev": "vite dev --port 3000",
    "typecheck": "tsc --noEmit",
    "clean": "rimraf dist"
  }
}
```

### Bước 5: Cập nhật file `package.json` ở Root

Tại thư mục gốc, định nghĩa các lệnh chạy toàn dự án thông qua `turbo`:

```json
{
  "name": "quan-ly-quan-an",
  "private": true,
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "typecheck": "turbo run typecheck",
    "lint": "turbo run lint",
    "clean": "turbo run clean && rimraf node_modules"
  },
  "devDependencies": {
    "turbo": "^2.0.0"
  }
}
```

---

## 4. GIẢI THÍCH CHI TIẾT CẤU HÌNH `turbo.json`

File `turbo.json` là trái tim của hệ thống. Dưới đây là giải nghĩa từng trường theo tài liệu gốc của Vercel:

| Thuộc tính       | Kiểu dữ liệu          | Ý nghĩa & Cách dùng                                                                                                                                      |
| :--------------- | :-------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`$schema`**    | `string`              | Đường dẫn JSON schema giúp VS Code tự động gợi ý (autocomplete) và kiểm tra lỗi cú pháp.                                                                 |
| **`ui`**         | `"tui"` \| `"stream"` | `"tui"` (Terminal UI) hiển thị giao diện phân luồng tương tác đẹp mắt (mới trong Turborepo 2).                                                           |
| **`tasks`**      | `object`              | Danh sách các tác vụ trùng tên với script trong các file `package.json`.                                                                                 |
| **`dependsOn`**  | `string[]`            | Xác định các task cần hoàn thành trước. Cú pháp `^<task>` = chờ các package phụ thuộc làm xong trước. Cú pháp `<task>` = chờ task đó trong cùng package. |
| **`inputs`**     | `string[]`            | Danh sách glob file ảnh hưởng đến kết quả task. Nếu các file này không đổi, Turborepo coi như cache còn hạn.                                             |
| **`outputs`**    | `string[]`            | Thư mục/file mà task tạo ra (như `dist/**`). Turborepo sẽ nén thư mục này lưu vào cache để tái sử dụng.                                                  |
| **`cache`**      | `boolean`             | `true` (mặc định) = lưu cache. `false` = luôn chạy lại mỗi lần gọi (bắt buộc đặt cho lệnh dev).                                                          |
| **`persistent`** | `boolean`             | Đặt là `true` cho các lệnh chạy liên tục (watch mode, dev server) để Turborepo không xem nó là tác vụ bị treo (hanging task).                            |
| **`env`**        | `string[]`            | Danh sách biến môi trường (ví dụ `["NODE_ENV", "API_URL"]`) có ảnh hưởng đến output của bản build.                                                       |

---

## 5. GIẢI QUYẾT BÀI TOÁN PACKAGE `shared` GIỮA NESTJS & REACT

Đây là vấn đề kinh điển nhất khiến nhiều dự án gặp lỗi:

- **Vite (Client)**: Bundler hiện đại, có thể đọc trực tiếp file TypeScript `.ts` từ mã nguồn.
- **NestJS (Server)**: Chạy trên Node.js runtime, cần file JavaScript `.js` và type definitions `.d.ts` đã được compile trong thư mục `dist/`.

### 2 Phương án xử lý chuẩn mực:

#### Phương án 1: Pre-compiled Package (Khuyên dùng - Chuẩn doanh nghiệp)

- `shared` được biên dịch ra `dist/` bằng `tsc` hoặc `tsup`.
- Trong `turbo.json`:
  ```json
  "build": {
    "dependsOn": ["^build"]
  }
  ```
- **Khi chạy `pnpm build`**: Turborepo tự động biên dịch `shared` xong xuôi trước, sau đó NestJS đọc trực tiếp từ `shared/dist/index.js` mà không bao giờ gặp lỗi thiếu file.
- **Khi chạy `pnpm dev`**: Cả 3 tiến trình cùng chạy:
  1. `shared`: `tsc --watch` (liên tục biên dịch mỗi khi bạn sửa code).
  2. `server`: `nest start --watch` (tự khởi động lại khi thấy file trong `shared/dist` thay đổi).
  3. `client`: `vite dev` (tự Hot Module Reload).

#### Phương án 2: Bundler Path Mapping (Dành riêng cho Client)

Trong [client/tsconfig.json](../client/tsconfig.json):

```json
{
  "compilerOptions": {
    "paths": {
      "@app/shared": ["../shared/src/index.ts"]
    }
  }
}
```

Nhờ cấu hình này, khi code Client, bạn nhận được tính năng **Instant Jump-to-definition** (F12 nhảy thẳng vào mã nguồn gốc của `shared/src` thay vì nhảy vào file `.d.ts` trong `dist`).

---

## 6. CẨM NANG LỆNH CLI & CÚ PHÁP LỌC (`--filter` CHEATSHEET)

Cú pháp `--filter` là sức mạnh lớn nhất khi làm việc hàng ngày với Turborepo:

### 1. Chỉ định một package cụ thể:

```bash
# Chỉ chạy dev server NestJS
pnpm turbo run dev --filter=server

# Chỉ chạy dev client
pnpm turbo run dev --filter=client

# Chỉ build package shared
pnpm turbo run build --filter=@app/shared
```

### 2. Chỉ định package và toàn bộ dependency phụ thuộc (`...` phía sau):

```bash
# Build server và tất cả các package mà server phụ thuộc (tự động build cả @app/shared)
pnpm turbo run build --filter=server...
```

### 3. Chỉ định package và các app bị ảnh hưởng bởi nó (`...` phía trước):

```bash
# Chạy test/build cho tất cả những app/package nào đang sử dụng @app/shared
pnpm turbo run build --filter=...@app/shared
```

### 4. Lọc theo thay đổi Git (Cực kỳ giá trị cho CI/CD):

```bash
# Chỉ build/test những package có code thay đổi so với nhánh main
pnpm turbo run build --filter=[main...HEAD]

# Chỉ test những package thay đổi trong commit gần nhất
pnpm turbo run typecheck --filter=[HEAD~1]
```

---

## 7. XỬ LÝ LỖI PHỔ BIẾN (TROUBLESHOOTING & BEST PRACTICES)

### 1. Lỗi: `Cache miss` liên tục dù mã nguồn không thay đổi

- **Nguyên nhân**: Bạn chưa khai báo đúng mục `inputs` trong `turbo.json`, hoặc trong mã nguồn có file tạm/log sinh ra tự động làm thay đổi mã băm.
- **Khắc phục**:
  - Khai báo rõ ràng: `"inputs": ["src/**", "tsconfig*.json", "package.json"]`.
  - Đảm bảo các file sinh tự động (`dist/`, `.turbo/`, `node_modules/`) đã được thêm vào `.gitignore`.

### 2. Lỗi: Tiến trình `dev` bị chặn, các app khác không chạy được

- **Nguyên nhân**: Thiếu thuộc tính `"persistent": true` trong task `dev`.
- **Khắc phục**: Trong `turbo.json`:
  ```json
  "dev": {
    "cache": false,
    "persistent": true
  }
  ```

### 3. Lỗi: NestJS báo `Cannot find module '@app/shared'`

- **Nguyên nhân**: Thư mục `shared/dist` chưa được tạo lần đầu.
- **Khắc phục**:
  Chạy lệnh build thủ công cho shared một lần:
  ```bash
  pnpm turbo run build --filter=@app/shared
  ```
  Sau đó khởi động lại lệnh dev.

### 4. Xóa sạch Cache khi cần kiểm tra lại từ đầu

Nếu nghi ngờ cache bị sai lệch:

```bash
# Xóa thư mục cache của Turborepo
rm -rf .turbo

# Hoặc chạy lệnh bỏ qua cache
pnpm turbo run build --force
```

---

## TỔNG KẾT

Tài liệu này được thiết kế để làm quy chuẩn chung cho dự án hiện tại cũng như bất kỳ dự án Monorepo TypeScript nào bạn khởi tạo sau này.  
Để tra cứu các cập nhật mới nhất từ đội ngũ Vercel, hãy luôn đối chiếu với: **[https://turbo.build/repo/docs](https://turbo.build/repo/docs)**.
