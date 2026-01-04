# DriveMate System

DriveMate là hệ thống đa nền tảng phục vụ quản lý/điều phối hoạt động liên quan đến học lái/đặt lịch/nguồn lực, gồm:

- **Backend**: Microservices trên **ASP.NET Core (.NET 8)** + **API Gateway (Ocelot)**
- **WebApp**: **Next.js 15** (React 19, TypeScript, TailwindCSS)
- **Mobile**: **Expo / React Native** (TypeScript, Redux Toolkit, SignalR)

## Repository Structure

```text
Project/
├── backend/                 # .NET 8 microservices + docker-compose
├── webapp/                  # Next.js web application
└── moblie/                  # Expo / React Native application
```

## Tech Stack

- **Backend**: .NET 8, Ocelot, Swagger, JWT, EF Core, Hangfire, SignalR
- **Database**: PostgreSQL
- **Messaging**: RabbitMQ (CloudAMQP)
- **WebApp**: Next.js 15, React 19, TypeScript, TailwindCSS
- **Mobile**: Expo, React Native, TypeScript, Redux Toolkit, Axios, SignalR

## Prerequisites

- **Docker Desktop** (khuyến nghị để chạy backend nhanh)
- **.NET SDK 8** (nếu chạy backend local)
- **Node.js 18+** (khuyến nghị **20+**) + npm

## Services & Ports

### Docker Compose (khuyến nghị)

| Component | Container Port | Host Port |
|---|---:|---:|
| API Gateway | 80 | **8080** |
| User Service | 80 | **8081** |
| Booking Service | 80 | **8082** |
| Resource Service | 80 | **8083** |
| Payment Service | 80 | **8084** |
| PostgreSQL | 5432 | **5432** |

### Local Development (dotnet run)

| Service | Local URL |
|---|---|
| API Gateway | `http://localhost:5000` |
| User Service | `http://localhost:5100` |
| Booking Service | `http://localhost:5200` |
| Payment Service | `http://localhost:5300` |
| Resource Service | `http://localhost:5400` |
| Messaging Service | `http://localhost:5500` |

## Quickstart (Docker)

### 1) Start Backend

Chạy toàn bộ backend + database:

```bash
# từ thư mục backend/
docker compose up --build
```

- API Gateway: `http://localhost:8080`
- Gateway Swagger UI: `http://localhost:8080/swagger`

### 2) Start WebApp

```bash
# từ thư mục webapp/
npm install
npm run dev
```

Mặc định WebApp chạy ở: `http://localhost:3000`

### 3) Start Mobile

```bash
# từ thư mục moblie/
npm install
npm start
```

## Environment Variables

> Lưu ý: các file `.env` trong repo hiện đang được `.gitignore` nên README này mô tả **danh sách biến môi trường cần có**. Hãy tạo `.env` theo nhu cầu môi trường của bạn.

### WebApp (`webapp/.env`)

Các biến đang được sử dụng trong code:

```env
# Base URL của API Gateway
NEXT_PUBLIC_BASE_URL=http://localhost:8080

# FPT AI (On-boarding/Identification)
NEXT_PUBLIC_FPT_AI_ID_API_URL=<url>
NEXT_PUBLIC_FPT_AI_DLC_API_URL=<url>
NEXT_PUBLIC_FPT_AI_API_KEY=<api_key>
```

### Mobile (`moblie/.env`)

Các biến đang được sử dụng trong code:

```env
# Base URL của API Gateway
EXPO_PUBLIC_API_URL=http://localhost:8080

# Axios timeout (ms)
EXPO_PUBLIC_API_TIMEOUT=15000

# Key dùng để lưu token trong AsyncStorage
EXPO_PUBLIC_STORAGE_TOKEN=@token

# WebApp URL (mở trong WebView / linking)
EXPO_PUBLIC_URL_WEBAPP=http://localhost:3000

# SignalR base url (Messaging service)
# Ví dụ local: http://localhost:5500
EXPO_PUBLIC_URL_HUB=http://localhost:5500

# Maps (Goong)
EXPO_PUBLIC_GOONG_API_KEY=<goong_api_key>

# FPT AI (OCR / license)
EXPO_PUBLIC_FPT_AI_ID_API_URL=<url>
EXPO_PUBLIC_FPT_AI_DLC_API_URL=<url>
EXPO_PUBLIC_FPT_AI_API_KEY=<api_key>
```

### Backend (`backend/.env`)

Backend dùng `DotNetEnv.Env.Load("../../.env")` trong các service (xem `backend/src/*/Program.cs`).

Tối thiểu bạn cần cấu hình các nhóm sau (tuỳ cách chạy):

- **JWT**: `Jwt:Key`, `Jwt:Issuer`, `Jwt:Audience`
- **Database connection strings** (PostgreSQL)
- **RabbitMQ** (Booking/Payment service)
- **Third-party**: SpeedSMS, Cloudinary, Payment providers (nếu có)

Khi chạy bằng Docker Compose, các biến môi trường chính đã được khai báo trực tiếp trong `backend/docker-compose.yml`.

> Khuyến nghị: không commit secret vào git. Dùng `.env` local hoặc Secrets của CI/CD.

## Backend: API Gateway Routes

API Gateway (Ocelot) map upstream paths (client gọi) sang downstream services.

Ví dụ:

- `GET/POST ... /auth/*` -> User Service `/api/auth/*`
- `... /booking/*` -> Booking Service `/api/booking/*`
- `... /payment/*` -> Payment Service `/api/payment/*`
- `... /resource/*` -> Resource Service `/api/resource/*`
- `... /chat/*`, `... /notifications/*` -> Messaging Service

Chi tiết xem:

- `backend/src/ApiGateway/ocelot.json`
- `backend/src/ApiGateway/ocelot.Development.json`

## Development Notes

- **Swagger**
  - Microservices expose swagger: `http://localhost:<service-port>/swagger`
  - Gateway aggregates swagger via `SwaggerForOcelot`.
- **SignalR (Mobile)**
  - Hubs:
    - `CHAT`: `/chatHub`
    - `NOTIFICATION`: `/notificationHub`
  - Base URL được lấy từ `EXPO_PUBLIC_URL_HUB`.

## Scripts

### WebApp

```bash
npm run dev
npm run build
npm run start
npm run lint
```

### Mobile

```bash
npm start
npm run android
npm run ios
npm run web
npm test
```

## Documentation

- Backend package diagram: `backend/src/PACKAGE_DIAGRAM.md`
- Mobile architecture: `moblie/README.md`

## Contributing

- Tạo branch theo convention: `feature/*`, `fix/*`
- Commit message rõ ràng
- Không commit secrets (`.env`, access tokens, API keys)

---

If you run into issues setting up environment variables (especially `.env` being ignored), tell me bạn đang chạy theo **Docker** hay **Local**, mình sẽ giúp map chính xác config cần thiết cho từng module.
