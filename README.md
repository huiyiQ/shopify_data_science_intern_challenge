# Shopify Winter 2022 Data Science Intern Challenge
## Question 1 Answers:
The naive calculation of AOV = $3145.13 could be going wrong because different shops can have very different performance, and therefore the data may be skewed, causing a simple mean to come out higher than expected. Going from there, after analyzing the average order value for each shop, we can see that shop #42 and shop #78 had a much higher AOV per shop; the former is potentially affected by credit card fraud while the latter simply has a much higher unit price for its sneakers. For more indepth analysis, please see the notebook in this repo. A better way to evaluate this data would be to use something that's less sensitive to outliers, such as the median or IQR.

For this dataset, I'd like to report the IQR.
```python
q1 = df.order_amount.quantile(0.25)
q3 = df.order_amount.quantile(0.75)
IQR = q3 - q1
```
And the value of IQR of order amount is $227.

---
## Question 2 Answers:
### a. How many orders were shipped by Speedy Express in total?

**steps:**
- filter ShipperName = 'Speedy Express'
- count OrderID (which is pk so no need for distinct)

**query:**
```sql
SELECT COUNT(OrderID) as num_orders
FROM Orders o
WHERE ShipperID in (SELECT ShipperID FROM Shippers WHERE ShipperName = 'Speedy Express');
```

**answer: 54**

### b. What is the last name of the employee with the most orders?

**steps:**
- get EmployeeID and their respective OrderID counts
- find EmployeeID with max OrderID counts
- get LastName

**query:**
```sql
WITH t AS (
  SELECT EmployeeID, COUNT(OrderID) as num_orders
  FROM Orders
  GROUP BY EmployeeID)

SELECT LastName
FROM Employees
WHERE EmployeeID IN (SELECT EmployeeID FROM t WHERE num_orders = (SELECT MAX(num_orders) FROM t))
```

**answer: Peacock**

### c. What product was ordered the most by customers in Germany?

**steps:**
- filter CustomerID from Germany
- filter these customers' OrderID
- using these OrderID's, in OrderDetails table, get ProductID and their respective counts
- find ProductID with max counts
- get ProductName

**query:**
```sql
WITH t AS (
  SELECT od.ProductID, COUNT(*) as num
  FROM Orders o
  JOIN OrderDetails od
  ON o.OrderID = od.OrderID
  WHERE o.CustomerID IN (SELECT CustomerID FROM Customers WHERE Country = 'Germany')
  GROUP BY 1
  ORDER BY 2 DESC)

SELECT ProductName
FROM Products
WHERE ProductID in (SELECT ProductID FROM t WHERE num = (SELECT MAX(num) FROM t));
```

**answer: Gorgonzola Telino**
