# Sample Queries

Sau khi đã tải thành công dữ liệu mẫu vào Neo4j, bạn có thể thực hiện các truy vấn sau để thử nghiệm:

## 1. Recommendation

- Những mặt hàng nào được bán cùng nhau thường xuyên nhất?

```cypher
MATCH path = (item:Item)-[:CONTAINS]-(:Transaction)-[:CONTAINS]-(item2:Item)
WHERE id(item) > id(item2)
WITH item, item2, count(distinct path) as instances
ORDER BY instances DESC
LIMIT 3
RETURN item.name, item2.name, instances;
```

## 2. Global reporting

- Doanh số bán hàng theo kỳ

```cypher
//total sales volume by period descending
MATCH (p:Period)-[:OCCURRED_IN]-(t:Transaction)-[:CONTAINS]-(i:Item)
WITH sum(i.price) as sales, p
ORDER BY sales DESC
LIMIT 10
RETURN sales, p.period;
```

- Các mặt hàng bán chạy nhất

```cypher
MATCH (t:Transaction)-[:CONTAINS]-(i:Item)
WITH count(distinct(t)) as itemSales, i
ORDER BY itemSales DESC
LIMIT 5
RETURN i.name as name, itemSales as count;
```

## 3. Sales Leaderboad

- Đại diện bán hàng hàng đầu theo tổng khối lượng bán hàng

```cypher
//Ai đã bán được nhiều nhất?
MATCH (rep)-[:SOLD]-(txn)-[:CONTAINS]-(itm)
WITH rep, round(sum(itm.price)) as volume
ORDER BY volume DESC
LIMIT 5
RETURN rep.name as name, volume;
```

- Ai đã chốt giao dịch lớn nhất?

```cypher
MATCH (rep)-[:SOLD]-(txn)
WITH rep, txn
MATCH (txn)-[:CONTAINS]-(itm)
WITH rep, txn, round(sum(itm.price)) as dealSize
ORDER BY dealSize DESC
LIMIT 5
RETURN rep.name as name, txn.transactionID as transction, dealSize as `deal size`;
```

## 4. Commission due to each rep, annually in accord with compensation rules

- Tổng số tiền bồi thường mà mỗi đại diện sẽ nhận được theo level

```cypher
//level 1 comp
MATCH (distributor:Person {level:1})-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.25) as tc1, distributor.name as n1
RETURN tc1, n1;
//level 2 comp
MATCH (success_builder:Person {level:2})<-[r:REPORTS_TO*..]-(downStreamers)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.05) as downStream2, success_builder
MATCH (success_builder)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.25) + downStream2 as tc2, success_builder.name as n2, downStream2
RETURN tc2, n2;
//level 3 comp
MATCH (senior_mage:Person {level:3})<-[r:REPORTS_TO*..]-(downStreamers)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.05)+sum(item.wholesalePrice*.25) as downStream3, senior_mage
MATCH (senior_mage)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.5) + downStream3 as tc3, senior_mage.name as n3, downStream3
RETURN tc3, n3;
//level 4 comp
MATCH (transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.01) as globalRoyalty
MATCH (guild_leader:Person {level:4})<-[r:REPORTS_TO*..]-(downStreamers)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.07)+sum(item.wholesalePrice*.3) + globalRoyalty as downStreamGlobal4, guild_leader, globalRoyalty
MATCH (guild_leader)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.5) + downStreamGlobal4 as tc4, guild_leader.name as n4, downStreamGlobal4
RETURN tc4, n4;
//level 5 comp
MATCH (transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.02) as globalRoyalty
MATCH (boss:Person {level:5})<-[r:REPORTS_TO*..]-(downStreamers)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.07)+sum(item.wholesalePrice*.3) + globalRoyalty as downStreamGlobal5, boss, globalRoyalty
MATCH (boss)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.5) + downStreamGlobal5 as tc5, boss.name as n5, downStreamGlobal5
RETURN tc5, n5;
//level 6 comp
MATCH (transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.05) as globalRoyalty
MATCH (big_boss:Person {level:6})<-[r:REPORTS_TO*..]-(downStreamers)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.1)+sum(item.wholesalePrice*.5) + globalRoyalty as downStreamGlobal6, big_boss, globalRoyalty
MATCH (boss)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.65) + downStreamGlobal6 as tc6, big_boss.name as n6, downStreamGlobal6
RETURN tc6, n6;
```

## 5. Commission due to each rep, by period in accord with compensation rules

Hoa hồng do mỗi đại diện, theo khoảng thời gian phù hợp với các quy tắc bồi thường

- Ví dụ cho đại diện cấp 6

```cypher
//level 6 comp with time period
MATCH (transaction)-[:OCCURRED_IN]-(p:Period {period:35})
WITH transaction, p
MATCH (transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.05) as globalRoyalty, p
MATCH (transaction)-[:OCCURRED_IN]-(p:Period {period:35})
WITH globalRoyalty, p, transaction
MATCH (big_boss:Person {level:6})<-[r:REPORTS_TO*..]-(downStreamers)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.1)+sum(item.wholesalePrice*.5) + globalRoyalty as downStreamGlobal6, big_boss, p, globalRoyalty
MATCH (transaction)-[:OCCURRED_IN]-(p:Period {period:35})
WITH transaction, downStreamGlobal6, big_boss
MATCH (boss)-[:SOLD]-(transaction)-[:CONTAINS]-(item)
WITH sum(item.price*.65) + downStreamGlobal6 as tc6, big_boss.name as n6, downStreamGlobal6
RETURN tc6, n6;
```
