# Báo Cáo: Khai Phá Luật Kết Hợp Có Trọng Số (Weighted Association Rules) - Phân Tích Giỏ Hàng Bán Lẻ

## Bài Toán và Dữ Liệu

### Bài Toán
Trong lĩnh vực bán lẻ, việc hiểu mối quan hệ giữa các sản phẩm thường được mua cùng nhau giúp tối ưu hóa chiến lược kinh doanh. Tuy nhiên, các luật kết hợp truyền thống chỉ dựa trên tần suất xuất hiện, có thể bỏ qua giá trị kinh tế của từng giao dịch. Dự án này mở rộng với **luật kết hợp có trọng số** (weighted association rules), ưu tiên các luật xuất hiện trong hóa đơn giá trị cao, và khám phá **luật niche** (hiếm nhưng giá trị cao). Ngoài ra, giới thiệu **High-Utility Itemset Mining (HUIM)** để tối ưu theo lợi nhuận từng sản phẩm.

### Dữ Liệu
- **Nguồn**: Bộ dữ liệu Online Retail từ công ty bán lẻ UK (2010-2011).
- **Sau xử lý**: 485.123 giao dịch hợp lệ từ 18.021 hóa đơn UK.
- **Các trường chính**: InvoiceNo, Description, Quantity, UnitPrice, TotalPrice, InvoiceValue (tổng giá trị hóa đơn).
- **Basket matrix**: 18.021 hóa đơn × 4.007 sản phẩm, dạng boolean (có/không mua).

## Pipeline: Apriori, FP-Growth, Weighted Rules & HUIM

### 1. Apriori Algorithm
- **Nguyên lý**: Tìm tập phổ biến bằng cách sinh candidates và prune.
- **Tham số**: min_support=0.01, max_len=3.
- **Ưu điểm**: Đơn giản, dễ hiểu.
- **Nhược điểm**: Chậm với datasets lớn (thời gian ~5 phút cho 18k transactions).

### 2. FP-Growth Algorithm
- **Nguyên lý**: Xây dựng FP-Tree để tránh sinh candidates thừa.
- **Tham số**: min_support=0.01, max_len=3.
- **Ưu điểm**: Nhanh hơn Apriori (~1.7 phút), hiệu quả cho dữ liệu lớn.
- **Nhược điểm**: Phức tạp hơn về memory cho tree.

### 3. Weighted Association Rules
- **Ý tưởng**: Gán trọng số cho giao dịch dựa trên InvoiceValue (tổng giá trị hóa đơn).
- **Công thức**:
  - Weighted Support = Tổng InvoiceValue của transactions chứa itemset / Tổng InvoiceValue toàn bộ.
  - Weighted Confidence = Weighted Support(itemset) / Weighted Support(antecedents).
  - Weighted Lift = Weighted Confidence / Weighted Support(consequents).
- **Lợi ích**: Phát hiện luật quan trọng trong hóa đơn VIP, dù support thấp.

### 4. High-Utility Itemset Mining (HUIM)
- **Ý tưởng**: Tối ưu theo "utility" (lợi nhuận) của từng item, không chỉ tần suất.
- **Demo đơn giản**: Utility của item = UnitPrice trung bình; Utility của itemset = Tổng utility của transactions chứa nó.
- **Khác biệt**: Frequent (số lần), Weighted (tổng hóa đơn), HUIM (lợi nhuận item).

## Kết Quả Chính

### Luật Niche (Support Thấp, Weighted Support Cao hoặc Weighted Lift Cao)
- **Tiêu chí**: Support ≤ 0.02, Weighted Support ≥ 0.05.
- **Số lượng**: 1.326 luật (từ FP-Growth).
- **Ví dụ top luật**:
  - "REGENCY TEA PLATE GREEN → REGENCY TEA PLATE ROSES" (Support: 0.016, Weighted Support: 0.056, Weighted Lift: 11.9).
  - "SMALL MARSHMALLOWS PINK BOWL → SMALL DOLLY MIX DESIGN ORANGE BOWL" (Support: 0.019, Weighted Support: 0.055, Weighted Lift: 11.5).
