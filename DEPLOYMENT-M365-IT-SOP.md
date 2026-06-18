# SOP cho IT — Triển khai Claude for Microsoft 365 (bộ phận Legal)

Quy trình chuẩn cho IT/admin để cài, cấu hình và vận hành Claude for Microsoft
365 cùng marketplace `claude-for-legal`. Đường mặc định: **Claude subscription**
(add-in chạy trên dịch vụ Anthropic quản lý — không cần tự dựng API/hạ tầng).

Liên quan: [DEPLOYMENT-M365.md](DEPLOYMENT-M365.md) ·
[checklist](DEPLOYMENT-M365-CHECKLIST.md) ·
[hướng dẫn cho luật sư](DEPLOYMENT-M365-LAWYER-GUIDE.md).

---

## 1. Phạm vi & điều kiện

- Người dùng: bộ phận legal có Microsoft 365 (Word + Excel desktop hoặc web).
- Gói Claude for Microsoft 365 (Team/Enterprise) — xác nhận seat với account
  team Anthropic.
- Quyền admin Microsoft 365 (để duyệt add-in nếu AppSource bị khóa).
- ≥1 research connector cho nhóm.

## 2. Quyết định backend (một lần)

| Lựa chọn | Khi nào | Hành động IT |
|---|---|---|
| **Subscription (mặc định)** | Không bắt buộc data residency | Không cần dựng hạ tầng |
| **Cloud riêng** | Bắt buộc Vertex/Bedrock/internal gateway | Chạy [`claude-for-msft-365-install`](https://github.com/anthropics/financial-services/tree/main/claude-for-msft-365-install) |

Ghi quyết định vào hồ sơ triển khai trước khi sang bước 3.

## 3. Phê duyệt & phân phối add-in

1. Microsoft 365 admin center → **Settings → Integrated apps**.
2. Tìm **Claude for Microsoft 365**, **Deploy/Allow** cho nhóm bảo mật của
   bộ phận legal (không bật toàn tổ chức nếu chưa cần).
3. Chọn assignment: theo nhóm/người dùng đích.
4. Kiểm thử: đăng nhập tài khoản test → mở Word → xác nhận sidebar Claude hiện.

> Nếu chính sách cho phép AppSource tự do, người dùng tự cài từ
> [AppSource](https://marketplace.microsoft.com/en-us/product/office/wa200010453);
> IT chỉ cần đảm bảo không bị Conditional Access chặn.

## 4. Cấu hình marketplace & plugin (nguồn chuẩn)

Nguồn cài chính thức — công bố cho người dùng:
```
/plugin marketplace add https://github.com/mrchinh189/claude-for-legal
```

Tập plugin tối thiểu cho M365 (cài **USER scope**):

```
/plugin install commercial-legal@legal-mrchinh189
/plugin install corporate-legal@legal-mrchinh189
/plugin install privacy-legal@legal-mrchinh189
/plugin install ip-legal@legal-mrchinh189
/plugin install ai-governance-legal@legal-mrchinh189
/plugin install litigation-legal@legal-mrchinh189
```

> **USER scope**, không phải project scope — nếu không, plugin không đọc được
> file ngoài thư mục dự án (hợp đồng trong Documents/Dropbox…). USER scope không
> cấp thêm quyền file; chỉ cho plugin chạy từ mọi thư mục.

Sau cài: **restart** Claude Code/Cowork.

## 5. Connector

| Loại | Connector | Ghi chú |
|---|---|---|
| Research (≥1, bắt buộc để cite tin được) | CourtListener / Trellis / Descrybe / Solve Intelligence | Không có → cite gắn cờ `[verify]` |
| CLM/DMS/VDR (tùy nhu cầu) | Ironclad · DocuSign(/CLM) · iManage · Box | Cấu hình trong `.mcp.json` từng plugin hoặc `claude mcp` |

Connector "customer subscription" cần tài khoản + API key của công ty. Lưu key
theo chính sách secret của tổ chức; đặt lịch xoay key.

## 6. Bảo mật & tuân thủ

- [ ] Xác nhận luồng dữ liệu hợp đồng khớp quyết định backend (mục 2).
- [ ] Giới hạn add-in theo nhóm bảo mật, không bật toàn tổ chức.
- [ ] Lưu API key connector trong vault; cấm hardcode.
- [ ] Ghi lại profile người dùng nằm ở
      `~/.claude/plugins/config/claude-for-legal/<plugin>/CLAUDE.md` (chứa
      playbook nội bộ — coi như dữ liệu nhạy cảm).
- [ ] Truyền thông nguyên tắc bắt buộc: mọi output là bản nháp để luật sư rà soát.

## 7. Vận hành

| Việc | Nhịp | Lệnh/Hành động |
|---|---|---|
| Cập nhật plugin | Hàng tuần/khi có update | `/plugin update` |
| Xoay API key connector | Theo chính sách | Cập nhật `.mcp.json`/vault |
| Rà quyền add-in | Định kỳ | Integrated apps review |
| Hỗ trợ người dùng | Liên tục | Bảng troubleshooting (mục 7, DEPLOYMENT-M365.md) |

## 8. Troubleshooting (IT-facing)

| Triệu chứng | Xử lý |
|---|---|
| Người dùng không thấy add-in | Kiểm Integrated apps assignment + Conditional Access |
| "Command not found" sau cài | Người dùng chưa restart Claude |
| "Run setup first" | Người dùng chưa chạy `/<plugin>:cold-start-interview` |
| Cite gắn `[verify]` | Research connector chưa authorize |
| "I can't read [file]" | Plugin cài project scope → cài lại USER scope |
