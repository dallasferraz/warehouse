# warehouse
Fictitious simplified SQL server database. Trigger for keeping records of inventory modifications. Feel free to modify and use it as you please.

In this database, everything was simplified to the extremes: there is only one group, the tables have no relationships and the dummy data is really easy to grasp just by looking at it. The tables are represented in the diagram below:

![model](https://raw.githubusercontent.com/dallasferraz/warehouse/master/model.png)

After creating the tables the chunk of code below is responsible for feeding them some content, just so the trigger developed for future records is implemented in a context:

```tsql
USE warehouse
GO

INSERT INTO employee VALUES(1,'John')
INSERT INTO employee VALUES(2,'Mary')
INSERT INTO employee VALUES(3,'Joseph')
GO

INSERT INTO inventory VALUES(1,'NVI-0100',34)
INSERT INTO inventory VALUES(2,'NVI-7200',100)
INSERT INTO inventory VALUES(3,'NVI-4130',18)
INSERT INTO inventory VALUES(4,'NVI-0090',50)
INSERT INTO inventory VALUES(5,'NVI-0506',41)
GO

INSERT INTO records VALUES(1,1,0,34,'REPLENISHMENT','2019-06-04 15:34:26','Mary')
INSERT INTO records VALUES(2,2,0,100,'REPLENISHMENT','2019-06-10 09:20:02','Mary')
INSERT INTO records VALUES(3,3,0,18,'REPLENISHMENT','2019-06-15 11:56:40','Joseph')
INSERT INTO records VALUES(4,4,0,50,'REPLENISHMENT','2019-06-21 20:00:57','John')
INSERT INTO records VALUES(5,5,0,41,'REPLENISHMENT','2019-06-22 14:11:33','Joseph')
INSERT INTO records VALUES(6,3,18,50,'REPLENISHMENT','2019-07-01 07:04:00','Joseph')
INSERT INTO records VALUES(7,1,34,30,'RETRIEVAL','2019-07-01 12:45:19','John')
INSERT INTO records VALUES(8,5,41,17,'RETRIEVAL','2019-07-02 10:33:20','Mary')
INSERT INTO records VALUES(9,5,17,70,'REPLENISHMENT','2019-07-02 14:34:06','John')
INSERT INTO records VALUES(10,2,100,32,'RETRIEVAL','2019-07-02 19:17:28','Joseph')
GO
```

So the tables now hold some information as shown below:

![tablecontents](https://raw.githubusercontent.com/dallasferraz/warehouse/master/tablecontents.png)

Now a trigger can be responsible for keeping up with the modifications in the inventory table, just as shown before. It can do so by automatically keeping records of modifications in the inventory table, whenever an employee retrieves or replenishes the SKU stock units. The chunk of code below displays how this trigger works:

```tsql
USE warehouse
GO

CREATE TRIGGER tgr_new_record
ON dbo.inventory
FOR UPDATE
AS
	DECLARE @idRecord INT
	DECLARE @idSKU INT
	DECLARE @quantityBefore INT
	DECLARE @quantityAfter INT
	DECLARE @operationType VARCHAR(13)
	DECLARE @timestamp DATETIME
	DECLARE @employee VARCHAR(50)

	SELECT @idRecord = COUNT(*) FROM records
	SELECT @idSKU = idSKU FROM inserted
	SELECT @quantityBefore = quantity FROM deleted
	SELECT @quantityAfter = quantity FROM inserted

	IF(@quantityBefore > @quantityAfter)
		BEGIN
			SET @operationType = 'RETRIEVAL'
		END
	ELSE
		BEGIN
			SET @operationType = 'REPLENISHMENT'

		END
	SET @timestamp = GETDATE()
	SET @employee = SUSER_NAME()
	SET @idRecord = @idRecord + 1

	INSERT INTO records(idRecord,idSKU,quantity_before,quantity_after,operation_type,"timestamp",employee) VALUES
	(@idRecord,@idSKU,@quantityBefore,@quantityAfter,@operationType,@timestamp,@employee)

	PRINT 'Records up to date.'
GO
```