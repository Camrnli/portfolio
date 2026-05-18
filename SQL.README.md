SQL Queries

Case Study Queries


-- To Identify if there are any unknowns in rows 
		SELECT * 

		FROM TABLE

		WHERE [STOCK ITEMS] = 'Unknown' 


-- How many products are we dealing with? 
		SELECT

			COUNT(DISTINCT[Stock Items]) 

		FROM dim.StockItem 
		WHERE [Stock Items] <> 'Unknown'

-- What is the cheapest non-packaging-related product? (Utilizing % in practice)
		SELECT *

		FROM dimStockItem

		WHERE 
			[Stock Item] <> 'Unknown' 
			AND [Stock Item] NOT LIKE '%box%' 
			AND [Stock Item] NOT LIKE '%bag%' 
			AND [Stock Item] NOT LIKE '%carton%'

		ORDER BY 
			[Unit Price] 
	
-- the vice versa of this ^  (How many products contain mug or shirt in their name?)

		SELECT 

		COUNT(*)

		FROM dimStockItem

		WHERE
			[Stock Item] LIKE '%mug%' 
			OR [Stock Item] LIKE '%shirt%'


-- How many products w/ same conditions ^ are also black? (utilizing the AND & OR fx) 
-- SQL reads AND before OR hence why you need parenthesis for OR if we are including AND as well

		SELECT 

		COUNT(*)

		FROM dimStockItem

		WHERE
			([Stock Item] LIKE '%mug%' 
			OR [Stock Item] LIKE '%shirt%')
			AND Color = 'Black'



-- What was the markup on a specific product? (utilizing CAST fx) 
-- CAST() converts a value from one data type to another 
-- the 4 means how many digits after the decimal 
-- the 10 means total digits allow (both sides combined)

		SELECT

		CAST(
			([Recommended Retail Price]-[Unit Price]) / [Unit Price] AS decimal(10,4)
		 ) as Markup

		FROM dimStockItem

		WHERE [WWI Stock Item ID] = 29


-- How many customers are in each buying group?

		SELECT 

			[Buying Group],
			COUNT([Customer]) as CustomerCount

		FROM dimCustomer

		GROUP BY [Buying Group]

		ORDER BY 
			CustomerCount DESC

-- Do any postcodes have more than 3 Wingtip Toys shops? If so, which postcode? (Utilizing the HAVING clause)

		SELECT

		[Postal Code],
		COUNT([Customer Key]) as Count 

		FROM dimCustomer

		WHERE [Buying Group] = 'Wingtip Toys'

		GROUP BY [Postal Code]

		HAVING
			COUNT([Customer Key]) > 3     

-- If we wanted to get into detail of which stores pertain to the count ^ 

		SELECT *

		FROM dimCustomer

		WHERE 
			[Buying Group] = 'Wingtip Toys'
			AND [Postal Code] = 90683			
	
	

-- Which Sales Territory has the highest population? 

		SELECT

		[Sales Territory],
		SUM([Latest Recorded Population]) AS LastRecordedPopulation

		FROM dimCity

		WHERE [Sales Territory] <> 'N/A'

		GROUP BY [Sales Territory] 

		ORDER BY LastRecordedPopulation DESC

-- How many cities are in the above territory? 

		SELECT

		[Sales Territory],
		COUNT([City]) AS CityCount,
		SUM([Latest Recorded Population]) AS LastRecordedPopulation

		FROM dimCity

		GROUP BY [Sales Territory] 

		ORDER BY LastRecordedPopulation DESC

--What is the approx. population of the biggest city in that territory

		SELECT 

		[Sales Territory],
		City,
		MAX([Latest Recorded Population]) AS CityPopulation

		FROM dimCity

		WHERE [Sales Territory] = 'Southeast'

		GROUP BY [Sales Territory], City  

		ORDER BY CityPopulation DESC

-- What is the total population across all sales territories 

		SELECT  

		SUM([Latest Recorded Population]) as TotalPopulation

		FROM dimCity


-- What is the maximum fiscal year in dimdate? 

		SELECT

		MAX([Calendar Year]) as max 

		FROM dimDate

