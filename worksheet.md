# Worksheet — AI Nutrition Agent (P-110)

Họ tên: Hà Thị Mỹ Linh · MSSV: 2A202602619 · Ngày làm: 09/10/2026

## Số liệu đầu vào

Nguồn: bài Mô hình tài chính & Value Metric (Day 22, file `HaThiMyLinh_Day22_model.xlsx`, giá kiểm tra 08/10/2026,
tỷ giá 26.160₫/USD). Tất cả là **giả định / ước tính** — P-110 chưa có đối tác và người dùng thật.

| Số | Giá trị | Ô nguồn |
|---|---|---|
| Job | 1 thực đơn tuần (21 bữa) được chuyên gia duyệt và giao tới người chăm sóc | 1_Cost_Job!B5 |
| Giá / ARPU | 140.000₫/người bệnh/tháng ($1,24/job) · **$642/đối tác/tháng** (120 người bệnh) | 2_Pricing!B19 · 4_Channel_Fit!B5 |
| Gross margin | **67,4%** (55,0% nếu tính overhead $56/đối tác/tháng) | 2_Pricing!B21 · C63 |
| CAC | Ngân sách CAC **$5.194/đối tác** (= $642 × 67,4% × 12). CAC thực của kênh Partner-Led: **chưa đo** | 4_Channel_Fit!B9 |
| CAC payback mục tiêu | **12 tháng** (SMB — Bessemer, Scaling to $100M, 2024) | 4_Channel_Fit!B8 · 6_Benchmarks!C38 |
| Runway | **Chưa có.** Mô hình chưa có số vốn ban đầu và dòng tiền theo tháng (ROI_BUSINESS_CASE.md §4 đã bỏ payback/IRR vì lý do này) | — |
| Cost/Job | **$0,404** (≈10.570₫) · $0,558 có overhead. Hạ tầng + Zalo ZNS + ảnh chiếm 60%, LLM 6% | 1_Cost_Job!B66:B67 |
| Breakeven containment | 57,0% (GM 60%) · GM < 50% khi containment < 46% · hiện ước tính 70% | 2_Pricing!B33 · B56 · 1_Cost_Job!B10 |
| Hạ tầng cố định | $62,4/tháng → GM < 50% khi 1 đối tác < ~53 người bệnh | 1_Cost_Job!E41 · 2_Pricing!B57 |

## Trạm 1 — Loại mô hình

**Ba câu hỏi, theo thực tế hôm nay** (09/10/2026 — chưa ký đối tác nào; trả lời theo mô hình đã chốt ở Day 22):

1. **Ai trả tiền?** Phòng khám / trung tâm dinh dưỡng (doanh nghiệp): 140.000₫/người bệnh đang theo dõi/tháng, đối tác
   đầu tiên nhắm tới là **Partner A** — một trung tâm khám tư vấn dinh dưỡng công lập ở Hà Nội (ẩn tên vì chưa được đối
   tác cho phép, theo RULES.md). Gia đình **không** trả: 58% người
   trả lời khảo sát chỉ chịu chi < 100.000₫/tháng (SURVEY_REPORT, n = 79).
2. **Ai dùng?** Người dùng hằng ngày là **người chăm sóc / người bệnh — khách của phòng khám**: ghi bữa, nhận cảnh báo,
   nhận thực đơn tuần. Chuyên gia của phòng khám chỉ dùng app duyệt (~1 lượt/người bệnh/tuần).
3. **Có chạm được end-user không?** **Có.** Người chăm sóc đăng nhập web app của P-110 bằng tài khoản của P-110 (SĐT + OTP),
   P-110 giữ dữ liệu của họ (`meal_logs`, `meal_alerts`, `agent_runs` gắn `patient_id`), và cảnh báo hiện trực tiếp trên
   màn hình của P-110. Tin ZNS 5h45 gửi qua Zalo OA của Partner A chỉ là điểm dẫn vào app.

**Câu chốt loại:** Chúng tôi là **B2B2C** vì tiền đến từ **phòng khám dinh dưỡng** (140.000₫/người bệnh/tháng), người dùng
thật là **người chăm sóc của người bệnh mạn tính — khách của phòng khám**, và chúng tôi chạm họ qua **web app người chăm sóc
của P-110** (tài khoản, dữ liệu bữa ăn và cảnh báo nằm ở phía chúng tôi), được dẫn vào bằng tin ZNS trên Zalo OA của phòng
khám. Không phải B2B vì phòng khám chỉ duyệt thực đơn, còn lượng dùng hằng ngày (và hoá đơn token, phí ZNS) do số người bệnh
phòng khám đưa vào quyết định, không do chúng tôi.

