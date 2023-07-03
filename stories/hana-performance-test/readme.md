# Hana Modeling Performance Report

## Objective

To learn the performance difference between two different table designs in the HANA database. We expect it further implies the performance of tables in SAC.

1) Column tables with hierarchies, i.e., parent-child tables.
2) Column tables with flattened & normalized attributes.

## Data Preparation

### Modeling

```mermaid
classDiagram
	TableS <|-- TableCr
	TableS <|-- TableLogs
	TableS <|-- TableSnap
	TableSnap <|-- TableI
	TableI <|-- TableIcr

	TableLogs : LogId
	TableLogs : (FK) S-ID
	TableSnap : SNAP-ID
	TableSnap : (FK) S-ID
	TableCr : CR-ID
	TableCr : CR-Code
	TableCr : (FK) S-ID
	TableIcr : ICR-ID
	TableIcr : ICR-Code
	TableIcr : (FK) I-ID
	TableS : S-ID
	TableS : ...
	TableI : I-ID
	TableI : P-ID
	TableI : (FK) SNAP-ID
```

### Mock Data

An API and a SQL script are implemented to create two kinds of insertion in two different models. The service is deployed on `dev-eu10`.


<details>
    <summary>SQL Script for mocking data</summary>

```SQL

DO
BEGIN
	DECLARE COUNT BIGINT = 769065;

	WHILE :COUNT < 800000 DO
		
		-- subscription def
		DECLARE SUBSCRIPTION_ID varchar(255);
		DECLARE DOCUMENT_NUMBER varchar(255);
	
		DECLARE STATUSLIST NVARCHAR(20) ARRAY = ARRAY('Status1', 'Not started', 'Active', 'Not Applicable', 'Status2');
		DECLARE MARKET_PREFIX_LIST NVARCHAR(20) ARRAY = ARRAY('A', 'B', 'C', 'D', 'E');
	
	
		-- snapshot def
		DECLARE SNAPSHOT_ID varchar(255);
	
		-- item def
		DECLARE ITEM_ID_1 varchar(255);
		DECLARE ITEM_ID_2 varchar(255);
		DECLARE ITEM_ID_3 varchar(255);
		DECLARE ITEM_ID_4 varchar(255);
	
		SELECT NEWUID() INTO SUBSCRIPTION_ID FROM DUMMY;
		SELECT NEWUID() INTO SNAPSHOT_ID FROM DUMMY;
		SELECT NEWUID() INTO ITEM_ID_1 FROM DUMMY;
		SELECT NEWUID() INTO ITEM_ID_2 FROM DUMMY;
		SELECT NEWUID() INTO ITEM_ID_3 FROM DUMMY;
		SELECT NEWUID() INTO ITEM_ID_4 FROM DUMMY;
	
		SELECT :COUNT INTO DOCUMENT_NUMBER FROM DUMMY;
		
		-- ====== Insert in to subscription table
		INSERT INTO SAP_CAPIRE_SACPOC_SUBSCRIPTION 
			(SUBSCRIPTIONID, VERSION, DOCUMENTNUMBER, SUBSCRIPTIONDOCUMENTID, STATUS, CUSTOMERID, MARKETID, 
			VALIDFROM, VALIDUNTIL, 
			CHANGEDAT, CHANGEDBY, CREATEDAT, CREATEDBY, CANCELLATIONREASON)
		VALUES
			(:SUBSCRIPTION_ID, 0, :COUNT, CONCAT('SB', :COUNT), :STATUSLIST[FLOOR(1+rand()*5)], CONCAT('C', FLOOR(rand()*100)), CONCAT(:MARKET_PREFIX_LIST[FLOOR(1+rand()*5)], FLOOR(1+rand()*5)), 
			ADD_MONTHS(CURRENT_UTCTIMESTAMP, -(FLOOR(rand()*3))), ADD_MONTHS(CURRENT_UTCTIMESTAMP, FLOOR(rand()*10)), -- validfrom, validuntil
			CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', 'JUST_CANCELLED');
		
		-- ====== Insert into CR table
		-- CR 1
		INSERT INTO SAP_CAPIRE_SACPOC_CUSTOMREFERENCE 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ID, TYPECODE, SUBSCRIPTION_SUBSCRIPTIONID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', CONCAT('cr-id-', FLOOR(rand() * 1000)), CONCAT('typecode-', FLOOR(rand() * 500)), :SUBSCRIPTION_ID);
		-- CR 2
		INSERT INTO SAP_CAPIRE_SACPOC_CUSTOMREFERENCE 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ID, TYPECODE, SUBSCRIPTION_SUBSCRIPTIONID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', CONCAT('cr-id-', FLOOR(rand() * 1000)), CONCAT('typecode-', FLOOR(rand() * 500)), :SUBSCRIPTION_ID);
		-- CR 3
		INSERT INTO SAP_CAPIRE_SACPOC_CUSTOMREFERENCE 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ID, TYPECODE, SUBSCRIPTION_SUBSCRIPTIONID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', CONCAT('cr-id-', FLOOR(rand() * 1000)), CONCAT('typecode-', FLOOR(rand() * 500)), :SUBSCRIPTION_ID);
		-- CR 4
		INSERT INTO SAP_CAPIRE_SACPOC_CUSTOMREFERENCE 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ID, TYPECODE, SUBSCRIPTION_SUBSCRIPTIONID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', CONCAT('cr-id-', FLOOR(rand() * 1000)), CONCAT('typecode-', FLOOR(rand() * 500)), :SUBSCRIPTION_ID);
		
		-- ====== Insert into Eventlog table
		-- eventlog 1
		INSERT INTO SAP_CAPIRE_SACPOC_EVENTLOGENTRY 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ID, EVENTTYPE, SEQUENCENUMBER, SUBSCRIPTION_SUBSCRIPTIONID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', NEWUID(), 'Create', FLOOR(rand()*100), :SUBSCRIPTION_ID);
		-- event log 2
		INSERT INTO SAP_CAPIRE_SACPOC_EVENTLOGENTRY 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ID, EVENTTYPE, SEQUENCENUMBER, SUBSCRIPTION_SUBSCRIPTIONID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', NEWUID(), 'Update', FLOOR(rand()*100), :SUBSCRIPTION_ID);
		
		-- ===== Insert into snapshot table
		INSERT INTO SAP_CAPIRE_SACPOC_SNAPSHOT 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, SNAPSHOTID, EFFECTIVEDATE, SUBSCRIPTION_SUBSCRIPTIONID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', :SNAPSHOT_ID, ADD_DAYS(CURRENT_UTCTIMESTAMP, -FLOOR(rand()*10)), :SUBSCRIPTION_ID);
		
		-- ===== Insert into item table
		-- Item 1
		INSERT INTO SAP_CAPIRE_SACPOC_ITEM 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ITEMID, PRODUCTID, SNAPSHOT_SNAPSHOTID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', :ITEM_ID_1, FLOOR(rand()*1000), :SNAPSHOT_ID);
		--Item 2
		INSERT INTO SAP_CAPIRE_SACPOC_ITEM 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ITEMID, PRODUCTID, SNAPSHOT_SNAPSHOTID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', :ITEM_ID_2, FLOOR(rand()*1000), :SNAPSHOT_ID);
		--Item 3
		INSERT INTO SAP_CAPIRE_SACPOC_ITEM 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ITEMID, PRODUCTID, SNAPSHOT_SNAPSHOTID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', :ITEM_ID_3, FLOOR(rand()*1000), :SNAPSHOT_ID);
		--Item 4
		INSERT INTO SAP_CAPIRE_SACPOC_ITEM 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ITEMID, PRODUCTID, SNAPSHOT_SNAPSHOTID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', :ITEM_ID_4, FLOOR(rand()*1000), :SNAPSHOT_ID);
	
		-- ===== Insert into icr table
		-- icr 3-1
		INSERT INTO SAP_CAPIRE_SACPOC_ITEMCUSTOMREFERENCE 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ID, TYPECODE, ITEM_ITEMID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', CONCAT('icr-id-', FLOOR(rand()*1000)) , CONCAT('icr-typecode', FLOOR(rand()*500)), ITEM_ID_3);
		-- icr 3-2
		INSERT INTO SAP_CAPIRE_SACPOC_ITEMCUSTOMREFERENCE 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ID, TYPECODE, ITEM_ITEMID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', CONCAT('icr-id-', FLOOR(rand()*1000)) , CONCAT('icr-typecode', FLOOR(rand()*500)), ITEM_ID_3);
		-- icr 4-1
		INSERT INTO SAP_CAPIRE_SACPOC_ITEMCUSTOMREFERENCE 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ID, TYPECODE, ITEM_ITEMID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', CONCAT('icr-id-', FLOOR(rand()*1000)) , CONCAT('icr-typecode', FLOOR(rand()*500)), ITEM_ID_4);
		-- icr 4-2
		INSERT INTO SAP_CAPIRE_SACPOC_ITEMCUSTOMREFERENCE 
			(CREATEDAT, CREATEDBY, MODIFIEDAT, MODIFIEDBY, ID, TYPECODE, ITEM_ITEMID) 
		VALUES
			(CURRENT_UTCTIMESTAMP, 'anonymous', CURRENT_UTCTIMESTAMP, 'anonymous', CONCAT('icr-id-', FLOOR(rand()*1000)) , CONCAT('icr-typecode', FLOOR(rand()*500)), ITEM_ID_4);
	
		
		-- ===== Insert into flatten table
		INSERT INTO SAP_CAPIRE_SACPOC_SUBSCRIPTIONFLATTEN L 
		(SUBSCRIPTIONID, VERSION, DOCUMENTNUMBER, SUBSCRIPTIONDOCUMENTID, STATUS, CUSTOMERID, MARKETID, VALIDFROM, VALIDUNTIL, CHANGEDAT, CHANGEDBY, CREATEDAT, CREATEDBY, CANCELLATIONREASON, SNAPSHOTID, EFFECTIVEDATE, EVENTID, EVENTTYPE, SEQUENCENUMBER, ITEMID, PRODUCTID, CUSTOMREFERENCEID, TYPECODE, ITEMCUSTOMREFERENCEID, ITEMTYPECODE)
		(SELECT 
		SUBSCRIPTIONID, VERSION, DOCUMENTNUMBER, SUBSCRIPTIONDOCUMENTID, STATUS, CUSTOMERID, MARKETID, VALIDFROM, VALIDUNTIL, scss.CHANGEDAT, scss.CHANGEDBY, scss.CREATEDAT, scss.CREATEDBY, CANCELLATIONREASON, SNAPSHOTID, EFFECTIVEDATE, scse.ID, EVENTTYPE, SEQUENCENUMBER, ITEMID, PRODUCTID, scsc.ID, scsc.TYPECODE, scsi2.ID, scsi2.TYPECODE
		FROM SAP_CAPIRE_SACPOC_SUBSCRIPTION scss
		LEFT JOIN SAP_CAPIRE_SACPOC_SNAPSHOT scss2 ON scss.SUBSCRIPTIONID = scss2.SUBSCRIPTION_SUBSCRIPTIONID 
		LEFT JOIN SAP_CAPIRE_SACPOC_EVENTLOGENTRY scse  ON scss.SUBSCRIPTIONID = scse.SUBSCRIPTION_SUBSCRIPTIONID
		LEFT JOIN SAP_CAPIRE_SACPOC_ITEM scsi ON scsi.SNAPSHOT_SNAPSHOTID = scss2.SNAPSHOTID 
		LEFT JOIN SAP_CAPIRE_SACPOC_CUSTOMREFERENCE scsc ON scss.SUBSCRIPTIONID = scsc.SUBSCRIPTION_SUBSCRIPTIONID 
		LEFT JOIN SAP_CAPIRE_SACPOC_ITEMCUSTOMREFERENCE scsi2 ON  scsi2.ITEM_ITEMID = scsi.ITEMID 
		WHERE scss.SUBSCRIPTIONID = :SUBSCRIPTION_ID);
		

		-- loop end
		-- count++
		SELECT :COUNT+1 INTO "COUNT" FROM DUMMY;
	
	END WHILE;
END;

```
 
