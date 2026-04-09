# Individual reflection — Đoàn Văn Tuấn (2A202600046)

## 1. Role
User stories 4 paths, thiết kế flow cho demo, viết agent tool

## 2. Đóng góp cụ thể

- **User stories 4 paths:** Thiết kế 2 feature chính (AI gợi ý chuyên khoa + Cảnh báo triệu chứng nguy hiểm) với 4 path mỗi feature (Tuy nhiên nhóm thống nhất chỉ tập trung vào một feature chính là AI gợi ý chuyên khoa):
  - Happy path: AI gợi ý chính xác, lễ tân xác nhận ngay → rút ngắn thời gian xử lý
  - Low-confidence path: hiển thị top-3 + confidence score giúp lễ tân quyết định nhanh hơn
  - Failure path: AI sai → lễ tân phát hiện → lưu log correction để improve model
  - Correction path: closed-loop learning từ user feedback vào training data

- **Flow demo:** Soạn user journey từ bệnh nhân nhập triệu chứng → AI gợi ý → lễ tân xác nhận/sửa → hệ thống lưu signal học. Giúp nhóm demo rõ ràng flow augmentation (lễ tân luôn quyết định cuối, không tự động).

- **Agent tool spec:** Viết spec cho AI agent gọi API, xử lý triệu chứng → chuyên khoa mapping + confidence scoring, chuẩn bị code design cho team.

## 3. SPEC mạnh/yếu
- **Mạnh nhất:** 
  - Failure modes section — nhóm xác định 2 failure mode thực tế nhất:
    1. Triệu chứng chung chung → AI gợi ý quá rộng (Recall thấp) → mitigation: ưu tiên high-recall + auto-ask follow-up
    2. Model bias trên rare diseases → Recall thấp ở nhóm triệu chứng hiếm → mitigation: active learning + trọng số recall khi retrain
  - Eval metrics rõ ràng: chọn **Recall** làm metric cốt lõi (≥95%) thay vì accuracy chung chung, với hard threshold cho Safety Red Flag (under-triage ≤1.5%)
  - Learning signal structure: xác định cách dữ liệu correction từ lễ tân tuân vòng closed-loop (log → retrain → improve)

- **Yếu nhất:** 
  - ROI analysis quá đơn giản — 3 kịch bản chỉ khác số user/ngày, assumption gần giống nhau (% giảm thời gian). Nên phân hóa:
    - Conservative = 1 chi nhánh thử nghiệm, ROI ~1-3 triệu VND/ngày
    - Realistic = 3-5 chi nhánh, ROI ~5-10 triệu VND/ngày  
    - Optimistic = toàn bộ hệ thống Vinmec, ROI ~20+ triệu VND/ngày
  - Mini AI spec quá ngắn — chưa nêu rõ data input format, schema của symptoms, specialist mapping, hay API contract

## 4. Đóng góp khác
- **Test prompt với 10+ triệu chứng:** Kiểm chứng prompt LLM có đúng mapping symptoms → specialists trên context Vinmec. Ghi log: input → output gợi ý → confidence % → lees tân xác nhận/sửa. Giúp identify prompt gaps sớm.

- **Brainstorm product risks:** Có đóng góp vào xác định các kill criteria (Project dừng nếu: Net âm 2 tháng liên tục / Safety Red Flag > 3% / CSAT < 3.5 / Adoption < 40%)

## 5. Điều học được
- **Precision vs Recall không phải engineering decision thuần túy:** Ban đầu em nghĩ đó là metric chuyên môn. Sau khi thiết kế AI, em hiểu được:
  - Recall cao (≥95%) ưu tiên nhất vì bỏ sót specialist đúng = bệnh nhân chờ lâu / under-triage / rủi ro sức khỏe → sản phẩm fail
  - Precision đủ (≥60%) để tránh quá nhiều gợi ý rác → lễ tân bỏ qua hoặc frustration
  - **Metric là product strategy, không chỉ engineering decision** — phải align với business KPI (rút ngắn hàng chờ) + user pain (lễ tân hợp lý)

- **Augmentation + closed-loop learning là chìa khóa:** Không thể tự động 100% vì triệu chứng không rõ ràng + risk safety. Nhưng mỗi lần lễ tân sửa gợi ý AI → nó là gold label + signal để improve. Cần design chính xác cách capture correction để retrain hiệu quả.

- **"Happy path" = AI đúng + lễ tân nhanh quyết, không phải AI cứu cánh:** Demo design phải rõ ràng: lễ tân vẫn là người quyết định, AI chỉ đẩy nhanh quá trình từ 5-10 phút xuống <1 phút (scroll gợi ý + click chọn).

## 6. Nếu làm lại
- **Test prompt sớm hơn:** Ngày đầu (D5 tối) chỉ nên dành 2 giờ viết SPEC framework, rồi ngay bắt đầu test prompt với 5 triệu chứng thực tế. Chờ đến trưa D6 mới test → mất 12+ giờ có thể dùng iterate. Nếu test từ D5 tối → D6 sáng có 2-3 vòng iterate prompt, chất lượng sẽ tốt hơn 30-40%.

- **Confirm lệnh Vinmec ASAP:** Spec chưa validate with actual Vinmec specialist mapping. Nên hỏi bác sĩ/lễ tân sớm: 
  - Các triệu chứng điển hình nào? (Đau ngực → Tim mạch? Cơ Xương Khớp?)
  - Có trường hợp một triệu chứng → nhiều chuyên khoa không?
  - Đâu là "rare" symptoms mà AI hay miss?

- **Build mock API + demo sớm:** Thay vì viết spec rồi chờ các bạn code, em sẽ tự viết mock API endpoint (Flask/FastAPI đơn giản) để test user flow sớm. Demo với mock data sẽ hơn chỉ slides.

## 7. AI giúp gì / AI sai gì
- **Giúp rất nhiều:**
  - **Claude và Gemini brainstorm failure modes:** Tự động gợi ý nhiều trường hợp - những case nhóm chưa nghĩ đến. Tuy không tất cả đều applicable cho Vinmec.
  - **ChatGPT brainstorm recovery flow:** Gợi ý các cách "revert" gợi ý sai, fallback khi AI confidentnot-confident, logging format - giúp vẽ rõ system flow.

- **AI sai/mislead:**
  - **Metrics hallucination:** Khi hỏi "metrics cho medical triage" → AI list 20 metrics (Sử dụng Gemini), chưa phân biệt critical vs nice-to-have. Em phải tự sắp xếp prioritize lại.