**Bảng đèn §3.3 B2B2C** (ghi đủ 9 đèn; ✅ đo được hôm nay · 🔧 đo được trong 2 tuần · ❌ chưa đo được):

| Đèn | ✅ / 🔧 / ❌ | Số nằm ở đâu / cần gì để đo |
|---|---|---|
| **L · Partner activation rate** ⭐ | 🔧 | Có dữ liệu: người bệnh → chuyên gia phụ trách → `expert_profiles.organization`; end-user thật = có ≥1 `meal_logs` đã xác nhận. Cần: chuẩn hoá `organization` thành mã đối tác + loại tài khoản demo/test (seed). Hôm nay mẫu số = 0 đối tác go-live |
| **L · End-user reach trong partner** | ❌ | Tử số có (số người bệnh của đối tác có tài khoản). Mẫu số = tổng người bệnh mạn tính đang theo dõi tại phòng khám — dữ liệu của đối tác, P-110 không có. Cần: điều khoản pilot để phòng khám báo số này mỗi tháng |
| **L · Time-to-first-end-user** | 🔧 | Ngày end-user đầu tiên = `min(meal_logs.confirmed_at)` theo đối tác. Ngày ký: ghi tay khi ký biên bản pilot (chưa có bảng hợp đồng trong DB) |
| **O · Volume volatility** | ❌ | Cách đo đã rõ: số thực đơn `approved`/tháng theo đối tác (`plan_review_events`). Nhưng cần ≥3 tháng vận hành thật mới có độ lệch chuẩn — sớm nhất 01/2027 |
| **O · GM sau rev-share** | 🔧 | Hôm nay chỉ có số mô hình 67,4% (chưa có rev-share vì phòng khám là bên trả tiền, không chia doanh thu). Cần: token thật từ `agent_runs`, hoá đơn Zalo ZNS, hoá đơn DigitalOcean theo tháng + điều khoản hợp đồng có/không chia doanh thu |
| **O · Chi phí inference ÷ doanh thu theo TỪNG partner** | 🔧 | `agent_runs` đã ghi `input_tokens`/`output_tokens` theo `patient_id` → gộp được theo đối tác. Cần: bảng giá model theo ngày + cột cached tokens (hiện chưa tách, nên số tính ra sẽ cao hơn thật) + chi phí ảnh/vision và ZNS theo người bệnh |
| **O · Tập trung volume** | 🔧 | Tính được ngay khi có đối tác đầu tiên từ `plan_review_events`. Biết trước kết quả: 90 ngày đầu chỉ có 1 đối tác → **100%, đỏ theo cấu trúc** |
| **O · Chất lượng nhìn từ end-user** | 🔧 | Offline đã có ✅: phát hiện vi phạm 13/14, duyệt-nguyên-bản proxy 5/7 hồ sơ (`eval/results/report.md`, 07/10). Trên end-user thật: `plan_change_requests` (xin đổi món, `urgency = safety`), thực đơn bị `reject`. Cần: SLA chất lượng ký với đối tác + kênh khiếu nại |
| **G · Doanh thu/partner · partner NRR · GM tổng** | ❌ | Chưa có doanh thu (pilot tháng 1 miễn phí). Doanh thu/partner có số sớm nhất sau tháng trả phí đầu tiên; NRR cần 12 tháng hợp đồng |

## Trạm 2 — Thẻ đèn

**North Star:** **Người bệnh theo dõi tích cực / tuần** — người bệnh có thực đơn tuần `approved` đang hiệu lực **và** có
≥4 ngày ghi bữa (`meal_logs.status = confirmed`) trong 7 ngày qua; không đếm tài khoản demo/seed, nhân viên đối tác hay
nhóm P-110 — hiện tại **0** (chưa có đối tác) — mục tiêu **120 vào ngày 90** (Day 22: tháng 2–3 = 1 đối tác × 120 người
bệnh trả phí).

*Vì sao không lấy Partner activation làm North Star:* 90 ngày đầu chỉ có 1 đối tác, nên tỷ lệ này chỉ là 0/1 hoặc 1/1 và
không nhích theo tuần. Người bệnh theo dõi tích cực đổi mỗi tuần, và nó là chính đơn vị tính tiền của phòng khám (tháng
không có thực đơn được duyệt thì không thu phí).

