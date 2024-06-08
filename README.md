# sqlserver-query
## insert into with lef-join many table with trigger database After Insert Table
```
USE [KKL]
GO
/****** Object:  Trigger [dbo].[trg_sentto_report_web]    Script Date: 08/06/2024 9:57:17 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
ALTER TRIGGER [dbo].[trg_sentto_report_web]
   ON  [dbo].[LTU110] 
   AFTER INSERT 

AS 
BEGIN
	
 insert into [kklottery].[dbo].[reports]   
 Select pay_no = c.pay_no ,a.bill_no ,a.barcode ,a.cust_dealer ,b.cust_nm ,c.vat ,c.amt,amtvat=c.vat+c.amt  ,a.cust_seller ,
 cust_seller_name=( select top 1 cust_nm from ltc100 where cust_cd = a.cust_seller ) ,c.pay_dt ,c.pay_by ,c.pay_on_round ,c.pay_ty ,c.rmk ,
 c.insert_fr , created_at=c.cdt , updated_at=c.mdt 
  from LTU140 a  
 left join LTC100 b  on a.cust_dealer =b.cust_cd
 join LTU110 c on a.bill_no=c.bill_no 
 where a.roundId = ( select roundId from inserted)
  and a.bill_no = ( select bill_no from inserted)
  and pay_by is not null 


END

```

## Trigger Delete
```
USE [KKL]
GO
/****** Object:  Trigger [dbo].[TRG_BEF_DEL_LTU100]    Script Date: 08/06/2024 10:02:19 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
ALTER TRIGGER [dbo].[TRG_BEF_DEL_LTU100]
   ON  [dbo].[LTU110] 
  for DELETE
AS 
BEGIN
	
INSERT INTO dbo.LTU110_BK
           (pay_no
           ,bill_no
           ,cust_dealer
           ,vat
           ,amt
           ,pay_dt
           ,pay_by
           ,pay_on_round
           ,pay_ty
           ,rmk
           ,insert_fr
           ,device_id
           ,cdt
           ,mdt
           ,pid)
  select
            pay_no 
           ,bill_no
           ,cust_dealer
           ,vat
           ,amt
           ,pay_dt
           ,pay_by
           ,pay_on_round
           ,pay_ty
           ,rmk
           ,insert_fr
           ,device_id
           ,cdt
           ,mdt
           ,pid
     FROM DELETED     


	END

```