-- How many fiscal years do we have sales data for? 

		SELECT

			dd.[Calendar Year] as Year,
			SUM([Total Excluding Tax]) as TotalSales


		FROM factSale as fs
			LEFT JOIN dimDate as dd
			ON fs.[Invoice Date Key] = dd.[Date]

		GROUP BY
			dd.[Calendar Year]

		ORDER BY [Year] ASC
		
		
		

-- What were the sales excluding tax in fiscal year 2015 & what fiscal year had the highest profit? 

		SELECT

		d.[Fiscal Year] as FiscalYear,
		SUM([Total Excluding Tax]) as TotalSales,
		SUM([Quantity]) as Quantity,
		SUM([Profit]) as TotalProfit

		FROM factSale as fs
			LEFT JOIN dimDate as d
			ON fs.[Invoice Date Key] = d.[Date]

		GROUP BY d.[Fiscal Year]

		ORDER BY FiscalYear 

		
		
-- What explanation can you offer as to why the profit in 2016 is significantly lower than 2015?

		SELECT

		d.[Fiscal Year] as FY,
		d.[Fiscal Month Number] as MonthNo,
		d.[Fiscal Month Label] as Month,
		SUM(Quantity) as TotalQTY,
		AVG([Unit Price]) as ASP,
		SUM([Total Excluding Tax]) as TotalSales,
		SUM([Profit]) as Profit


		FROM factSale as fs
			LEFT JOIN dimDate as d
			ON fs.[Invoice Date Key] = d.[Date]

		WHERE d.[Fiscal Year] IN(2015,2016)

		GROUP BY 
			d.[Fiscal Year],
			d.[Fiscal Month Number],
			d.[Fiscal Month Label]

		ORDER BY
			FY,
    MonthNo


-- What was the total sales excluding tax in fiscal year 2016?

		SELECT

			SUM([Total Excluding Tax]) as TotalSales
			
		FROM factSale as fs
			INNER JOIN dimDate as d
			ON fs.[Invoice Date Key] = d.[Date]

		WHERE d.[Fiscal Year] = 2016

-- What was the top-selling product in 2016?
		SELECT TOP(5)

			[Stock Item] as Product,
			SUM([Total Excluding Tax]) as TotalSales


		FROM factSale as fs 
			INNER JOIN dimDate as d
				ON fs.[Invoice Date Key] = d.[Date]
			INNER JOIN dimStockItem as s 
				ON fs.[Stock Item Key] = s.[Stock Item Key]

		WHERE d.[Fiscal Year] = 2016

		GROUP BY [Stock Item]

		ORDER BY TotalSales DESC

-- What was the top-performing product/sales person combination in fiscal 2016?
		SELECT TOP(5)

			[Stock Item] as Product,
			[Employee] as Employee,
			SUM([Total Excluding Tax]) as TotalSales


		FROM factSale as fs 
			INNER JOIN dimDate as d
			ON fs.[Invoice Date Key] = d.[Date]
			INNER JOIN dimStockItem as s 
			ON fs.[Stock Item Key] = s.[Stock Item Key]
			INNER JOIN dimEmployee as e 
			ON fs.[Salesperson Key] = e.[Employee Key]

		WHERE d.[Fiscal Year] = 2016

		GROUP BY [Stock Item], e.Employee

		ORDER BY TotalSales DESC
   
-- Think about how you make your query reusable if the CY Changes. 

-- by adding YEAR(GETDATE()) into the where clause above ^^



-- How many chiller products have zero quantity sold to date? (utilzing left join w/ dim table first --> essentially "show all products, even unsold ones")

		SELECT

		[Stock Item] as Product,
		SUM(Quantity) as QuantitySold

		FROM dimStockItem as s 
			LEFT JOIN factSale as fs
			ON s.[Stock Item Key] = fs.[Stock Item Key]

		WHERE 
			s.[Is Chiller Stock] = 1 

		GROUP BY [Stock Item], [Total Chiller Items] 

		-- HAVING SUM(fs.Quantity) IS NULL

		ORDER BY QuantitySold DESC 
