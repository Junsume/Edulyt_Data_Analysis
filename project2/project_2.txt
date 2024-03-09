SELECT * FROM project_2.credit_card_transactions limit 10;

-- 1. Top 5 cities with highest spends and their percentage contribution
SELECT City, SUM(Amount) AS TotalSpend, (SUM(Amount) / (SELECT SUM(Amount) FROM credit_card_transactions)) * 100 AS PercentageContribution
FROM credit_card_transactions
GROUP BY City
ORDER BY TotalSpend DESC
LIMIT 5;
SELECT `Card Type` FROM `project_2`.`credit_card_transactions`;

-- 2. Highest spend month and amount spent for each card type:
SELECT CardType, EXTRACT(MONTH FROM Date) AS Month, SUM(Amount) AS TotalSpent
FROM credit_card_transactions
GROUP BY CardType, Month
ORDER BY TotalSpent DESC
LIMIT 1;

-- 3. Transaction details for each card type when it reaches a cumulative of 1000000 total spends:
SELECT *
FROM (
    SELECT *, SUM(Amount) OVER (PARTITION BY CardType ORDER BY Date) AS CumulativeSpends
    FROM credit_card_transactions
) AS subquery
WHERE CumulativeSpends >= 1000000;

-- 4. City with the lowest percentage spend for gold card type:
SELECT City, (SUM(Amount) / (SELECT SUM(Amount) 
FROM credit_card_transactions WHERE CardType = 'Gold') * 100) as PercentageSpend
FROM credit_card_transactions
WHERE CardType = 'Gold'
GROUP BY City
ORDER BY PercentageSpend ASC
LIMIT 1;

-- 5. City, highest_expense_type, lowest_expense_type:
SELECT City, 
       MAX(ExpType) AS highest_expense_type, 
       MIN(ExpType) AS lowest_expense_type
FROM credit_card_transactions
GROUP BY City;

-- 6. Percentage contribution of spends by females for each expense type:
SELECT ExpType, 
       (SUM(CASE WHEN Gender = 'F' THEN Amount ELSE 0 END) * 100 / SUM(Amount)) AS PercentageContribution
FROM credit_card_transactions
GROUP BY ExpType;

-- 7. Card and expense type combination with the highest month over month growth in Jan-2014:
SELECT CardType, ExpType, MAX(MonthlyGrowth) AS MaxMonthlyGrowth
FROM (
    SELECT CardType, ExpType,
           MAX(CASE WHEN rn = 1 THEN NULL ELSE (Amount - lag_amount) / lag_amount END) AS MonthlyGrowth
    FROM (
        SELECT CardType, ExpType, Amount,
               ROW_NUMBER() OVER (PARTITION BY CardType, ExpType ORDER BY Date) AS rn,
               LAG(Amount) OVER (PARTITION BY CardType, ExpType ORDER BY Date) AS lag_amount
        FROM credit_card_transactions
        WHERE EXTRACT(MONTH FROM Date) = 1 AND EXTRACT(YEAR FROM Date) = 2014
    ) AS subquery
    GROUP BY CardType, ExpType, rn
) AS subquery_outer
GROUP BY CardType, ExpType;

-- 8. City with the highest total spend to total number of transactions ratio during weekends:
SELECT City, 
       SUM(Amount) / COUNT(*) AS SpendToTransactionRatio
FROM credit_card_transactions
WHERE EXTRACT(DAY FROM Date) IN (1, 7) -- Assuming 1 and 7 represent Saturday and Sunday
GROUP BY City
ORDER BY SpendToTransactionRatio DESC
LIMIT 1;

-- 9. City with the least number of days to reach its 500th transaction after the first transaction:
SELECT City,
       DATEDIFF(MIN(Date), MIN(FirstTransactionDate)) AS DaysTo500thTransaction
FROM (
    SELECT City,
           Date,
           ROW_NUMBER() OVER (PARTITION BY City ORDER BY Date) AS TransactionNumber,
           MIN(Date) OVER (PARTITION BY City) AS FirstTransactionDate
    FROM credit_card_transactions
) AS TransactionNumbered
WHERE TransactionNumber = 500
GROUP BY City
ORDER BY DaysTo500thTransaction ASC
LIMIT 1;