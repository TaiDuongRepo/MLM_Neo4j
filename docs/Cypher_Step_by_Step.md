# Import data from CSV Step-by-Step

## Giới thiệu bộ dữ liệu

Các dữ liệu được lưu trong các file `.csv` .

- `employees.csv`
  - Dữ liệu có các trường dữ liệu: employeeID, name, reportsTo.
- `item.csv`
  - Dữ liệu có các trường: item, factor, name, price, wprice, kicker.
- `transactions.csv`
  - Dữ liệu có các trường: transactionID, salesRepID, item1, item2, item3, period.

## Import dữ liệu

### Step 1: Prepare

Trước khi nhập dữ liệu, bạn nên chuẩn bị cơ sở dữ liệu bạn muốn sử dụng bằng cách tạo các chỉ mục và ràng buộc.

Bạn nên đảm bảo rằng các nút `Person`, `Item`, `Transaction`, và `Period` có các thuộc tính id duy nhất bằng cách tạo các ràng buộc đối với chúng.

Tạo một ràng buộc duy nhất cũng ngầm tạo ra một chỉ mục. Bằng cách lập chỉ mục thuộc tính `id`, node lookup (ví dụ: bằng MATCH) sẽ nhanh hơn nhiều.

- Bạn tạo một ràng buộc trên thuộc tính `employeeID` của các nút `Person` để đảm bảo rằng các nút có nhãn `Person` sẽ có thuộc tính `employeeID` duy nhất.

```cypher
CREATE CONSTRAINT employeeIDConstraint FOR (p:Person) REQUIRE p.employeeID IS UNIQUE
```

- Tạo ràng buộc trên thuộc tính `itemID` của `Item` node.

```cypher
CREATE CONSTRAINT itemIDConstraint FOR (i:Item) REQUIRE i.itemID IS UNIQUE
```

- Tương tự cho các nút `Transaction` và `Period`.

```cypher
CREATE CONSTRAINT transactionIDConstraint FOR (t:Transaction) REQUIRE t.transactionID IS UNIQUE;
CREATE CONSTRAINT periodContraint FOR (p:Period) REQUIRE p.period IS UNIQUE;
```

- Tạo một index trên thuộc tính `price` của `Item` node.

```cypher
CREATE INDEX FOR (i:Item) ON (i.price);
CREATE INDEX FOR (i:Item) ON (i.wholesalePrice);
```

- Tạo 52 nút `Period` để biểu diễn 52 tuần trong năm.

```cypher
WITH range(1,52) as periods
FOREACH (period IN periods |
	MERGE (p:Period {period:period}))
```

- Tạo quan hệ `NEXT` giữa các nút `Period` để biểu diễn thứ tự tuần trong năm.

```cypher
MATCH (p:Period)
WITH p
ORDER BY p.period
WITH COLLECT(p) as periods
FOREACH (i in RANGE(0, size(periods)-2) |
	FOREACH (p1 in [periods[i]] |
		FOREACH (p2 in [periods[i+1]] |
			MERGE (p1)-[:NEXT]->(p2))))
```

### Step 2: Import Data

- Tải dữ liệu từ các file `.csv` vào Neo4j. Đồng thời tạo các nút và quan hệ tương ứng.

```cypher
//
LOAD CSV WITH HEADERS FROM "https://raw.githubusercontent.com/TaiDuongRepo/MLM_Neo4j/main/data/item.csv" as line
WITH line, toFLOAT(line.price) as price, toInteger(line.item) as itemID, toFLOAT(line.kicker) as kick, toFLOAT(line.wprice) as wholesale
CREATE (:Item {itemID:itemID, name:line.name, price:price, kicker:kick, wholesalePrice:wholesale});
//
LOAD CSV WITH HEADERS FROM "https://raw.githubusercontent.com/TaiDuongRepo/MLM_Neo4j/main/data/employees.csv" as line
WITH line, toInteger(line.employeeID) as empID
CREATE (:Person {employeeID:empID, name:line.name});
//
LOAD CSV WITH HEADERS FROM "https://raw.githubusercontent.com/TaiDuongRepo/MLM_Neo4j/main/data/employees.csv" as line
WITH line, toInteger(line.employeeID) as empID, toInteger(line.reportsTo) as reportsToID
MATCH (sub:Person {employeeID:empID}), (boss:Person {employeeID:reportsToID})
MERGE (sub)-[:REPORTS_TO]->(boss);
//
LOAD CSV WITH HEADERS FROM "https://raw.githubusercontent.com/TaiDuongRepo/MLM_Neo4j/main/data/transactions.csv" as line
WITH line, toInteger(line.transactionID) as transID
CREATE (:Transaction {transactionID:transID});
//
LOAD CSV WITH HEADERS FROM "https://raw.githubusercontent.com/TaiDuongRepo/MLM_Neo4j/main/data/transactions.csv" as line
WITH line, toInteger(line.transactionID) as transID, toInteger(line.period) as period
MATCH (t:Transaction {transactionID:transID}), (p:Period {period:period})
CREATE (t)-[:OCCURRED_IN]->(p);
//
LOAD CSV WITH HEADERS FROM "https://raw.githubusercontent.com/TaiDuongRepo/MLM_Neo4j/main/data/transactions.csv" as line
WITH line,
toInteger(line.transactionID) as transID,
toInteger(line.item1) as itemID1,
toInteger(line.item2) as itemID2,
toInteger(line.item3) as itemID3
MATCH
(tx:Transaction {transactionID:transID}),
(i1:Item {itemID:itemID1}),
(i2:Item {itemID:itemID2}),
(i3:Item {itemID:itemID3})
CREATE
(tx)-[:CONTAINS]->(i1),
(tx)-[:CONTAINS]->(i2),
(tx)-[:CONTAINS]->(i3);
//
LOAD CSV WITH HEADERS FROM "https://raw.githubusercontent.com/TaiDuongRepo/MLM_Neo4j/main/data/transactions.csv" as line
WITH line,
toInteger(line.transactionID) as transID,
toInteger(line.salesRepID) as repID
MATCH (rep:Person {employeeID:repID}),
(tx:Transaction {transactionID:transID})
CREATE
(rep)-[:SOLD]->(tx);
```

### Step 3: Tạo cấp bậc cho các cộng tác viên

- Tạo level cho các cộng tác viên, dựa trên số lượng cấp dưới mà họ có.

```cypher
MATCH (target:Person)<-[r:REPORTS_TO*..]-(e)
WITH target, count(e) as totalReports
SET target.reportsCount = totalReports
WITH target,
//setting the right "level" based on number of reports
CASE
WHEN target.reportsCount > 124
THEN 6
WHEN target.reportsCount < 124 and target.reportsCount >= 75
THEN 5
WHEN target.reportsCount < 75 and target.reportsCount >= 25
THEN 4
WHEN target.reportsCount < 25 and target.reportsCount >= 10
THEN 3
WHEN target.reportsCount < 10 and target.reportsCount >= 2
THEN 2
ELSE 1
END AS levels
SET target.level = levels;
```
