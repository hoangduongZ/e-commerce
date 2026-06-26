# 03 — Quy Trình Phát Triển (Flow Chuẩn Doanh Nghiệp)

> Mục tiêu: mỗi task đi theo một vòng đời rõ ràng, có ID, có kiểm soát chất lượng, truy vết được từ yêu cầu → code → release.

---

## 1. Vòng đời một task (Ticket lifecycle)

```
        ┌──────────────────────── Backlog Refinement ────────────────────────┐
        ▼                                                                     │
   [Backlog] ──(đủ DoR)──▶ [To Do] ──▶ [In Progress] ──▶ [Code Review] ──▶ [QA] ──▶ [Done]
                                            │                                  │
                                            └────────── [Blocked] ◀────────────┘
```

1. PO/Lead tạo task với ID `ECM-NNN`, mô tả, Acceptance Criteria.
2. **Backlog Refinement:** ước lượng SP, làm rõ, gắn phụ thuộc → đạt **Definition of Ready**.
3. **Sprint Planning:** kéo task vào sprint (`To Do`).
4. Engineer nhận task → `In Progress` → tạo branch theo ID.
5. Hoàn thành → mở PR → `Code Review` (≥1 approve, CI xanh).
6. Merge → `QA` (QA kiểm theo AC).
7. Đạt **Definition of Done** → `Done`.

---

## 2. Definition of Ready (DoR)

Một task được phép vào sprint khi:
- [ ] Có mô tả rõ ràng + Acceptance Criteria đo được.
- [ ] Đã ước lượng Story Point.
- [ ] Phụ thuộc (dependencies) đã xác định, task chặn đã xong hoặc song song được.
- [ ] Đã rõ ảnh hưởng schema/API (nếu có).
- [ ] Đủ nhỏ (≤ 8 SP); nếu lớn hơn phải tách.

## 3. Definition of Done (DoD)

Một task được đóng khi:
- [ ] Code hoàn thành đúng Acceptance Criteria.
- [ ] Unit test + (nếu là use-case) integration test, coverage không giảm dưới ngưỡng.
- [ ] Có migration Flyway (nếu đổi schema) + cập nhật entity.
- [ ] Cập nhật OpenAPI/Swagger cho endpoint mới.
- [ ] PR được ≥1 reviewer approve, CI xanh (build + test + lint + scan).
- [ ] Không lỗi linter/format; không secret trong code.
- [ ] Đã merge vào `develop` và QA pass.
- [ ] Cập nhật tài liệu liên quan nếu cần.

---

## 4. Git workflow

**Mô hình:** Trunk-based nhẹ với nhánh `develop` tích hợp + nhánh release.

| Nhánh | Vai trò |
|---|---|
| `main` | Production, chỉ nhận từ `release/*` (tag `v*`) |
| `develop` | Tích hợp liên tục, deploy môi trường `dev` |
| `feature/ECM-NNN-mo-ta` | Phát triển task |
| `bugfix/ECM-NNN-mo-ta` | Sửa lỗi |
| `release/x.y.z` | Đóng băng cho UAT/Staging |
| `hotfix/ECM-NNN-mo-ta` | Sửa khẩn cấp từ `main` |

**Quy ước commit (Conventional Commits + ID):**
```
feat(catalog): them API tao san pham [ECM-029]
fix(order): release ton kho khi huy don [ECM-065]
```

**Pull Request:**
- Tiêu đề: `[ECM-NNN] <mô tả ngắn>`.
- Mô tả: link task, tóm tắt thay đổi, cách test, ảnh hưởng schema/API.
- Bắt buộc: CI xanh + ≥1 approve. Task chạm bảo mật/thanh toán cần Lead review.
- Squash merge để giữ lịch sử sạch.

---

## 5. CI/CD Pipeline (GitHub Actions)

**CI — chạy mỗi PR:**
1. Checkout + setup JDK 21 + cache Maven.
2. `mvn verify` (build + unit + integration test với Testcontainers).
3. Spotless/Checkstyle check (format + style).
4. OWASP Dependency-Check / Trivy (quét lỗ hổng).
5. SonarQube (coverage, code smell, security hotspot).
6. Báo cáo coverage; chặn merge nếu dưới ngưỡng.

**CD:**
- Merge `develop` → build Docker image → deploy môi trường `dev`.
- Tạo `release/x.y.z` → deploy `staging` cho UAT.
- Tag `vx.y.z` trên `main` → deploy `prod` (có approval gate).

---

## 6. Chuẩn code & chất lượng

- **Format:** Spotless (Google Java Format hoặc Palantir).
- **Style:** Checkstyle rule set thống nhất.
- **Test:**
  - Unit: domain/service logic (Mockito).
  - Integration: repository + use-case quan trọng (Testcontainers PostgreSQL/Redis).
  - Contract: response API khớp OpenAPI.
  - Đặc biệt: test đồng thời cho reserve tồn kho.
- **Coverage tối thiểu:** 70% line ở `app`/`domain` (cấu hình SonarQube quality gate).
- **Logging:** structured (JSON) + correlation/trace id; không log dữ liệu nhạy cảm (mật khẩu, token, số thẻ).
- **Error handling:** `GlobalExceptionHandler` map exception → `ErrorCode` chuẩn; không nuốt lỗi im lặng.

---

## 7. Quản lý môi trường & cấu hình

- Cấu hình theo profile (`application-{env}.yml`).
- Secret qua biến môi trường / Vault — không commit.
- Migration tự chạy khi khởi động (Flyway), trừ prod có thể chạy có kiểm soát.

---

## 8. Theo dõi tiến độ

- Board Kanban/Scrum (Jira/Linear/Azure DevOps) import từ [backlog.csv](backlog.csv).
- Burndown chart mỗi sprint.
- Mỗi task cập nhật trạng thái realtime; `Blocked` phải ghi rõ task/điều kiện chặn.
- Demo cuối sprint theo Acceptance Criteria của các task `Done`.
