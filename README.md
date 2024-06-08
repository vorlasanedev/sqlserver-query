# sqlserver-query
## insert into with lef-join many table
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
