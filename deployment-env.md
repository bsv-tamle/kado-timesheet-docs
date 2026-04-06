# Env Mapping & Runbook (Local vs AWS)

## 1. Mục tiêu
Tránh nhầm lẫn cấu hình giữa local Docker và môi trường deploy AWS.

## 2. Hai lớp `.env` trong local
### 2.1 `kado-timesheet-api/.env`
Vai trò:
- Cấu hình cho Docker Compose.
- Quyết định cổng local, DB container, auto setup, admin mặc định.

Biến chính:
- `APP_PORT`
- `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, `DB_EXTERNAL_PORT`
- `ADMIN_EMAIL`, `ADMIN_PASSWORD`, `ADMIN_FULL_NAME`
- `AUTO_SETUP`, `AUTO_MIGRATE`

### 2.2 `kado-timesheet-api/src/.env`
Vai trò:
- Cấu hình runtime của Laravel app.
- Dùng bởi `php artisan`, app config, mail/cache/session...

Biến DB trong file mẫu đã chuẩn hóa:
- `DB_CONNECTION=pgsql`
- `DB_HOST=db`
- `DB_PORT=5432`
- `DB_DATABASE=kado`
- `DB_USERNAME=kado`
- `DB_PASSWORD=kado`

## 3. Thứ tự ưu tiên env khi chạy Docker
1. Env được inject từ `docker-compose.yml` (cao nhất).
2. Giá trị trong `src/.env`.

Ý nghĩa:
- Dù `src/.env` có khác, app chạy trong container vẫn ưu tiên env Compose.
- Tuy nhiên vẫn nên giữ `src/.env` đồng bộ để người đọc không nhầm.

## 4. Runbook local chuẩn
```bash
cd /Users/tamle/Project/kado/kado-timesheet-api
cp .env.example .env
cp src/.env.example src/.env
docker compose up -d --build
docker compose exec app php artisan migrate:fresh --seed
```

Kiểm tra:
```bash
docker compose ps
docker compose exec app php artisan migrate:status
docker compose exec db psql -U kado -d kado -c "SELECT id, full_name, email, role FROM users;"
```

## 5. Quy ước deploy AWS (không nhầm lẫn)
- Không dùng file `.env` trong repo làm nguồn cấu hình production.
- Inject env trực tiếp từ hạ tầng (ECS Task Definition / EC2 env / SSM Parameter Store / Secrets Manager).
- Tách rõ:
  - Secret: `DB_PASSWORD`, `APP_KEY`, JWT secret, SMTP password...
  - Non-secret: `APP_ENV`, `APP_URL`, `DB_HOST`, `DB_PORT`...

Khuyến nghị:
- Dùng `APP_ENV=production`, `APP_DEBUG=false`.
- Không bật `AUTO_SETUP`, `AUTO_MIGRATE` ở production.
- Migration production chạy qua pipeline/command thủ công có kiểm soát.

## 6. Mapping biến đề xuất cho AWS runtime
- `APP_NAME=Kado Timesheet API`
- `APP_ENV=production`
- `APP_DEBUG=false`
- `APP_URL=https://<api-domain>`
- `DB_CONNECTION=pgsql`
- `DB_HOST=<rds-endpoint>`
- `DB_PORT=5432`
- `DB_DATABASE=<prod-db>`
- `DB_USERNAME=<prod-user>`
- `DB_PASSWORD=<secret>`
- `CACHE_STORE=database|redis`
- `QUEUE_CONNECTION=database|redis`

## 7. Lưu ý vận hành
- Không commit file:
  - `kado-timesheet-api/.env`
  - `kado-timesheet-api/src/.env`
- Nếu reset DB local:
```bash
docker compose down -v
docker compose up -d --build
docker compose exec app php artisan migrate:fresh --seed
```