**Cây 3 tầng:**

```
LEADING                         OPERATING                              LAGGING
1 Partner activation ⭐ ───────► 4 Người bệnh tính phí / đối tác ─────► 7 GM theo từng đối tác
2 Kích hoạt người chăm sóc ───► 4                                 ┌──► 8 Pilot → hợp đồng trả phí
3 Tỷ lệ duyệt nguyên bản ─────► 5 Chi phí theo lượt dùng ÷ DT ─────► 7
                                6 Ca an toàn lọt / 100 thực đơn ──┘
```

| # | Tầng (L/O/G) | Đèn | Định nghĩa (đếm gì · **không** đếm gì) | Công thức | Nhịp · ai lấy số | Báo trước cho |
|---|---|---|---|---|---|---|
| 1 | L | **Partner activation rate** ⭐ | Đối tác đã go-live có **≥10 end-user thật** trong 30 ngày kể từ go-live. *Go-live* = ngày người bệnh đầu tiên được phân cho một chuyên gia đã xác minh của đối tác (`expert_profiles.organization`). *End-user thật* = người bệnh đã nhận ≥1 thực đơn `approved` **và** có ≥3 bữa `confirmed` sau đó. **Không** đếm: tài khoản seed/demo, tài khoản của nhân viên đối tác hoặc nhóm P-110, hồ sơ chỉ có thực đơn `pending_review`/`rejected`. Lấy ngưỡng 10 thay vì 1 vì một user có thể chỉ là người thân của chuyên gia dùng thử | Số đối tác có ≥10 end-user thật trong 30 ngày ÷ số đối tác đã go-live ≥30 ngày | Tuần · Linh (SQL trên DB) | #4 Người bệnh tính phí/đối tác → #7 GM theo đối tác, #8 Pilot → trả phí |
| 2 | L | **Kích hoạt người chăm sóc tuần đầu** | Trong cohort người bệnh nhận thực đơn `approved` **đầu tiên** trong tuần, những người có **≥4 trên 7 ngày** có ≥1 bữa `confirmed`, tính từ lúc thực đơn được giao. **Không** đếm: log `discarded`/`needs_confirmation`, log do chuyên gia/support nhập hộ, bản ghi "nhớ lại" (không sinh cảnh báo), tài khoản demo | Số người bệnh đạt ≥4/7 ngày có log ÷ số người bệnh trong cohort tuần | Tuần (theo cohort) · Linh | #4 Người bệnh tính phí (có xin thực đơn tuần sau không) · D30 người chăm sóc (PE-02) |
| 3 | L | **Tỷ lệ duyệt nguyên bản** (containment) | Thực đơn AI soạn được chuyên gia `approve` với **≤1** `edit_item` và **không** bị `reject` trước đó ở cùng version. Mẫu số = thực đơn đã có quyết định cuối (approve/reject) trong tuần. **Không** đếm: thực đơn còn `pending_review`, version do chuyên gia tự tạo khi xử lý "Xin đổi món", thực đơn của hồ sơ demo | Số thực đơn duyệt nguyên bản ÷ số thực đơn đã có quyết định | Tuần · Hưng (từ `plan_review_events`) | #5 Chi phí/DT (vì đây là mẫu số của Cost/Job) → #7 GM · #8 (đối tác thấy chuyên gia có thật sự đỡ việc không) |
| 4 | O | **Người bệnh tính phí / đối tác** | Người bệnh có ≥1 thực đơn `approved` được giao trong tháng, đúng điều kiện tính phí của Day 22; trong pilot miễn phí vẫn đếm như "đủ điều kiện tính phí". **Không** đếm: người bệnh chỉ có thực đơn chờ duyệt hoặc bị từ chối, tài khoản demo, người bệnh đã chuyển đối tác (tính cho đối tác nhận) | Số người bệnh đủ điều kiện tính phí của đối tác X trong tháng | Tháng (xem tạm theo tuần) · Linh | #7 GM theo đối tác (hạ tầng cố định $62,4 chia cho ít người hơn) · #8 |
| 5 | O | **Chi phí theo lượt dùng ÷ doanh thu — theo TỪNG đối tác** (đèn chi phí AI) | Chi phí phát sinh theo mỗi lượt dùng của người bệnh thuộc đối tác X: token LLM (`agent_runs.input_tokens/output_tokens` × giá model tại ngày tính) + nhận diện ảnh + tin Zalo ZNS + lượt chạy lại. **Không** đếm: hạ tầng cố định (VPS, backup), lượt chạy eval/dev/demo, giờ chuyên gia của đối tác (biến thể A — đối tác tự chịu). Không lấy trung bình mọi đối tác | Σ chi phí theo lượt dùng của đối tác X ÷ (số người bệnh tính phí × 140.000₫) | Tuần · Hưng (script gộp `agent_runs` theo đối tác + hoá đơn ZNS) | #7 GM theo đối tác — thấy trước khoảng 1 tháng so với lúc chốt sổ |
| 6 | O | **Ca an toàn lọt / 100 thực đơn** (chất lượng nhìn từ end-user) | Yêu cầu "Xin đổi món" trên thực đơn **đã duyệt** có `urgency = safety` **và** được chuyên gia xử lý bằng cách đổi món, tức là xác nhận có vấn đề thật (dị ứng, tương tác thuốc, vượt ngưỡng). **Không** đếm: yêu cầu bị `declined` (chuyên gia xác nhận an toàn), lý do `dislike`/`unavailable`/`expensive`/`hard_to_cook` | Số ca an toàn được xác nhận ÷ số thực đơn `approved` đã giao × 100 | Tuần · Linh + chuyên gia đối tác xác nhận | #8 Pilot → trả phí · gia hạn (sản phẩm hỏng là khủng hoảng của phòng khám, họ sẽ gỡ mình ra) |
| 7 | G | **GM theo từng đối tác** | (Doanh thu − COGS) ÷ doanh thu của đối tác X. COGS = chi phí theo lượt dùng (#5) + hạ tầng phân bổ theo số job + QA nội bộ 5%. Rev-share hiện = 0 vì phòng khám là bên trả tiền; có điều khoản chia thì trừ thêm. **Không** đếm overhead hỗ trợ đối tác ($56/tháng, báo riêng) | (DT − COGS − rev-share) ÷ DT, từng đối tác | Tháng · Linh | — (bảng điểm) |
| 8 | G | **Pilot → hợp đồng trả phí** | Đối tác kết thúc pilot 4 tuần **ký hợp đồng có phí** trong 30 ngày sau pilot. **Không** đếm: LOI, "đồng ý miệng", hợp đồng giảm 100%, gia hạn pilot miễn phí | Số đối tác ký trả phí ÷ số đối tác đã xong pilot | Quý (mỗi đối tác) · Linh | — (bảng điểm) |

Đèn chi phí AI là đèn số: **5** (đèn số 3 cũng bắt chi phí AI từ phía mẫu số: duyệt nguyên bản tụt thì Cost/Job tăng trước
khi GM kịp xấu đi).

Kiểm tra: 3 Leading · 3 Operating · 2 Lagging (25%) · 1 đèn chi phí AI · mọi đèn L/O đều có cột "báo trước cho".

**Đèn §3.3 đã bỏ và lý do:**

| Đèn bỏ | Lý do |
|---|---|
| End-user reach trong partner | Không có mẫu số (tổng người bệnh của phòng khám), đã thay bằng North Star tính số tuyệt đối |
| Time-to-first-end-user | Đã gộp vào #1 (cửa sổ 30 ngày từ go-live) |
| Volume volatility | Cần ≥3 tháng vận hành thật mới có số; 90 ngày đầu chưa ra được quyết định → ghi vào "Chưa đo được" |
| Tập trung volume | 90 ngày đầu chỉ có 1 đối tác, nên 100% theo cấu trúc: biết trước đáp án, không dẫn tới quyết định nào |
| Doanh thu/partner · partner NRR | NRR cần 12 tháng hợp đồng; doanh thu/partner đã nằm trong #4 × 140.000₫ |
| Phút chuyên gia duyệt 1 thực đơn (PE-04) | Đáng đo nhất vì neo giá Day 22 gãy nếu soạn tay dưới 30 phút, nhưng DB không ghi lúc chuyên gia **mở** thực đơn → chỉ đo được bằng bấm giờ tay trong pilot → "Chưa đo được" |

## Trạm 3 — Ngưỡng

| # | Đèn | 🟢 | 🟡 | 🔴 | Nguồn [BM]/[MH]/[TB] | Lý do (1 câu) · ngày kiểm tra nếu [BM] |
|---|---|---|---|---|---|---|
| 1 | Partner activation rate ⭐ | ≥60% đối tác · *giai đoạn 1 đối tác:* ≥15 end-user thật trong 30 ngày | 30–60% · *10–14* | <30% · ***<10*** | [TB] | Tỷ lệ lấy theo HANDBOOK §3.3 (tự đặt, chưa có chuẩn ngành). Khi chỉ có 1 đối tác thì đổi sang số tuyệt đối: 10 = một nửa nhóm pilot 20 người bệnh (Day 22, 5_90Day_Plan!B13); dưới 10 người thì Pilot Report không đủ mẫu cho #3 và #6. Thay bằng baseline thật khi có đối tác thứ 2 |
| 2 | Kích hoạt người chăm sóc tuần đầu | ≥ baseline × 1,3 | baseline ±30% | < baseline × 0,7 | [TB] | Chưa có chuẩn, và khảo sát P-110 không hỏi tần suất ghi bữa. Đo 2 cohort đầu của pilot (dự kiến tuần 19/10 và 26/10/2026) để lấy baseline, **có số ngày 09/11/2026**. Trước ngày đó đèn chỉ được vàng, không kích hoạt luật |
| 3 | Tỷ lệ duyệt nguyên bản | ≥70% | 57–70% | **<57%** | [MH] | 57,0% là containment tối thiểu để GM đạt 60%; 70% là giả định mà Cost/Job $0,404 đang dựa vào (phép tính MH-1) |
| 4 | Người bệnh tính phí / đối tác (từ tháng trả phí đầu tiên) | ≥79 | 53–78 | **<53** | [MH] | Hạ tầng cố định $62,4/tháng: dưới 79 người bệnh thì GM < 60%, dưới 53 người thì GM < 50% (phép tính MH-2) |
| 5 | Chi phí theo lượt dùng ÷ doanh thu (từng đối tác) | ≤16,6% | 16,6–23,6% | **>23,6%** | [MH] | Phần ngân sách chi phí theo lượt dùng còn lại sau khi trừ hạ tầng cố định và QA, ứng với GM 60% và 50%; mô hình hiện ở 11,5% (phép tính MH-3) |
| 6 | Ca an toàn lọt / 100 thực đơn | 0 | ≤1/100 **và** không ca nào là dị ứng / tương tác thuốc | >1/100, **hoặc** ≥1 ca dị ứng / tương tác thuốc | [TB] | Chưa có chuẩn ngành. Mục tiêu CS-03 của P-110 là phát hiện 100%, eval offline mới đạt 13/14 (07/10/2026). Một ca lọt nghĩa là cả AI lẫn chuyên gia đều bỏ sót, và trách nhiệm lâm sàng thuộc về phòng khám. Pilot ~80 thực đơn → chỉ 1 ca là đã đỏ (1,25/100). Baseline: hết pilot, ngày 15/11/2026 |
| 7 | GM theo từng đối tác | ≥60% | 50–60% | **<50%** | [MH] + [BM] đối chiếu | 60% là GM mục tiêu và 50% là vạch "mô hình gãy" của Day 22 (2_Pricing!B32, B56). Đối chiếu: GM công ty AI-native 53% (2026P) — ICONIQ *State of AI 2026* (07/2026), iconiq.com/growth/reports/state-of-ai-2026 — **kiểm tra 09/10/2026**: "45% in 2025 to a projected 53% in 2026, and 59% in 2027" |
| 8 | Pilot → hợp đồng trả phí | ≥50% · *1 đối tác:* ký trong ≤30 ngày sau pilot | 36–50% · *ký sau 31–60 ngày* | <36% · ***chưa ký sau 60 ngày*** | [BM] | POC/free-trial → paid **~50% (2026)**, **~36% (2025)** — ICONIQ *State of Go-to-Market 2026*, iconiq.com/growth/reports/state-of-go-to-market-2026 — **kiểm tra 09/10/2026**: "conversion has climbed to roughly 50%, up 14 points year-over-year". Lấy mức 2026 làm vạch xanh, mức 2025 làm vạch đỏ |

**Tự kiểm tra:** 8/8 đèn có ngưỡng, nguồn và lý do · 4 ngưỡng [MH] (#3, #4, #5, #7) · 2 [BM] đều có ngày · 3 [TB] có
lịch đo. Các ngưỡng [MH] không tròn số (57%, 53, 79, 16,6%, 23,6%) vì được tính ra, không chọn tay.

Hai [BM] đã được mở lại tại nguồn gốc ngày 09/10/2026; số khớp với HANDBOOK §8 (chốt 27/08/2026).

### Phụ lục [MH] — phép tính (≥2)

Đầu vào chung — Day 22 (`HaThiMyLinh_Day22_model.xlsx`, giá kiểm tra 08/10/2026):

```
P   giá / job (1 thực đơn tuần)             = $1,24      (140.000₫ ÷ 4,33 tuần ÷ 26.160)      2_Pricing!B19
J   job thử / tháng / đối tác               = 520        (120 người bệnh × 4,33)               1_Cost_Job!B9
R   containment (duyệt nguyên bản), giả định = 70%                                              1_Cost_Job!B10
u   chi phí theo lượt dùng / job thử        = $0,142     LLM 0,0236 + ảnh 0,028 + ZNS 0,0883 + retry 0,0024
F   hạ tầng cố định / tháng                 = $62,4      (DigitalOcean $48 + backup 30%)       1_Cost_Job!E41
q   QA nội bộ / job thử                     = $0,0208    (5% × 5 phút × $5/giờ)                 2_Pricing!B30
v   = u + F/J = 0,142 + 0,120               = $0,262     chi phí biến đổi / job thử             2_Pricing!B29
Cost/Job = (v + q) × J ÷ (R × J) = (0,262 + 0,0208) ÷ 0,7 = $0,404 → GM = 1 − 0,404/1,24 = 67,4%
```

**[MH] 1 — Tỷ lệ duyệt nguyên bản (#3)**

```
Đầu vào: v = $0,262 · q = $0,0208 · P = $1,24 · biến thể A (chuyên gia đối tác chịu ca escalate → e = 0)
Điều kiện GM ≥ g:  (v + q) ÷ R ≤ P × (1 − g)   ⇒   R ≥ (v + q) ÷ (P × (1 − g))
GM 60%: R ≥ 0,2828 ÷ (1,24 × 0,40) = 0,2828 ÷ 0,496 = 57,0%    (khớp 2_Pricing!B33)
GM 50%: R ≥ 0,2828 ÷ 0,620                         = 45,6%    (khớp 2_Pricing!B56)
Kết quả → 🟢 ≥ 70% (giả định Cost/Job đang dựa vào) · 🟡 57–70% · 🔴 < 57% (GM dưới mục tiêu 60%)
         Dưới 46% là vạch GM 50% → dùng ở KILL CRITERIA.
```

**[MH] 2 — Người bệnh tính phí / đối tác (#4)**

```
Đầu vào: F = $62,4/tháng cố định · u + q = 0,142 + 0,0208 = $0,1628 / job thử · R = 70% · P = $1,24 · 4,33 job/người bệnh/tháng
Với N người bệnh: J = 4,33N.   Cost/Job = ((u+q) × J + F) ÷ (R × J)
Điều kiện GM ≥ g:  (u+q) × J + F ≤ P(1−g) × R × J   ⇒   J ≥ F ÷ (P(1−g)R − (u+q))
GM 60%: P(1−g)R = 1,24 × 0,4 × 0,7 = 0,3472 → J ≥ 62,4 ÷ (0,3472 − 0,1628) = 62,4 ÷ 0,1844 = 338 → N ≥ 338/4,33 = 78,2 → 79
GM 50%: P(1−g)R = 1,24 × 0,5 × 0,7 = 0,4340 → J ≥ 62,4 ÷ 0,2712 = 230 → N ≥ 53,1 → 53   (khớp 2_Pricing!B57)
Kết quả → 🟢 ≥ 79 · 🟡 53–78 · 🔴 < 53
         (Kế hoạch Day 22 là 120 người bệnh; pilot miễn phí 20 người không áp ngưỡng này vì chưa thu tiền.)
```

**[MH] 3 — Chi phí theo lượt dùng ÷ doanh thu (#5, đèn chi phí AI)**

```
Đầu vào: P = $1,24 · R = 70% · J = 520 · F = $62,4 · q × J = $10,83
Phần cố định + QA / job hoàn thành = (62,4 + 10,83) ÷ (0,7 × 520) = 73,23 ÷ 364 = $0,2012
Ngân sách Cost/Job:  GM 60% → ≤ 0,4 × 1,24 = $0,496 ;  GM 50% → ≤ 0,5 × 1,24 = $0,620
Ngân sách cho u / job hoàn thành:  0,496 − 0,2012 = $0,2948 ;  0,620 − 0,2012 = $0,4188
Quy về u / job thử (× R = 0,7):    $0,2064 ;  $0,2932
Chia cho giá / job:                0,2064 ÷ 1,24 = 16,6% ;  0,2932 ÷ 1,24 = 23,6%
Hiện tại (mô hình): 0,142 ÷ 1,24 = 11,5%.  Nếu prompt cache không trúng: u = $0,181 → 14,6% (vẫn xanh)
Kết quả → 🟢 ≤ 16,6% · 🟡 16,6–23,6% · 🔴 > 23,6%
         Đèn chuyển đỏ khi u tăng ~2× so với mô hình, ví dụ người bệnh gửi ảnh nhiều gấp đôi giả định 7 ảnh/tuần,
         hoặc phí ZNS thật cao hơn bảng giá đại lý.
```

**[MH] 4 — GM theo từng đối tác (#7)**

```
Đầu vào: GM mô hình 67,4% (2_Pricing!B21) · GM mục tiêu 60% (2_Pricing!B32) · vạch gãy 50% (2_Pricing!B56)
Phép tính: GM = 1 − Cost/Job ÷ P; mức 60% và 50% chính là 2 ngưỡng dùng cho MH-1 đến MH-3, nên 4 đèn #3, #4, #5, #7
           cùng đổi màu theo một mô hình duy nhất.
Đối chiếu [BM]: AI-native 53% (2026P, ICONIQ) nằm trong vùng vàng của mình → mục tiêu 60% chặt hơn trung vị ngành,
           vì P-110 không trả lương chuyên gia (biến thể A).
Kết quả → 🟢 ≥ 60% · 🟡 50–60% · 🔴 < 50%
```

## Trạm 4 — 5 luật quyết định

Đánh dấu ⏹ cho luật dừng (cần ≥2). Đội: Linh (BA, đầu mối đối tác) + Hưng (planner/backend). Đèn #2 chưa có luật
riêng vì trước 09/11/2026 chưa có baseline; đèn #7 và #8 là bảng điểm, được xử lý ở cổng gác.

1. ⏹ **Partner activation (#1)** — **NẾU** đối tác pilot có **< 10 end-user thật** **TRONG** 30 ngày kể từ go-live
   **THÌ** **dừng** mọi việc tìm đối tác mới (không gửi email, không hẹn gặp BV tuyến tỉnh), và ngay tuần sau Linh + Hưng
   ngồi tại quầy tái khám của Partner A 2 buổi: phát QR, đăng ký tận tay cho người bệnh cùng chuyên gia. Song song, Linh hỏi
   trưởng Partner A "nếu bỏ P-110 ngày mai, Partner A mất gì?", rồi bật báo cáo tuần tự động cho chuyên gia để Partner A có
   lợi rõ hơn. **KHÔNG THÌ** không khởi động pilot với đối tác thứ hai, và không báo cáo "số đối tác đã ký / đang trao
   đổi" như thành tích tăng trưởng.

2. ⏹ **Tỷ lệ duyệt nguyên bản (#3)** — **NẾU** tỷ lệ duyệt nguyên bản **< 57%** **TRÊN** 2 tuần liên tiếp **VÀ** mỗi tuần
   có ≥ 20 thực đơn đã có quyết định **THÌ** **dừng** nhận người bệnh mới (giữ nguyên số đang theo dõi). Trong tuần đó,
   Hưng gom mọi `edit_item`/`reject` của 2 tuần, xếp theo lý do, sửa 2 lý do nhiều nhất trong planner, chạy lại
   `eval_clinical_golden.py` cho đạt rồi mới mở lại. **KHÔNG THÌ** không nới định nghĩa "duyệt nguyên bản" (ví dụ lên
   ≤ 2 món) để đèn xanh lại, và không đề nghị chuyên gia "duyệt nhanh / bớt sửa" — chuyên gia sửa ít đi thì số đẹp lên
   nhưng chất lượng tụt (đèn #6).

3. **Chi phí theo lượt dùng ÷ doanh thu (#5)** — **NẾU** chi phí theo lượt dùng của một đối tác **> 23,6% doanh thu**
   **TRONG** 2 tuần liên tiếp **VÀ** đối tác đó có ≥ 20 người bệnh hoạt động **THÌ** trong 3 ngày Hưng tách chi phí theo
   thành phần (LLM / ảnh / ZNS) và theo người bệnh (p95), sau đó đặt trần 7 ảnh/tuần/người bệnh (quá trần thì app chuyển sang
   ghi bằng giọng nói, $0). Linh **đàm phán lại** phụ lục hợp đồng để phần ảnh và ZNS vượt trần được tính pass-through cho
   đối tác. **KHÔNG THÌ** không tăng giá 140.000₫ cho mọi người bệnh (giá đã bằng 69% chi phí soạn tay, sát trần neo nhân
   công của Day 22), và không cắt bước chuyên gia duyệt để bù chi phí.

4. ⏹ **Ca an toàn lọt (#6)** — **NẾU** có **≥ 1 ca dị ứng hoặc tương tác thuốc** lọt tới người chăm sóc, **hoặc > 1 ca
   an toàn / 100 thực đơn** **TRÊN** 100 thực đơn `approved` gần nhất **THÌ** trong ngày Hưng **dừng** sinh thực đơn bằng
   chế độ LLM (quay về chế độ luật) và **dừng** nhận người bệnh mới 7 ngày. Trong 24 giờ, Linh báo trưởng Partner A bằng văn
   bản: ca nào, vì sao lọt, sửa gì. Hưng thêm ca đó vào `clinical_golden.json` và sửa rule, eval đạt mới mở lại.
   **KHÔNG THÌ** không sửa im lặng mà không báo đối tác, và không quy lỗi cho chuyên gia "duyệt sót" — HITL là lớp chặn
   cuối, không phải lớp chặn duy nhất.

5. **Người bệnh tính phí / đối tác (#4)** — **NẾU** số người bệnh tính phí của một đối tác **< 53** **TRONG** 2 tháng trả
   phí liên tiếp **THÌ** Linh **đàm phán lại** hợp đồng theo một trong hai cách: (a) cam kết tối thiểu 53 người bệnh/tháng,
   hoặc (b) phí nền cố định 1,63 triệu₫/tháng (= $62,4 hạ tầng) cộng phí theo người bệnh. Trong 2 tuần đó, Linh cùng trưởng
   Partner A chọn 1 phòng khám ĐTĐ hoặc thận mạn của mình để đặt QR trên phiếu tư vấn. **KHÔNG THÌ** không ký thêm đối
   tác để "bù" volume (đối tác mới tốn thêm một khoản hạ tầng và hỗ trợ cố định), và không giảm giá/người bệnh để kéo thêm
   người.

**Phản xạ sai mà mỗi vế KHÔNG THÌ chặn:**

| Luật | Phản xạ đầu tiên khi đèn đỏ | Vì sao sai với P-110 |
|---|---|---|
| 1 | "Đối tác này không đẩy thì tìm đối tác khác" | Đối tác thứ 2 cũng sẽ không đẩy nếu chưa biết vì sao đối tác 1 không đẩy; chỉ thêm chi phí hỗ trợ $56/tháng |
| 2 | Nới định nghĩa, hoặc nhờ chuyên gia sửa ít đi | Đèn xanh giả; chuyên gia vẫn mất thời gian thật, nên đối tác vẫn không thấy lợi |
| 3 | Tăng giá cho mọi người | Giá đã sát trần neo lương; tăng giá thì mất đối tác để cứu biên do vài người dùng nặng gây ra |
| 4 | Sửa nhanh rồi im lặng | Trong B2B2C, lỗi của mình là khủng hoảng của phòng khám; giấu thì mất kênh (bài học Klarna) |
| 5 | Ký thêm đối tác hoặc giảm giá | Mỗi đối tác mới thêm chi phí cố định; giảm giá làm GM tụt nhanh hơn |

**Kiểm tra:** 5 luật · đủ NẾU / TRONG-TRÊN / THÌ / KHÔNG THÌ · vế VÀ ở luật 2, 3 (mẫu nhỏ dễ nhiễu) · 3 luật dừng ⏹
(1, 2, 4) · mọi vế NẾU dùng đúng ngưỡng 🔴 của Trạm 3 · không luật nào kết thúc bằng "xem xét / theo dõi".