</details>


### Metholodgies

To simulate the SAC use case, create cube calculation views for both models and execute view SQLs to get subscription info (1st level) and item custom reference info (4th level) together. Use the CPU execution time of the SQL Analyzer Plan function in Database Explorer to measure the performance. 

### Results with 100k subscriptions

All ids are unique. In this case, only payload 1 is used. One subscription creates ~40 rows in flatten table.

**DB Size Consumption**

DB size differences are minor. 126.6 MB for hierarchy vs 126.0 MB flatten storage. *Db size data is fetched from Hana internal table `M_TABLE_PERSISTENCE_STATISTICS`*

**SQL Execution CPU Time**

Query | flatten | hierachy
-|-|-
Query Subscription Status (access 1 table) | ![hierachy_60k](./fig/flatten_100k_status.png) | ![flatten_60k](./fig/hierarchy_100k_status.png)
Query Custom Reference (access 2 table) | ![hierachy_60k](./fig/flatten_100k_cr.png) | ![flatten_60k](./fig/hierarchy_100k_cr.png)
Query Product (access 3 tables) | ![hierachy_60k](./fig/flatten_100k_product_query.png) | ![flatten_60k](./fig/hierachy_100k_product_query.png)
Query 1 (product id, access 4 tables) | ![hierachy_60k](./fig/flatten_10k_4tables_query.png) | ![flatten_60k](./fig/hierachy_10k_4tables_query.png)
Query 2 (icr, access 6 tables) | ![hierachy_60k](./fig/flatten_10k_6tables_query.png) | ![flatten_60k](./fig/hierachy_10k_6tables_query.png)