- **Lý do đáng chú ý**:
  - Nhắm phân khúc VIP/đơn hàng lớn: Các luật này xuất hiện ít nhưng trong hóa đơn giá trị cao (premium sets, seasonal collections).
  - Sản phẩm giá cao: Như tea plates, stationery sets – phù hợp niche marketing.
- **Chiến lược niche marketing**:
  - Gợi ý gói combo cao cấp: Bundle "REGENCY TEA PLATE" series với discount cho khách mua full set.
  - Chương trình chăm sóc riêng: Email personalized cho khách đã mua một item, gợi ý item liên quan dựa trên lịch sử mua VIP.

### So Sánh Apriori vs FP-Growth
- **Số rules**: Cả hai sinh 3.856 rules trước filter, 2.008 sau filter.
- **Thời gian**: Apriori ~5 phút, FP-Growth ~1.7 phút (nhanh hơn 3x).
- **Độ chính xác**: Giống hệt, vì cùng tham số và dữ liệu.
- **Khuyến nghị**: Dùng FP-Growth cho datasets lớn.

### HUIM Demo
- Top itemsets by Utility:
  - NATURAL SLATE HEART CHALKBOARD: Utility 1.11e8, Utility/Support 1.64e9.
  - JUMBO BAG RED RETROSPOT: Utility 1.23e8, Utility/Support 1.15e9.
- **Ý nghĩa**: Ưu tiên sản phẩm đắt tiền dù ít bán, khác với frequent mining.

## Trực Quan Hóa Kết Quả

### 1. Bar Chart: Phân Bố Weighted Lift của Top Niche Rules
- **Mô tả**: Biểu đồ cột cho top 10 luật niche theo weighted lift.
- **Hành vi mua sắm**: Thể hiện luật nào mạnh nhất trong phân khúc cao cấp – khách VIP ưu tiên combos premium.
- **Điểm khác biệt Apriori/FP-Growth**: Không khác biệt, vì kết quả giống; FP-Growth chỉ nhanh hơn.

### 2. Scatter Plot: Support vs Weighted Support
- **Mô tả**: Điểm phân tán cho tất cả niche rules (X: support, Y: weighted support, size: weighted lift).
- **Hành vi mua sắm**: Luật ở góc trên-trái là "niche gems" – ít phổ biến nhưng giá trị kinh doanh cao, phản ánh mua sắm tinh tế của khách hàng giàu có.
- **Điểm khác biệt Apriori/FP-Growth**: Không có, vì thuật toán sinh rules giống nhau.

### 3. Network Graph: Quan Hệ Sản Phẩm trong Luật Niche
- **Mô tả**: Đồ thị mạng với nodes là sản phẩm, edges là luật (weight = weighted lift).
- **Hành vi mua sắm**: Hiển thị clusters sản phẩm liên quan (e.g., tea sets, stationery), cho thấy hành vi mua combo trong phân khúc niche.
- **Điểm khác biệt Apriori/FP-Growth**: Không khác, nhưng FP-Growth hiệu quả hơn cho visualization trên dữ liệu lớn.

## Insights Kinh Doanh

### Từ Apriori (Truyền thống)
1. **Luật: WHITE HANGING HEART T-LIGHT HOLDER → RED HANGING HEART T-LIGHT HOLDER** (Support: 0.02, Confidence: 0.8, Lift: 40).
   - **Là quản lý**: Trưng bày chung các T-LIGHT HOLDER để tăng cross-selling, tạo section "Romantic Decor" với combo giảm giá.

2. **Luật: JUMBO BAG RED RETROSPOT → JUMBO BAG PINK POLKADOT** (Support: 0.015, Confidence: 0.75, Lift: 35).
   - **Là quản lý**: Gợi ý mua thêm qua app/website, điều chỉnh tồn kho theo mùa lễ (Christmas bags).

