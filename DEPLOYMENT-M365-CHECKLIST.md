# Checklist triển khai Claude for Microsoft 365 (bộ phận Legal)

Checklist công việc đưa Claude for Microsoft 365 từ "có repo" tới "vận hành
khai thác". Tích `[x]` khi xong. Đường triển khai mặc định: **dùng Claude
subscription** (Team/Enterprise) — add-in chạy trên dịch vụ Anthropic quản lý,
**không cần tự dựng API key/hạ tầng**. Nhánh cloud riêng (Vertex/Bedrock/gateway)
chỉ cần khi bắt buộc data residency.

> [!IMPORTANT]
> Mọi output là **bản nháp để luật sư rà soát** — không phải tư vấn pháp lý.
> Một luật sư rà soát, kiểm chứng và chịu trách nhiệm cho bất cứ gì rời tổ chức.

Tài liệu liên quan: [DEPLOYMENT-M365.md](DEPLOYMENT-M365.md) ·
[hướng dẫn cho luật sư](DEPLOYMENT-M365-LAWYER-GUIDE.md) ·
[SOP cho IT](DEPLOYMENT-M365-IT-SOP.md).

---

## Pha 0 — Quyết định & phê duyệt

- [ ] **0.1** Chốt phạm vi pilot: 2–3 luật sư + mảng việc (vd contract review).
- [ ] **0.2** Xác nhận dùng **Claude subscription** (mặc định). Chỉ chọn cloud
      riêng nếu Security yêu cầu data residency.
- [ ] **0.3** Rà soát bảo mật/định danh dữ liệu: hợp đồng có được rời cloud
      không? → kết luận bằng văn bản.
- [ ] **0.4** Xác nhận gói/seat Claude for M365 cho nhóm (kiểm với account team
      Anthropic).

## Pha 1 — Hạ tầng & quyền (IT, một lần)

- [ ] **1.1** Phê duyệt add-in Claude trong Microsoft 365 admin center →
      Integrated apps (nếu AppSource bị khóa).
- [ ] **1.2** (Chỉ khi chọn cloud riêng ở 0.2) Chạy `claude-for-msft-365-install`
      trỏ Vertex/Bedrock/gateway. **Bỏ qua nếu dùng subscription.**
- [ ] **1.3** Chuẩn bị ≥1 research connector (CourtListener/Trellis/Descrybe/
      Solve Intelligence) + tài khoản/API key.
- [ ] **1.4** Cấu hình connector hệ thống nội bộ nếu cần (Ironclad/DocuSign/
      iManage/Box) trong `.mcp.json`.
- [ ] **1.5** Công bố nguồn cài chính thức:
      `https://github.com/mrchinh189/claude-for-legal`.

## Pha 2 — Pilot (2–3 luật sư, 1 mảng việc)

- [ ] **2.1** Cài add-in M365 từ AppSource cho người pilot.
- [ ] **2.2** `/plugin marketplace add https://github.com/mrchinh189/claude-for-legal`
- [ ] **2.3** Cài plugin theo mảng việc — **USER scope** (commercial/corporate/
      privacy/ip/ai-governance/litigation).
- [ ] **2.4** **Restart** Claude Code/Cowork (bắt buộc).
- [ ] **2.5** Mỗi người chạy `/<plugin>:cold-start-interview` với tài liệu seed
      thật (MSA đã ký, playbook, memo cũ).
- [ ] **2.6** Authorize research connector → trích dẫn hết gắn cờ `[verify]`.
- [ ] **2.7** Chạy thử 1 skill **Word** end-to-end (vd `/commercial-legal:review`)
      → kiểm tracked changes giữ numbering/defined terms.
- [ ] **2.8** Chạy thử 1 skill **Excel** end-to-end (vd
      `/corporate-legal:tabular-review`) → kiểm `.xlsx` mở sạch, có sheet nguồn.
- [ ] **2.9** Thu phản hồi → tinh chỉnh practice profile từng plugin.

## Pha 3 — Rollout diện rộng

- [ ] **3.1** Phát [hướng dẫn cho luật sư](DEPLOYMENT-M365-LAWYER-GUIDE.md) +
      bảng skill (mục 4 của DEPLOYMENT-M365.md).
- [ ] **3.2** Buổi training ngắn: gọi skill bằng `/`, đọc output, quy trình rà.
- [ ] **3.3** Truyền thông nguyên tắc: mọi output là bản nháp để luật sư rà soát.
- [ ] **3.4** Cài cho toàn nhóm theo quy trình Pha 2.
- [ ] **3.5** Mỗi người chạy cold-start với seed của mình.

## Pha 4 — Vận hành & khai thác

- [ ] **4.1** Cập nhật plugin định kỳ: `/plugin update`.
- [ ] **4.2** Bảo trì practice profile
      (`~/.claude/plugins/config/claude-for-legal/<plugin>/CLAUDE.md`).
- [ ] **4.3** Re-run cold-start khi thay đổi lớn (jurisdiction/CLM/policy mới).
- [ ] **4.4** Rà soát quyền connector & xoay API key theo chính sách bảo mật.
- [ ] **4.5** Duy trì kênh hỗ trợ nội bộ + thu lỗi (bảng troubleshooting).
- [ ] **4.6** Theo dõi chỉ số khai thác: số hợp đồng rà/tuần, thời gian tiết
      kiệm, tỷ lệ skill được dùng.

---

## Tập plugin tối thiểu cho M365

| Plugin | Skill M365 chính |
|---|---|
| commercial-legal | review · amendment-history · renewal-tracker |
| corporate-legal | tabular-review · diligence-issue-extraction · entity-compliance |
| privacy-legal | dpa-review |
| ip-legal | ip-clause-review |
| ai-governance-legal | vendor-ai-review |
| litigation-legal | claim-chart |