**Observations**

1. More table joinings will cause performance issues in the hierarchy model. It takes at most 100% more CPU time (when joining 6 tables) to execute a SQL. 
2. At 100k subscription level. The hierarchy model has a significant performance advantage when only accessing 1 table, and have equal performance with Flatten model when joining 2 or 3 tables. Considering the CRUD programming and maintenance perspective. Using hierarchy is better if we need to join mostly less or equal to 3 tables.


### Results with 100k subscriptions (With limited product id, market id, and customer reference)

<!-- In this case, we have at most 1,000 products, 500 custom reference type codes, 100,000 custom reference ids, 500 item custom reference type codes, 100,000 item custom reference ids. -->

**SQL Execution CPU Time**

Query | flatten | hierarchy
-|-|-
Query Subscription Status (access 1 table) | ![flatten_99k_status](./fig/flatten_99k_status.png) | ![hierarchy_99k_status](./fig/hierarchy_99k_status.png)
Query Custom Reference (access 2 table) | ![flatten_99k_cr](./fig/flatten_99k_cr.png) | ![hierarchy_99k_cr](./fig/hierarchy_99k_cr.png)
Query Product (access 3 tables) | ![flatten_99k_product](./fig/flatten_99k_product.png) | ![hierarchy_99k_product](./fig/hierarchy_99k_product.png)
Query 1 (product id, access 4 tables) | ![flatten_99k_product](./fig/flatten_99k_icr.png) | ![hierarchy_99k](./fig/hierarchy_99k_icr.png)
Query 2 (icr, access 6 tables) | ![flatten_99k_icr_event](./fig/flatten_99k_icr_event.png) | ![hierarchy_99k_icr_event](./fig/hierarchy_99k_icr_event.png)


