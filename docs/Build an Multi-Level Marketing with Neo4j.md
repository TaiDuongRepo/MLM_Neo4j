# Build an Multi-Level Marketing with Neo4j

Tag: Graph, University
URL: https://github.com/TaiDuongRepo/MLM_Neo4j

# What Is Multi-Level Marketing?

Multi-level marketing (MLM), hay network selling, còn gọi là đa cấp là một chiến lược để tối đa hóa doanh số bán hàng thông qua mạng lưới các nhà phân phối. Các nhà phân phối được trả hoa hồng cho việc tự bán hàng và việc những cộng tác viên mà họ tuyển dụng bán hàng. Một hệ thống phân cấp trong đó các cộng tác viên được đặt dưới các nhà phân phối hay đại lý cấp trên tuyển dụng họ và các cấp càng cao sẽ được trả lương càng cao. Do đó, một người tham gia càng sớm, tuyến dưới của họ và hoa hồng tiềm năng sẽ càng lớn.

# MLMs Are Graphs

Trong MLM, ta xây dựng, nhà phân phối (đại lý) là các node, cộng tác viên, tân binh, sponsor, các nhà phân phối khác là các edge. Trong Neo4j, các edge được gọi là relationship.

# Multi-Level Marketing in World of Warcarft using Neo4j

Lập kế hoạch và tính toán bồi thường trong các tổ chức bán hàng phức tạp có thể bị đánh thuế vô cùng lớn đối với các cơ sở dữ liệu truyền thống. Các tổ chức đủ lớn thường sẽ kết thúc việc trộn các quy trình này qua đêm hoặc thành một công việc hàng tuần.

Slashco: the hottest multi-level marketing “guild” in the World of Warcarft.

Trong bang hội này, "associates" có thể tham gia và bán “advanturing gear” tuyệt vời của Slashco cho nhiều người ở Azeroth cũng như tuyển dụng bạn bè và gia đình của họ để bán các sản phẩm Slashco. Khi một “associate” tuyển dụng ai đó tham gia bang hội, thành viên mới đó sẽ trở thành một phần trong "hạ lưu"(down-stream) của “associate” đó. Các “associate” sẽ nhận được một khoản hoa hồng nhỏ cho mỗi mặt hàng được bán bởi các thành viên "down-stream" của họ.

Bán hàng trực tiếp: Đối với mỗi mặt hàng mà một thành viên trong nhóm bán trực tiếp, họ kiếm được hoa hồng.

Tiền bản quyền hạ nguồn: Tỷ lệ phần trăm kiếm được từ giá bán lẻ của các mặt hàng được bán bởi những người "trong hạ lưu của họ", có nghĩa là những người mà họ đã tuyển dụng để làm việc cho Slashco và mở rộng là những người mà tuyển dụng của họ đã tuyển dụng.

Lợi nhuận bán buôn: Một tỷ lệ phần trăm kiếm được từ hàng hóa được bán cho những người "trong hạ lưu của họ" (những hàng hóa này sau đó sẽ được bán lại cho người tiêu dùng.

Tiền bản quyền bán hàng toàn cầu: Một tỷ lệ phần trăm của tất cả doanh số bán hàng được thực hiện bởi những người trong Slashco (còa là chia sẻ doanh thu).

Mô hình mẫu dựa trên MLM:

- **Direct Sales**: đối với mỗi item mà một thành viên trong nhóm bán trực tiếp, họ kiếm được hoa hồng.
- **Downstream Royalty**: Tỷ lệ phần trăm kiếm được từ gía bán lẻ của các mặt hàng được bán bởi những người “within their down-stream”.
- **Wholesale Profit**: Một tỷ lệ phần trăm kiếm được từ hàng hóa được bán cho những người "within their down-stream" (những hàng hóa này sau đó sẽ được bán lại cho người tiêu dùng.
- **Global Sales Royalty**: Một tỷ lệ phần trăm của tất cả doanh số bán hàng được thực hiện bởi những người trong Slashco (a.k.a chia sẻ doanh thu (revenue sharing)).

Ví dụ MLM:

![image](http://i.imgur.com/ODDXeKb.png)

Chúng tôi thấy rằng, tùy thuộc vào cấp độ bạn đang ở, cách bạn được bồi thường cho cả doanh số bán hàng trực tiếp và cả các dòng thu nhập thụ động của bạn có thể khác nhau rất nhiều. Như vậy, việc tính toán bồi thường có thể ít nhất là khó khăn.

## Giới thiệu bộ dữ liệu

Các dữ liệu được lưu trong các file `.csv` .

- `employees.csv`
  - Dữ liệu có các trường dữ liệu: employeeID, name, reportsTo.
- `item.csv`
  - Dữ liệu có các trường: item, factor, name, price, wprice, kicker.
- `transactions.csv`
  - Dữ liệu có các trường: transactionID, salesRepID, item1, item2, item3, period.

Trước khi tải bất kỳ dữ liệu nào vào Neo4j, điều quan trọng là phải biết những câu hỏi nào sẽ quan trọng đối với doanh nghiệp. Trong trường hợp này, mô hình hóa một “sales compensation tool”. Các truy vấn hàng đầu có thể là:

- Hoa hồng do mỗi đại diện (rep), theo thời gian (period), phù hợp với compensation rules
- Hoa hồng do mỗi đại diện (rep), hàng năm phù hợp với các quy tắc bồi thường
- Sales Leaderboard
  - Top Sales Rep by Total Sales
  - Top Sales Rep by Largest Deal
- Global reporting
  - Sales by period
  - Top selling items
- Recommendations
  - Những mặt hàng nào được bán cùng nhau thường xuyên nhất?

# Slaschco Sales Compensation Application Data Model:

![](http://i.imgur.com/2y4MfIX.png)
