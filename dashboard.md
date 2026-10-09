# OPERATING DASHBOARD — AI Nutrition Agent (P-110)

**Loại mô hình:** B2B2C — phòng khám dinh dưỡng trả 140.000₫/người bệnh/tháng; người dùng thật là người chăm sóc (khách của phòng khám), chạm qua web app của P-110, dẫn vào bằng ZNS trên Zalo OA của phòng khám · **Cập nhật:** 09/10/2026 · Hà Thị Mỹ Linh – 2A202602619
**NORTH STAR:** Người bệnh theo dõi tích cực/tuần (có thực đơn `approved` đang hiệu lực + ≥4/7 ngày ghi bữa) — hiện tại **0** — mục tiêu **120** ngày 07/01/2027

*"Hiện" = chưa có đối tác thật; số trong ngoặc là giả định mô hình Day 22. Chi tiết định nghĩa, phép tính [MH]: `worksheet.md`.*

### Đèn báo sớm (Leading — nhìn hằng tuần)

| Đèn | Hiện | 🟢 / 🟡 / 🔴 | Nguồn | Báo trước cho |
|---|---|---|---|---|
| ⭐ Partner activation — end-user thật (≥1 thực đơn duyệt + ≥3 bữa ghi) trong 30 ngày từ go-live | — | ≥15 / 10–14 / **<10** (khi ≥2 đối tác: ≥60% / 30–60% / <30%) | [TB] ½ pilot 20 người | Người bệnh tính phí → GM, Pilot→trả phí |
| Kích hoạt người chăm sóc tuần đầu (≥4/7 ngày có bữa `confirmed`) | — | ≥BL×1,3 / BL±30% / **<BL×0,7** | [TB] baseline 09/11/2026 | Người bệnh tính phí, D30 |
| Tỷ lệ duyệt nguyên bản (approve, ≤1 món sửa, không reject) | (70%) | ≥70% / 57–70% / **<57%** | [MH] GM 60% | Chi phí/DT → GM |

### Đèn vận hành (Operating — nhìn hằng tuần/tháng)

| Đèn | Hiện | 🟢 / 🟡 / 🔴 | Nguồn | Báo trước cho |
|---|---|---|---|---|
| Người bệnh tính phí / đối tác (từ tháng trả phí) | — (120) | ≥79 / 53–78 / **<53** | [MH] hạ tầng $62,4 cố định | GM, Pilot→trả phí |
| **Chi phí AI** theo lượt dùng (LLM + ảnh + ZNS) ÷ doanh thu, **từng đối tác** | (11,5%) | ≤16,6% / 16,6–23,6% / **>23,6%** | [MH] GM 60% / 50% | GM — trước ~1 tháng |
| Ca an toàn lọt / 100 thực đơn đã duyệt (chuyên gia xác nhận) | — | 0 / ≤1, không dị ứng-thuốc / **>1 hoặc ≥1 dị ứng-thuốc** | [TB] CS-03; eval 13/14 | Pilot→trả phí, gia hạn |

### Đèn kết quả (Lagging — nhìn hằng quý)

| Đèn | Hiện | 🟢 / 🟡 / 🔴 | Nguồn |
|---|---|---|---|
| GM theo từng đối tác | (67,4%) | ≥60% / 50–60% / **<50%** | [MH] Day 22 · [BM] AI-native 53% 2026P (ICONIQ State of AI 2026, kiểm tra 09/10/2026) |
| Pilot → hợp đồng trả phí | — | ≥50% / 36–50% / dưới 36% · 1 đối tác: ký ≤30 / 31–60 / >60 ngày sau pilot | [BM] ICONIQ GTM 2026: ~50% (2026), ~36% (2025), kiểm tra 09/10/2026 |

### 5 luật quyết định (⏹ = luật dừng)