3. **Luật: REGENCY CAKESTAND 3 TIER → ROSES REGENCY TEACUP AND SAUCER** (Support: 0.01, Confidence: 0.85, Lift: 50).
   - **Là quản lý**: Tạo combo "Tea Party Set" với discount, quảng bá cho events gia đình.

### Từ FP-Growth (Tương tự Apriori, nhưng nhanh hơn)
1. **Luật: NATURAL SLATE HEART CHALKBOARD → NATURAL SLATE RECTANGLE CHALKBOARD** (Support: 0.012, Confidence: 0.82, Lift: 45).
   - **Là quản lý**: Push notification cho khách mua chalkboard, gợi ý mua thêm để hoàn thiện decor.

2. **Luật: LUNCH BAG RED RETROSPOT → LUNCH BAG PINK POLKADOT** (Support: 0.014, Confidence: 0.78, Lift: 38).
   - **Là quản lý**: Combo khuyến mãi cho lunch bags theo màu, tăng doanh thu mùa picnic.

3. **Luật: CHRISTMAS CRAFT LITTLE FRIENDS → CHRISTMAS CRAFT TREE TOP ANGEL** (Support: 0.011, Confidence: 0.8, Lift: 42).
   - **Là quản lý**: Section "Christmas Crafts" với bundles, điều chỉnh tồn kho theo mùa.

### Từ Weighted Rules (Niche)
1. **Luật: REGENCY TEA PLATE GREEN → REGENCY TEA PLATE ROSES** (Weighted Support: 0.056, Weighted Lift: 11.9).
   - **Là quản lý**: Phát triển chương trình loyalty cho khách VIP mua tea sets, gợi ý upgrades cao cấp.

2. **Luật: SMALL MARSHMALLOWS PINK BOWL → SMALL DOLLY MIX DESIGN ORANGE BOWL** (Weighted Support: 0.055, Weighted Lift: 11.5).
   - **Là quản lý**: Tạo "Kids Party Bundle" cho đơn hàng lớn, cross-selling với decor khác.

3. **Luật: FLORAL FOLK STATIONERY SET → MODERN FLORAL STATIONERY SET** (Weighted Support: 0.062, Weighted Lift: 9.1).
   - **Là quản lý**: Email personalized cho khách mua stationery, gợi ý full collection cho events.

### Từ HUIM (High-Utility)
1. **Itemset: NATURAL SLATE HEART CHALKBOARD** (Utility: 1.11e8).
   - **Là quản lý**: Ưu tiên quảng bá sản phẩm high-value này, dù ít bán, để tối đa lợi nhuận.

2. **Itemset: JUMBO BAG RED RETROSPOT** (Utility: 1.23e8).
   - **Là quản lý**: Tăng visibility cho bags này qua displays, focus vào khách hàng mua sắm lớn.

## So Sánh và Kết Luận

### So Sánh Thuật Toán
- **Apriori vs FP-Growth**: FP-Growth nhanh hơn đáng kể (3x) mà kết quả giống hệt. Khuyến nghị dùng FP-Growth cho scalability.
- **Truyền thống vs Weighted**: Weighted phát hiện niche rules quan trọng kinh tế, phù hợp VIP marketing; truyền thống tốt cho phổ biến.
- **Weighted vs HUIM**: HUIM tinh tế hơn, tối ưu lợi nhuận item; Weighted đơn giản hơn, dựa tổng hóa đơn.
- **Chung**: Tất cả đều giúp cross-selling và combos, nhưng weighted/HUIM phù hợp phân khúc cao cấp.

### Kết Luận
Dự án thành công mở rộng association rules với trọng số, phát hiện insights quý giá cho niche marketing. FP-Growth là lựa chọn hiệu quả. Áp dụng: Tăng doanh thu qua combos VIP, personalized recommendations. Tương lai: Tích hợp real-time, high-utility algorithms chuyên nghiệp.

---

*Báo cáo này tập trung vào câu chuyện dữ liệu, tránh dump code. Dữ liệu từ notebook `weighted_association_rules.ipynb`. Liên hệ: [Your Contact]*
