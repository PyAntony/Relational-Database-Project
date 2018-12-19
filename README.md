# Relational Database Project

Final project for my Database Management Systems class. Project consisted of designing and implementing a database according to a produced business case. An overview of the sections will be provided here. The complete implementation can be found in the **DataBase_Project.pdf** document. Complete SQL code to create and populate tables can be found in the **Dbscript.txt.pdf** document.

## Proposal

Ginseng Vitamins, a small company that has been selling dietary supplements for 5 years, is
trying to carry out an ambitious plan to expand on the East Coast. Ginseng Vitamins has been able to
attract the attention of new investors and raise enough capital to begin implementation. Our
company, Data Pro, has been contracted to design and implement a new database, one that could
capture all the details and requirements for the new business model they are trying to implement.
After several meetings with management and owners we have been able to understand their current
position and all the requirements needed for the new stores...

##  Entity Relational Diagram

![pic1](https://github.com/PyAntony/Relational-database-Project/blob/master/images/pic1.png)

##  Relational Model

1) Doctor (docSSN, docFname, docLname, phone, docSpecialy)
PK ={docSSN}

2) Time_Alloc (allocNum, docSSN, storeID, date, hours)
PK ={allocNum}
FK1={docSSN} ref PK of Doctor
FK2={storeID} ref PK of Store

3) Store (storeID, street, city, state, zip)
PK ={storeID}

4) Staff (stID, stFname, stLname, position, storeID)
PK = {stID}
FK2={storeID} ref PK of Store

5) Membership (storeID, custNum, regDate)
PK ={storeID, custNum}
FK1={storeID} ref PK of Store
FK2={custNum} ref PK of Customer

6) Customer (custNum, custFname, custLname, custPhone, custDOB, level)
PK ={custNum}

7) Supp_Unit (suppCode, storeID, quantity)
PK ={suppCode, storeID}
FK1={suppCode} ref PK of Supp_List
FK2={storeID} ref PK of Store

8) Supp_List (suppCode, suppName, categNum, brand, price, sNum)
PK ={suppCode}
FK1={categNum} ref PK of Category
FK2={sNum} ref PK of Scientist

9) Category (categNum, categName)
PK ={categNum}

10) Scientist (sNum, sFname, sLname, specialty)
PK ={sNum}11) Sales_Info (SaleNum, suppCode, storeID, custNum, qty, Sdate, rating)
PK ={SaleNum}
FK1={suppCode, storeID} ref PK of Supp_Unit
FK2={custNum} ref PK of Customer

## Queries

1) qryAverageRatingByStore: Customer satisfaction is collected at point of sale (scale from 1 to 5).
This query helps to identify the stores with highest and lowest customer satisfaction throughout the
year. Many Strategies can be implemented to reward or help stores according to customer
satisfaction.
```
SELECT st.storeid, st.street, st.city, state, Round(Avg(rating),1) AS Rating_Average
FROM store AS st, membership AS m, customer AS c, sales_info AS s
WHERE (st.storeid)=[m].[storeid] AND (c.custnum)=[m].[custnum] And (c.custnum)=[s].[custnum]
AND Year([s.sdate]) = Year(Date())
GROUP BY st.storeid, st.street, st.city, state
ORDER BY Round(Avg(rating),1) DESC;
```

2) qryCategListByStore: list of categories that are currently displayed in stores.
```
SELECT DISTINCT store.storeid, street, city, category.categname
FROM category, supp_list, supp_unit, store
WHERE category.categnum = supp_list.categnum AND supp_list.suppcode = supp_unit.suppcode
AND supp_unit.storeID = store.storeID;
```

3) qryCustomerDisc%Parameter: this query asks the user to enter the discount level (as a
parameter). Level 0 = no discount, level 1 = 25% discount, and level 2 = 35% discount (as explained
in the proposal). All customers with the selected parameter will show up in the report.
```
SELECT m.custnum, c.custfname, c.custlname, m.storeid, s.street, s.city, m.regdate, c.level
FROM store AS s, membership AS m, customer AS c
WHERE (((m.custnum)=c.custnum) And ((c.level)=[Enter level (0,1,2):]) And ((s.storeID)=m.storeid));
```


10) qryYearSalesByCustomer: total sales by customers for the current year.
```
SELECT c.custnum, c.custfname, c.custlname, FORMAT(Sum((spl.price*sales.qty)),"currency") AS
Total
FROM sales_info AS sales, supp_list AS spl, supp_unit, customer AS c
WHERE (sales.suppcode)=supp_unit.suppcode And ((supp_unit.suppcode)=spl.suppcode) And
((c.custnum)=sales.custnum) And ((sales.storeID)=supp_unit.storeID) And
(Year([sales.sdate])=Year(Date()))
GROUP BY c.custnum, c.custfname, c.custlname
ORDER BY Sum(spl.price*sales.qty) DESC;
```

12) qryYearSalesBySupplement: total sales by supplements in the current year.
```
SELECT s.storeID, s.street, s.city, s.state, FORMAT((ROUND(Sum(spl.price*sales.qty),2)),"currency")
AS Total_Sales
FROM store AS s, membership AS m, customer AS c, sales_info AS sales, supp_unit AS supu, supp_list
AS spl
WHERE (((s.storeId)=[m].[storeid]) AND ((m.custnum)=[c].[custnum]) AND
((c.custnum)=[sales].[custnum]) AND ((sales.suppcode)=[supu].[suppcode]) AND
((supu.suppcode)=[spl].[suppcode])) AND ((sales.storeID)=[supu].[storeID]) AND Year([sales.sdate]) =
Year(Date())
GROUP BY s.storeID, s.street, s.city, s.state
ORDER BY (Sum(spl.price*sales.qty)) DESC;
```

13) qryYearTotalDiscount: total discounts given to customers in the current year.
```
SELECT qt.custnum, qt.custfname, qt.custlname, qt.custDOB, qt.total_Disc_Given
FROM (SELECT c.custnum, c.custfname, c.custlname, c.custDOB,
Format((Round(Sum(spl.price*sales.qty),2)),"Currency") AS Total_Sales,
IIf([c].[level]='2',0.35,IIf([c].[level]='1',0.25,0)) AS [Discount%],
FORMAT(((Format((Round(Sum(spl.price*sales.qty),2)),"Currency")) *
(IIf([c].[level]='2',0.35,IIf([c].[level]='1',0.25,0)))),"Currency") AS total_Disc_Given FROM store AS s,
membership AS m, customer AS c, sales_info AS sales, supp_unit AS supu, supp_list AS spl WHERE
(((s.storeId)=m.storeid) And ((m.custnum)=c.custnum) And ((c.custnum)=sales.custnum) And
((sales.suppcode)=supu.suppcode) And ((supu.suppcode)=spl.suppcode) And
((sales.storeID)=supu.storeID) And ((Year([sales.sdate]))=Year(Date()))) GROUP BY c.custnum,
c.custfname, c.custlname, c.custDOB, IIf([c].[level]='2',0.35,IIf([c].[level]='1',0.25,0))) AS qt
WHERE total_Disc_Given > 0
ORDER BY total_Disc_Given DESC;
```




## Login Screen

![pic2](https://github.com/PyAntony/Relational-database-Project/blob/master/images/pic2.png)
![pic3](https://github.com/PyAntony/Relational-database-Project/blob/master/images/pic3.png)
