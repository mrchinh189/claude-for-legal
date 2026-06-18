# Triển khai cá nhân — Claude trong Word & Excel (Office 365 Personal)

Hướng dẫn cho **một luật sư tự cài** trên máy của mình với Office 365 Personal.
Không cần admin, không cần IT, không cần tenant doanh nghiệp. Sau bước này bạn
gọi được các skill pháp chế ngay trong sidebar Word/Excel.

> [!IMPORTANT]
> Mọi output là **bản nháp để bạn rà soát** — không phải tư vấn pháp lý. Bạn
> kiểm chứng, sửa và chịu trách nhiệm cho bất cứ gì gửi ra ngoài.

> [!NOTE]
> Đây là đường **cá nhân/thử nghiệm**. Office 365 Personal không có admin center
> nên không phân phối được cho cả nhóm. Khi mở rộng cho bộ phận legal, nâng lên
> Microsoft 365 Business/Enterprise và xem [DEPLOYMENT-M365.md](DEPLOYMENT-M365.md).

---

## Điều kiện

- [ ] Office 365 Personal có **Word + Excel** (desktop hoặc web).
- [ ] Tài khoản **Claude** có quyền dùng Claude for Microsoft 365 (kiểm gói với
      Anthropic nếu chưa chắc).
- [ ] (Khuyến nghị) Một **research connector** miễn phí: CourtListener.

---

## Các bước cài (làm một lần, ~10 phút)

### 1. Cài add-in Claude vào Office

- Word/Excel → **Home → Add-ins → Get Add-ins** (hoặc **Insert → Add-ins**).
- Tìm **Claude** trong Office Store, bấm **Add**.
- Hoặc cài từ [Microsoft AppSource](https://marketplace.microsoft.com/en-us/product/office/wa200010453).
- Đăng nhập tài khoản Claude khi được hỏi. Sidebar Claude hiện trong Word.

> Add-in từ Office Store cài được trên Office 365 Personal cho từng máy/từng tài
> khoản — đây là điểm khác với bản nhóm (vốn cần admin duyệt).

### 2. Thêm bộ plugin pháp chế

Mở **Claude Code** (terminal) hoặc **Claude Cowork** (desktop) một lần:

```
/plugin marketplace add https://github.com/mrchinh189/claude-for-legal
```

### 3. Cài plugin theo việc bạn làm — chọn **USER scope**

```
/plugin install commercial-legal@claude-for-legal
```

Thêm plugin khác nếu cần (mỗi mảng một dòng):

```
/plugin install corporate-legal@claude-for-legal
/plugin install privacy-legal@claude-for-legal
/plugin install ip-legal@claude-for-legal
/plugin install ai-governance-legal@claude-for-legal
/plugin install litigation-legal@claude-for-legal
```

> Khi được hỏi scope, chọn **USER scope** (không phải project scope) — nếu không,
> plugin không đọc được file ngoài thư mục dự án (hợp đồng trong Documents…).

### 4. Restart Claude (bắt buộc)

Đóng và mở lại Claude Code/Cowork. Chưa restart thì lệnh chưa chạy.

### 5. Chạy setup một lần cho từng plugin

Đưa Claude tài liệu mẫu của bạn (MSA đã ký, playbook, memo cũ — càng nhiều càng tốt;
có sẵn chế độ quick start 2 phút):

```
/commercial-legal:cold-start-interview
```

### 6. Kết nối research tool

Cowork: **Settings → Connectors → CourtListener**. Claude Code: sẽ được nhắc
authorize lần đầu một skill cần tới. Không có nó, mọi trích dẫn bị gắn cờ `[verify]`.

---

## Dùng hằng ngày

**Trong Word** — mở hợp đồng → sidebar Claude → gõ `/`:

| Cần | Lệnh |
|---|---|
| Rà MSA/NDA/SaaS theo playbook | `/commercial-legal:review` |
| Truy vết thay đổi qua phụ lục | `/commercial-legal:amendment-history` |
| Rà điều khoản IP | `/ip-legal:ip-clause-review` |
| Rà điều khoản AI nhà cung cấp | `/ai-governance-legal:vendor-ai-review` |
| Rà DPA | `/privacy-legal:dpa-review` |

→ Trả về **tracked changes**, bạn Accept/Reject từng thay đổi.

**Trong Excel** — gõ `/`:

| Cần | Lệnh |
|---|---|
| Bảng review data room | `/corporate-legal:tabular-review` |
| Claim chart theo yếu tố | `/litigation-legal:claim-chart` |
| Sổ tuân thủ pháp nhân | `/corporate-legal:entity-compliance` |
| Sổ gia hạn theo cancel-by | `/commercial-legal:renewal-tracker` |

→ Trả về workbook **.xlsx** mở sạch.

---

## Bảo trì

- Cập nhật plugin: `/plugin update`.
- Profile của bạn ở `~/.claude/plugins/config/claude-for-legal/<plugin>/CLAUDE.md`
  — sửa trực tiếp cho chỉnh nhỏ; tồn tại qua các lần update.
- Chạy lại `/<plugin>:cold-start-interview` khi nghiệp vụ đổi nhiều.

## Lỗi hay gặp

| Triệu chứng | Xử lý |
|---|---|
| "Command not found" | Chưa restart Claude (bước 4). |
| "Run setup first" | Chạy `/<plugin>:cold-start-interview` (bước 5). |
| Cite gắn `[verify]` | Chưa kết nối research tool (bước 6). |
| "I can't read [file]" | Cài lại **USER scope** (bước 3). |
| Không thấy Claude trong Word | Add-in chưa cài/đăng nhập (bước 1). |