### Summary

![results_100k](./fig/results_100k.png)
![results_1million](./fig/results_1million.png)

	
**Pros and Cons**

Aspect | Normalized Model | Hierarchy Model
:-:|-|-
Read Performance|Good when joining over 3 tables, but already more than 5 seconds. | Good when working with fewer table joins and less data processing
Storage Consumption in HANA|Two models consume equal storage at 100k (126 mb); Hierarchy model (613 mb) is around 50% less compared with Normalized model (1,000 mb) at 1 million subscriptions.｜
Maintenance|Hard to maintain. One subscription payload insertion will lead to tens of rows insertions in the normalized table to be manually maintained. | CAP standard function to execute read/write. 
Extendbilitiy | Easy to extend by only modifying a subtable. Data is easier to fill in by copying data from PostgreSQL with a similiar table structure | Relatively hard. Modifying the large table is expensive and we need to fillin more old data.

	
### Conclusions
	
- We implement write API in this performance test. We realize that we can utilize the CAP native implementation of CRUD for the hierarchy model. On the contrary, we'd have to manually implement the database read/write logic with large loops which are difficult to maintain if anything goes wrong.
- From the performance perspective, even though the normalized table has better performance while joining multiple tables, the response time already reaches 5+ seconds which is a huge impact on SAC plotting. According to the test, we can use some tricks to improve the performance.
	- While joining multiple tables, force users to use filters to reduce the data volume (e.g., filtering the custome reference typecode).
	- We could normalize some tables instead of normalizing all tables. The hierarchy model provides good performance as long as we limit the number of join tables to less than or equal to 3. Normalizing Subscription, Snapshot, and Item tables could be a nice option.