1. ⏹ **NẾU** < 10 end-user thật **TRONG** 30 ngày từ go-live **THÌ** dừng tìm đối tác mới; tuần sau Linh + Hưng ngồi quầy tái khám 2 buổi đăng ký tận tay; bật báo cáo tuần cho chuyên gia **KHÔNG THÌ** không mở pilot đối tác thứ 2, không báo cáo "số đối tác đã ký".
2. ⏹ **NẾU** duyệt nguyên bản < 57% **TRÊN** 2 tuần liền **VÀ** ≥20 thực đơn/tuần **THÌ** dừng nhận người bệnh mới; Hưng sửa 2 lý do sửa/từ chối nhiều nhất, chạy lại eval golden rồi mới mở **KHÔNG THÌ** không nới định nghĩa, không nhờ chuyên gia "bớt sửa".
3. **NẾU** chi phí theo lượt dùng > 23,6% DT **TRONG** 2 tuần liền **VÀ** đối tác ≥20 người bệnh **THÌ** trong 3 ngày tách chi phí theo thành phần + p95 người bệnh, đặt trần 7 ảnh/tuần, đàm phán pass-through phần vượt **KHÔNG THÌ** không tăng giá đại trà, không bỏ bước chuyên gia duyệt.
4. ⏹ **NẾU** ≥1 ca dị ứng/tương tác thuốc lọt hoặc > 1 ca/100 **TRÊN** 100 thực đơn gần nhất **THÌ** trong ngày tắt chế độ LLM, dừng nhận người bệnh mới 7 ngày, báo Partner A bằng văn bản trong 24 h, thêm ca vào golden set **KHÔNG THÌ** không sửa im lặng, không đổ lỗi chuyên gia.
5. **NẾU** người bệnh tính phí < 53 **TRONG** 2 tháng trả phí liền **THÌ** đàm phán lại: cam kết tối thiểu 53 người hoặc phí nền 1,63 triệu₫/tháng; đặt QR ở 1 phòng khám ĐTĐ/thận **KHÔNG THÌ** không ký thêm đối tác để bù volume, không giảm giá.

### Cổng gác 90 ngày (ngày 0 = 09/10/2026)

| Ngày | Metric (1) | Ngưỡng qua cổng | Bằng chứng | Nếu trượt |
|---|---|---|---|---|
| 30 · 08/11 | Phút chuyên gia **soạn tay** 1 thực đơn tuần tại đối tác (học — neo giá Day 22) | Có số đo trên ≥10 thực đơn **và** trung bình ≥30 phút | Biên bản pilot đã ký + file bấm giờ 10 thực đơn | Không có số → FIX 1 lần: chuyển sang đối tác dự phòng (khoa DD BV tỉnh / PK tư). <30 phút → PIVOT value metric (giá vượt trần lương) |
| 60 · 08/12 | Partner activation: end-user thật trong 30 ngày từ go-live | ≥10 | SQL export `meal_logs` + `plan_review_events` theo đối tác; file chi phí theo lượt dùng của đối tác từ `agent_runs` | FIX 1 lần (luật 1); lần 2 → PIVOT sang đối tác khác loại |
| 90 · 07/01 | Hợp đồng trả phí đã ký | 140.000₫/người bệnh, cam kết ≥53 người bệnh | Hợp đồng ký + Pilot Report có số (duyệt nguyên bản, ca an toàn, chi phí/DT) | Duyệt nguyên bản pilot <57% → PIVOT (biến thể / value metric); còn lại → KILL theo dòng dưới |

**KILL CRITERIA:** Đến **07/01/2027** không đối tác nào ký trả phí với ≥53 người bệnh, **hoặc** tỷ lệ duyệt nguyên bản cả pilot (≥80 thực đơn) < 46% (GM < 50%) sau 1 lần FIX → dừng hướng B2B2C qua phòng khám.

**CHƯA ĐO ĐƯỢC:** ① **Runway** — mô hình chưa có vốn ban đầu + dòng tiền tháng (cần: bảng dòng tiền; có số: 31/10/2026) · ② **Phút chuyên gia duyệt** 1 thực đơn — DB không ghi lúc mở thực đơn (cần: sự kiện `open` hoặc bấm giờ tay trong pilot; 15/11/2026) · ③ **Token thật** — `agent_runs` chưa tách cached tokens, chi phí ảnh/ZNS chưa gắn người bệnh (cần: cột cached + hoá đơn Zalo; 08/12/2026) · ④ **End-user reach** — không có tổng người bệnh của phòng khám (cần: điều khoản pilot đối tác báo số; 08/12/2026) · ⑤ **Volume volatility, NRR** — cần ≥3 / 12 tháng vận hành (01/2027 / 10/2027) · ⑥ Baseline kích hoạt tuần đầu: 09/11/2026 · ⑦ Giá ZNS chính chủ (đang dùng bảng đại lý ⚠️).
