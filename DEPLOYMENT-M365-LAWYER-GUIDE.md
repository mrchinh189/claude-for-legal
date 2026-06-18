# Claude trong Word & Excel — Hướng dẫn nhanh cho luật sư

Một trang. In ra hoặc ghim lại. Mục tiêu: bạn dùng được Claude ngay trong Word
và Excel để rà hợp đồng và dựng bảng diligence.

> **Nhớ một điều:** mọi thứ Claude trả về là **bản nháp để bạn rà soát**. Bạn
> kiểm chứng, sửa, và chịu trách nhiệm — Claude chỉ giúp nhanh hơn.

---

## Cài lần đầu (5 phút, làm một lần)

1. **Cài add-in** Claude for Microsoft 365 (từ AppSource, hoặc IT đã đẩy sẵn cho
   bạn). Mở Word → tab/sidebar Claude hiện ra.
2. **Thêm bộ plugin pháp chế** — mở Claude Code/Cowork một lần, gõ:
   ```
   /plugin marketplace add https://github.com/mrchinh189/claude-for-legal
   ```
3. **Cài plugin của bạn** (chọn theo việc bạn làm) — khi hỏi scope, chọn
   **USER scope**:
   ```
   /plugin install commercial-legal@claude-for-legal
   ```
4. **Restart** Claude (bắt buộc — chưa restart thì lệnh chưa chạy).
5. **Chạy setup** một lần, đưa Claude tài liệu mẫu của bạn (MSA đã ký, playbook):
   ```
   /commercial-legal:cold-start-interview
   ```
6. Xong. Mở Word/Excel → sidebar Claude → gõ `/` để thấy các lệnh.

---

## Trong Word — rà hợp đồng (trả về tracked changes)

Mở file hợp đồng trong Word, mở sidebar Claude, gõ:

| Bạn cần | Gõ |
|---|---|
| Rà MSA / NDA / SaaS theo playbook | `/commercial-legal:review` |
| Truy vết thay đổi qua các phụ lục | `/commercial-legal:amendment-history` |
| Rà điều khoản IP | `/ip-legal:ip-clause-review` |
| Rà điều khoản AI của nhà cung cấp | `/ai-governance-legal:vendor-ai-review` |
| Rà DPA (controller/processor) | `/privacy-legal:dpa-review` |

Claude trả markup dạng **tracked changes** — bạn Accept/Reject từng thay đổi như
markup của đồng nghiệp. Numbering, defined terms, cross-reference được giữ nguyên.

## Trong Excel — dựng bảng (trả về workbook .xlsx)

| Bạn cần | Gõ |
|---|---|
| Bảng review data room, mỗi dòng một tài liệu | `/corporate-legal:tabular-review` |
| Claim chart theo từng yếu tố | `/litigation-legal:claim-chart` |
| Sổ tuân thủ pháp nhân theo hạn nộp | `/corporate-legal:entity-compliance` |
| Sổ gia hạn theo ngày cancel-by | `/commercial-legal:renewal-tracker` |

---

## 3 lỗi hay gặp

- **"Command not found"** → bạn quên restart sau khi cài. Restart Claude.
- **"Run setup first"** → chạy `/<plugin>:cold-start-interview` trước.
- **Trích dẫn gắn `[verify]`** → chưa kết nối research tool; báo IT bật
  CourtListener/Trellis/Descrybe/Solve Intelligence. Không có nó, cite lấy từ
  trí nhớ mô hình, không phải cơ sở dữ liệu hiện hành.

Cần thêm: xem [DEPLOYMENT-M365.md](DEPLOYMENT-M365.md) hoặc hỏi đầu mối IT.
