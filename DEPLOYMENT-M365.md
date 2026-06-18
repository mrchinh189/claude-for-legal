# Triển khai Claude for Microsoft 365 cho bộ phận Legal

Hướng dẫn này dành cho IT/admin và trưởng nhóm pháp chế khi đưa **Claude for
Microsoft 365** (add-in Word/Excel/PowerPoint/Outlook) cùng các plugin trong
repo này vào sử dụng cho bộ phận legal. Mọi thứ trong repo là markdown + JSON,
không có bước build — "triển khai" ở đây là: cài add-in, bật marketplace từ
GitHub, cài plugin theo mảng việc, và để mỗi luật sư chạy cold-start interview.

> [!IMPORTANT]
> **Mọi output từ các plugin này là bản nháp để luật sư rà soát — không phải tư
> vấn pháp lý, không phải kết luận pháp lý.** Một luật sư phải rà soát, kiểm
> chứng và chịu trách nhiệm chuyên môn cho bất cứ gì rời khỏi tổ chức. Các
> plugin giúp việc rà soát nhanh hơn; chúng không thay thế nó.

---

## 1. Bức tranh tổng thể

Hai mảnh ghép độc lập, cài một lần cho mỗi người dùng:

| Mảnh | Là gì | Cài từ đâu |
|---|---|---|
| **Claude for Microsoft 365 add-in** | Thanh sidebar Claude trong Word/Excel/PowerPoint/Outlook | [Microsoft AppSource](https://marketplace.microsoft.com/en-us/product/office/wa200010453) |
| **Plugin `claude-for-legal`** | Các skill pháp chế hiện ra qua `/` trong sidebar | GitHub repo này (`/plugin marketplace add`) |

Sau khi cài cả hai, mọi skill thuộc plugin đã bật sẽ gọi được bằng `/` ngay
trong sidebar của Word/Excel. Một luồng (thread) duy nhất trải được qua Word,
Excel, PowerPoint và Outlook.

---

## 2. Điều kiện tiên quyết

- [ ] Microsoft 365 (Word + Excel bản desktop hoặc web) cho người dùng legal.
- [ ] Quyền cài add-in từ AppSource — hoặc admin phê duyệt qua **Microsoft 365
      admin center → Integrated apps** nếu tổ chức khóa AppSource.
- [ ] Tài khoản Claude có quyền dùng Claude for Microsoft 365.
- [ ] Quyết định backend cloud (xem mục 3).
- [ ] (Khuyến nghị mạnh) Ít nhất một **research connector** để xác minh trích
      dẫn — CourtListener, Trellis, Descrybe, hoặc Solve Intelligence. Không có
      nó, mọi trích dẫn bị gắn cờ `[verify]`.

---

## 3. Chọn backend cloud (quyết định của IT)

| Backend | Khi nào chọn | Tài liệu |
|---|---|---|
| **Anthropic API** (mặc định) | Mặc định, nhanh nhất để bắt đầu | Cài thẳng từ AppSource |
| **Vertex AI / Bedrock / internal gateway** | Yêu cầu data residency / chạy trên cloud riêng của công ty | Bộ công cụ [`claude-for-msft-365-install`](https://github.com/anthropics/financial-services/tree/main/claude-for-msft-365-install) |

Nếu bộ phận legal yêu cầu dữ liệu hợp đồng không rời khỏi cloud của công ty,
chọn nhánh thứ hai và làm theo `claude-for-msft-365-install` trước khi rollout
diện rộng.

---

## 4. Các skill chạy trong Word/Excel

Đây là tập skill được viết riêng để hoạt động trong sidebar M365. Đưa danh sách
này cho người dùng — đó là lý do họ cài.

### Word — output dạng tracked changes (rà soát hợp đồng)

| Lệnh | Plugin | Làm gì trong Word |
|---|---|---|
| `/commercial-legal:review` | commercial-legal | Rà MSA/NDA/SaaS theo playbook, trả về tracked changes |
| `/commercial-legal:amendment-history` | commercial-legal | Truy vết thay đổi qua bản gốc + các phụ lục |
| `/ip-legal:ip-clause-review` | ip-legal | Rà điều khoản IP — assignment, license, warranties |
| `/ai-governance-legal:vendor-ai-review` | ai-governance-legal | Rà điều khoản AI của nhà cung cấp |
| `/privacy-legal:dpa-review` | privacy-legal | Rà DPA với vai controller hoặc processor |
| `/corporate-legal:diligence-issue-extraction` | corporate-legal | Bóc issue từ tài liệu VDR theo ngưỡng nội bộ |

Người rà soát chấp nhận/từ chối từng thay đổi như với markup của người thật —
giữ nguyên numbering, defined terms, cross-reference và styles.

### Excel — output dạng workbook `.xlsx` mở sạch

| Lệnh | Plugin | Workbook xuất ra |
|---|---|---|
| `/corporate-legal:tabular-review` | corporate-legal | Bảng review nhiều sheet, mỗi dòng một tài liệu, có sheet nguồn |
| `/litigation-legal:claim-chart` | litigation-legal | Claim chart theo từng yếu tố, có cột trích dẫn |
| `/corporate-legal:entity-compliance` | corporate-legal | Sổ tuân thủ pháp nhân với cột hạn nộp |
| `/commercial-legal:renewal-tracker` | commercial-legal | Sổ gia hạn, sắp theo ngày cancel-by |

> Các plugin chứa những skill này: **commercial-legal, corporate-legal,
> ip-legal, ai-governance-legal, privacy-legal, litigation-legal.** Đây là tập
> tối thiểu nên bật cho rollout M365. Các plugin khác vẫn dùng được nhưng không
> tối ưu riêng cho sidebar Word/Excel.

---

## 5. Quy trình cài (mỗi người dùng)

```text
1) Cài add-in Claude for Microsoft 365 từ AppSource (hoặc admin đẩy qua
   Integrated apps).

2) Mở Claude Code (terminal) HOẶC Claude Cowork (desktop) một lần để thêm
   marketplace và cài plugin:

   /plugin marketplace add https://github.com/mrchinh189/claude-for-legal

3) Cài các plugin phục vụ M365 (chọn theo mảng việc):

   /plugin install commercial-legal@claude-for-legal
   /plugin install corporate-legal@claude-for-legal
   /plugin install privacy-legal@claude-for-legal
   /plugin install ip-legal@claude-for-legal
   /plugin install ai-governance-legal@claude-for-legal
   /plugin install litigation-legal@claude-for-legal

   → Khi được hỏi scope, chọn USER scope (không phải project scope).

4) Restart Claude Code/Cowork. Bước này BẮT BUỘC — plugin chưa "sống" cho tới
   khi restart.

5) Chạy cold-start interview cho từng plugin đã cài (2 phút quick start hoặc
   10–15 phút bản đầy đủ). Mọi skill khác đọc từ profile mà bước này tạo ra:

   /commercial-legal:cold-start-interview
   /corporate-legal:cold-start-interview
   /privacy-legal:cold-start-interview
   /ip-legal:cold-start-interview
   /ai-governance-legal:cold-start-interview
   /litigation-legal:cold-start-interview

6) Kết nối một research tool (CourtListener / Trellis / Descrybe / Solve
   Intelligence). Trong Cowork: Settings → Connectors. Trong Claude Code: sẽ
   được nhắc authorize lần đầu một skill cần tới.

7) Mở Word/Excel → mở sidebar Claude → gõ "/" để thấy các skill đã bật.
```

> **Tại sao USER scope?** Project scope chặn plugin đọc file ngoài thư mục dự
> án — hợp đồng trong Documents, file khách trong Dropbox… Hầu hết skill cần đọc
> file của bạn. USER scope không cấp thêm quyền truy cập file; nó chỉ cho plugin
> chạy được từ mọi thư mục.

---

## 6. Checklist rollout cho IT/Admin

**Pha 0 — Chuẩn bị**
- [ ] Chốt backend cloud (mục 3); nếu là Vertex/Bedrock/gateway thì chạy
      `claude-for-msft-365-install` trước.
- [ ] Phê duyệt add-in Claude trong Microsoft 365 admin center (Integrated apps)
      nếu AppSource bị khóa.
- [ ] Xác nhận license Claude for M365 cho nhóm legal.
- [ ] Chuẩn bị tài khoản research connector (ít nhất một cái).

**Pha 1 — Pilot (2–3 luật sư, 1 mảng việc)**
- [ ] Cài theo mục 5 cho nhóm pilot.
- [ ] Mỗi người pilot chạy cold-start với tài liệu seed thật (MSA đã ký,
      playbook, memo rà soát cũ) — càng nhiều seed, output càng sắc.
- [ ] Chạy thử một skill Word (vd `/commercial-legal:review`) và một skill Excel
      (vd `/corporate-legal:tabular-review`) end-to-end, kiểm tracked changes /
      workbook mở sạch.
- [ ] Thu phản hồi, tinh chỉnh practice profile (`CLAUDE.md` của từng plugin).

**Pha 2 — Diện rộng**
- [ ] Phát hướng dẫn người dùng (mục 5) + danh sách skill (mục 4).
- [ ] Nhắc nguyên tắc: **mọi output là bản nháp để luật sư rà soát.**
- [ ] Đặt lịch `/plugin update` định kỳ để cập nhật plugin từ repo.

**Vận hành**
- [ ] Cập nhật plugin: `/plugin update`.
- [ ] Practice profile sống ở
      `~/.claude/plugins/config/claude-for-legal/<plugin>/CLAUDE.md` — sửa trực
      tiếp cho chỉnh nhỏ; nó tồn tại qua các lần update plugin.

---

## 7. Xử lý sự cố thường gặp

| Triệu chứng | Nguyên nhân & cách xử lý |
|---|---|
| "Command not found" sau khi cài | Chưa restart Claude Code/Cowork (bước 4). |
| "Run setup first" | Chạy `/<plugin>:cold-start-interview` trước khi dùng skill khác. |
| Trích dẫn bị gắn `[verify]` | Chưa kết nối research tool (bước 6) — mọi cite đang lấy từ training data. |
| "I can't read [file]" | Thường là plugin đang ở project scope còn file nằm ngoài thư mục dự án — cài lại USER scope. |
| Không thấy skill trong sidebar Word/Excel | Plugin chưa bật, hoặc add-in M365 chưa cài/đăng nhập. |

---

## 8. Tham chiếu

- [DEPLOYMENT-M365-CHECKLIST.md](DEPLOYMENT-M365-CHECKLIST.md) — checklist công việc theo pha.
- [DEPLOYMENT-M365-PERSONAL.md](DEPLOYMENT-M365-PERSONAL.md) — cài cá nhân trên Office 365 Personal (không cần IT).
- [DEPLOYMENT-M365-LAWYER-GUIDE.md](DEPLOYMENT-M365-LAWYER-GUIDE.md) — hướng dẫn 1 trang cho luật sư.
- [DEPLOYMENT-M365-IT-SOP.md](DEPLOYMENT-M365-IT-SOP.md) — quy trình chuẩn cho IT/admin.
- [QUICKSTART.md](QUICKSTART.md) — cài trong 60 giây (Claude Code/Cowork).
- [README.md](README.md) — tham chiếu đầy đủ: toàn bộ agent, skill, connector.
- [CONNECTORS.md](CONNECTORS.md) — chuẩn một MCP connector pháp lý tốt.
- [`claude-for-msft-365-install`](https://github.com/anthropics/financial-services/tree/main/claude-for-msft-365-install)
  — triển khai add-in trên cloud riêng (Vertex/Bedrock/gateway).
