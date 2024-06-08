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

## Trigger Update
```
USE [KKL]
GO
/****** Object:  Trigger [dbo].[updat_ont]    Script Date: 08/06/2024 10:15:04 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER TRIGGER [dbo].[updat_ont] on [dbo].[LIV100]
    FOR UPDATE
AS 
BEGIN
	

 update LIV100
    set receive_amt_his=  (select receive_amt from inserted)
  where inv_no =(select inv_no from inserted)
	and receive_amt_his is null


	update a 
	   set ontime_amt = case when pay_dt <= (select ont_dt from LTM300 where roundId =a.roundId ) then a.inv_amt*2/100 else 0 end  
	  from LIV100 a
	 where pay_dt=(select pay_dt from inserted)
	   and inv_no =(select inv_no from inserted)
	   and a.cust_dealer not in ('BcelOne','IBCools','LaoQR','LottoPoints','MMoney','UMoney') 

    update a 
	   set receive_amt = case when  a.cust_dealer in ('MMoney') then inv_amt - isnull(dealer_amt,0)- luky_amt-pr_rd_amt  else inv_amt - isnull(dealer_amt,0)- isnull(ontime_amt,0) - isnull(target_avg,0) - isnull(dl_pay_amt,0)- isnull(pr_rd_amt,0) - isnull(data_pkg,0)+ (isnull( carry,0)) end
	  from LIV100 a
	 where pay_dt=(select pay_dt from inserted)
	   and inv_no =(select inv_no from inserted)

END

```
## triger delete
```
USE [KKL]
GO
/****** Object:  Trigger [dbo].[TRG_BEF_DEL]    Script Date: 08/06/2024 10:16:00 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
ALTER TRIGGER [dbo].[TRG_BEF_DEL]
   ON  [dbo].[LIV100]
   FOR DELETE
AS 

INSERT INTO LIV100_BK  (
       inv_no
      ,seq
      ,cntr_no
      ,cust_dealer
      ,roundid
      ,inv_amt
      ,inv_dt
      ,dealer_amt
      ,ontime_amt
      ,target_avg
      ,luky_amt
      ,dl_pay_amt
      ,unit_avg
      ,receive_amt
      ,pr_rd_chk
      ,pr_rd_amt
      ,data_pkg
      ,issue_qty
      ,disable_qty
      ,active_qty
      ,cur_bc
      ,rmk
      ,pay_no
      ,pay_dt
      ,insert_fr
      ,crnt_rate
      ,carry
      ,carry_inv
      ,receive_amt_his
      ,device_id
      ,cid
      ,cdt
      ,mid
      ,mdt
      ,stat_bc

)
     SELECT inv_no
      ,seq
      ,cntr_no
      ,cust_dealer
      ,roundid
      ,inv_amt
      ,inv_dt
      ,dealer_amt
      ,ontime_amt
      ,target_avg
      ,luky_amt
      ,dl_pay_amt
      ,unit_avg
      ,receive_amt
      ,pr_rd_chk
      ,pr_rd_amt
      ,data_pkg
      ,issue_qty
      ,disable_qty
      ,active_qty
      ,cur_bc
      ,rmk + 'From Delete'
      ,pay_no
      ,pay_dt
      ,insert_fr
      ,crnt_rate
      ,carry
      ,carry_inv
      ,receive_amt_his
      ,device_id
      ,cid
      ,cdt
      ,mid
      ,mdt
      ,stat_bc
     FROM DELETED 

--BEGIN
--	-- SET NOCOUNT ON added to prevent extra result sets from
--	-- interfering with SELECT statements.
--	SET NOCOUNT ON;

--    -- Insert statements for trigger here

--END
--GO

```

## Trigger Merge table
```
USE [ERP]
GO
/****** Object:  Trigger [dbo].[TG_ACA100]    Script Date: 08/06/2024 10:18:37 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER trigger [dbo].[TG_ACA100] ON [dbo].[ACA100]
    for insert, update, delete
as
begin
  ---------------------------------
  --Account Summry
  ---------------------------------
  merge ACA200 a
  using (
          select company, plant, ymd, a.acc, bankacc, dr=sum(dr), cr=sum(cr)
              from (
                    select company, plant, ymd=cfm_date, acc, bankacc=isnull(bankacc,'*'),
                           dr=isnull(dr,0), cr=isnull(cr,0)
                      from inserted
                     union all
                    select company, plant, ymd=cfm_date, acc, bankacc=isnull(bankacc,'*'),
                           dr=0-isnull(dr,0), cr=0-isnull(cr,0)
                      from deleted
                   ) a
		  left join cda200 b on a.acc =b.acc 
		    where b.yn_balance =1
             group by a.company, a.plant, a.ymd, a.acc, a.bankacc
        ) b on a.company=b.company and a.plant=b.plant 
           and a.ymd=b.ymd and a.acc=b.acc and a.bankacc=b.bankacc
  when matched then
    update set
      a.dr=a.dr+b.dr,
      a.cr=a.cr+b.cr
  when not matched then
    insert (company,plant,ymd,acc,bankacc,dr,cr)
    values (b.company,b.plant,b.ymd,b.acc,b.bankacc,b.dr,b.cr);

  ---------------------------------
  --Account Balance
  ---------------------------------
  merge ACA300 a
  using (
          select a.company,a.plant,a.slip_no,offset=isnull(a.offset,'*'),a.vendor,a.acc,idate=a.rmks_DATE1,
                 ddate=a.rmks_DATE2,a.curr,
                 iamt=(case when b.yn_dc='D' and dr<>0 then dr 
                            when b.yn_dc='C' and cr<>0 then cr else 0 end),
                 oamt=(case when b.yn_dc='D' and cr<>0 then cr 
                            when b.yn_dc='C' and dr<>0 then dr else 0 end),
                 invoice=a.rmks_NO1,cntlno=a.rmks_NO2,a.rmks
            from (
                  select company, plant, slip_no, offset, vendor, acc,
                         rmks_DATE1, rmks_DATE2, curr, dr, cr,
                         rmks_NO1, rmks_NO2, rmks
                    from inserted
                   union all
                  select company, plant, slip_no, offset, vendor, acc,
                         rmks_DATE1, rmks_DATE2, curr, 0-dr, 0-cr,
                         rmks_NO1, rmks_NO2, rmks
                    from deleted
                 ) a
            join cda200 b on a.acc=b.acc
           where b.yn_balance=1
         ) b on a.company=b.company and a.plant=b.plant and a.slip_no=b.slip_no and a.offset=b.offset

  when matched then
    update set
      a.iamt=a.iamt+b.iamt,
      a.oamt=a.oamt+b.oamt
  when not matched then
    insert (company,plant,slip_no,offset,vendor,acc,idate,ddate, 
            curr,iamt,oamt,invoice,cntlno,rmks)
    values (b.company,b.plant,b.slip_no,offset,b.vendor,b.acc,b.idate,b.ddate,
            b.curr,b.iamt,b.oamt,b.invoice,b.cntlno,b.rmks);
end;
```
## triger insert
```
USE [ERP]
GO
/****** Object:  Trigger [dbo].[BCA100_Insert_BCA100L]    Script Date: 08/06/2024 10:22:20 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER Trigger [dbo].[BCA100_Insert_BCA100L] on [dbo].[BCA100]
For Insert
as


declare		@main_cd	varchar(10) = 'SC220',
			@lan_cd		varchar(10),
			@lan_id		int

select top 1 @lan_cd = base_cd,
			 @lan_id = m1
from BCA200V
where main_cd = @main_cd
and use_yn = '1'
and	isnull(m3,'0') = '1'
order by ord_sq

set @lan_cd = isnull(@lan_cd,'SC220KO');
set @lan_id = case when isnull(@lan_id,0) = 0 then 1 else @lan_id end;


--기본언어 반영
with BASE as (select c.main_cd, lan_id = case when isnumeric(g.m1) = 1 then g.m1 else @lan_id end,
						rtrim(ltrim(c.title)) as nm
				from INSERTED c
					left join BCA100 e on c.main_cd = e.main_cd
					left join SCU100 f on e.mid = f.reg_id
					left join BCA200V g on isnull(f.lan_cd,@lan_cd) = g.base_cd
					left join BCA100L h on c.main_cd = h.main_cd and case when isnumeric(g.m1) = 1 then g.m1 else @lan_id end = h.lan_id
				where isnull(rtrim(ltrim(c.title)),'') <> isnull(rtrim(ltrim(h.nm)),'')
				)
merge into BCA100L as a
using (select c.main_cd, c.lan_id,
				c.nm
		from BASE c
		union 
		--기본언어 이외의 값 가져오기
		select y.main_cd, y.lan_id,
				isnull(max(y.nm),x.nm)
		from (
				--기본언어 Title 가져오기
				select c.main_cd, lan_id = e.mng_val,
						max(g.nm) as nm
				from BASE c
					join BCA200 d on d.main_cd = @main_cd and d.use_yn = '1'
					join BCA250 e on d.base_cd = e.base_cd and e.mng_sq = 1 and isnumeric(e.mng_val) = 1
					left join BCA100L f on c.nm = f.nm and c.lan_id = f.lan_id 
											and exists(select top 1 0
														from BCA100L z
														where f.main_cd = z.main_cd and f.lan_id <> z.lan_id
														and isnull(z.nm,'') <> '')
					left join BCA100L g on f.main_cd = g.main_cd and c.lan_id <> g.lan_id and e.mng_val = g.lan_id
				group by c.main_cd, e.mng_val, g.nm
				)y
				join BASE x on y.main_cd = x.main_cd
		where not exists(select top 1 0
						from BASE z
						where y.main_cd = z.main_cd and y.lan_id = z.lan_id)
		group by y.main_cd, y.lan_id, x.nm
		) AS b 
		ON a.main_cd = b.main_cd and a.lan_id = b.lan_id
when not matched by target then
	insert (main_cd, lan_id, 
			nm)
	values (b.main_cd, b.lan_id, 
			b.nm);

```
## triger update
```
USE [ERP]
GO
/****** Object:  Trigger [dbo].[BCA100_Update_BCA100L]    Script Date: 08/06/2024 10:23:19 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER Trigger [dbo].[BCA100_Update_BCA100L] on [dbo].[BCA100]
For Update
as

declare		@main_cd	varchar(10) = 'SC220',
			@lan_cd		varchar(10),
			@lan_id		int

select top 1 @lan_cd = base_cd,
			 @lan_id = m1
from BCA200V
where main_cd = @main_cd
and use_yn = '1'
and	isnull(m3,'0') = '1'
order by ord_sq

set @lan_cd = isnull(@lan_cd,'SC220KO');
set @lan_id = case when isnull(@lan_id,0) = 0 then 1 else @lan_id end;


--기본언어 반영
with BASE as (select c.main_cd, lan_id = case when isnumeric(g.m1) = 1 then g.m1 else @lan_id end,
						rtrim(ltrim(c.title)) as nm
				from INSERTED c
					left join DELETED d on c.main_cd = d.main_cd
					left join BCA100 e on c.main_cd = e.main_cd
					left join SCU100 f on e.mid = f.reg_id
					left join BCA200V g on isnull(f.lan_cd,@lan_cd) = g.base_cd
					left join BCA100L h on c.main_cd = h.main_cd and case when isnumeric(g.m1) = 1 then g.m1 else @lan_id end = h.lan_id
				where (isnull(rtrim(ltrim(c.title)),'') <> isnull(rtrim(ltrim(d.title)),'') or isnull(rtrim(ltrim(c.title)),'') <> isnull(rtrim(ltrim(h.nm)),''))
				)
merge into BCA100L as a
using (select c.main_cd, c.lan_id,
				c.nm
		from BASE c
		) AS b 
		ON a.main_cd = b.main_cd and a.lan_id = b.lan_id
when matched then
	update set nm = b.nm
when not matched by target then
	insert (main_cd, lan_id, 
			nm)
	values (b.main_cd, b.lan_id, 
			b.nm);

```
## Store Procedure
```
USE [ERP]
GO
/****** Object:  StoredProcedure [dbo].[AdvClose_Quarter]    Script Date: 08/06/2024 10:24:24 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


ALTER procedure [dbo].[AdvClose_Quarter] (@pay_no varchar(30),@n_payno varchar(30))
AS 
begin

declare @_dt date

select @_dt = case when month(getdate())=1 then getdate()
                   when month(getdate()) < =4 and month(getdate()) >=2 then convert(varchar(4),getdate(),120)+'-04'+'-01'
                   when month(getdate()) < =7 and month(getdate()) >4 then convert(varchar(4),getdate(),120)+'-07'+'-01'         
                   when month(getdate()) < =10 and month(getdate()) >7 then convert(varchar(4),getdate(),120)+'-10'+'-01'         
                   when month(getdate()) < =12 and month(getdate()) >10 then convert(varchar(4),dateadd(year,1,getdate()),120)+'-01'+'-01'end



if exists (select 1 from hbg300 where pay_no=@n_payno)
begin
return
end 


--insert into hbg310 
--       (bu_cd,pay_no,seq,yymm,st_yymm,pc_code,ac_code,ac_group,need_ar,
--        curr,req_amt,act_amt,serial_no,supp,pay_desc,rmks,usd_rate,set_rate,usd_amt,
--        ref0,ref1,ref2,ref3,ref4,ref5,ref6,fi_cfm,chk_cl,chk_ty,cid,cdt,mid,mdt,ar_dt
--       )
--select bu_cd,a.pay_no,seq=a.seq+b.seq,yymm,st_yymm,pc_code,ac_code='ZZ999',ac_group,need_ar,
--       curr,req_amt=0,act_am=req_amt,serial_no,supp,pay_desc,rmks,usd_rate,set_rate,usd_amt,
--       ref0,ref1,ref2,ref3,ref4,ref5,ref6,fi_cfm,chk_cl,chk_ty,cid,cdt,mid,mdt,ar_dt
--  from hbg310 a
--  join (select pay_no,seq=max(seq) from hbg310 where pay_no=@pay_no group by pay_no) b on a.pay_no=b.pay_no 
-- where a.pay_no=@pay_no
--  and ac_code<>'ZZ999'
--  --and ac_code<>'AA010'

insert into hbg300 
       (pay_no,bu_cd,req_dt,pay_bc,pay_ty,req_ty,dept_cd,dt1,dt2,pay_dt,pay_desc,stat_bc,
        pc_code,pay_method,method_nm,acc_no,acc_nm,ivc_no,serial_no,voucher_no,rmks,
        reg_id,emp_no,head_no,chk_ceo_no,ceo_no,chk_appr_no,appr_no,chk_fi_no,fi_no,rec_no,
        sig_emp,sig_head,sig_chk_appr,sig_appr,sig_chk_ceo,sig_ceo,
        sig_chk_fi,sig_fi,sig_rec,head_dt,appr_dt,ceo_dt,fi_dt,emp_dt,chk_emp_dt,
        chk_appr_dt,chk_ceo_dt,chk_fi_dt,rec_dt,emp_cm,head_cm,appr_cm,chk_appr_cm,ceo_cm,chk_ceo_cm,
        chk_fi_cm,fi_cm,dept_cfm,bu_cfm,final_cfm,s_dept_cfm,s_bu_cfm,s_final_cfm,cfm_dt,cfm_no,cust_cd,
        site_cd,wh_cd,chk_set,disburser_no,sig_disburser,disburser_dt,
        s_disburser_dt,rec_tel,rec_nm,closed,co_appr_no,sig_co_appr,
        co_appr_dt,co_appr_cm,exp_dt,bank_ty,f_pay_no,
        cid,cdt,mid,mdt
       )
select pay_no=@n_payno,bu_cd,req_dt=@_dt,pay_bc,pay_ty,req_ty,dept_cd,dt1,dt2,pay_dt,pay_desc,stat_bc='FA994200',
       pc_code,pay_method,method_nm,acc_no,acc_nm,ivc_no,serial_no,voucher_no,rmks,
       reg_id,emp_no,head_no,chk_ceo_no,ceo_no,chk_appr_no,appr_no,chk_fi_no,fi_no,rec_no,
       sig_emp,sig_head,sig_chk_appr,sig_appr,sig_chk_ceo,sig_ceo,
       sig_chk_fi,sig_fi,sig_rec,head_dt,appr_dt,ceo_dt,fi_dt,emp_dt,chk_emp_dt,
       chk_appr_dt,chk_ceo_dt,chk_fi_dt,rec_dt,emp_cm,head_cm,appr_cm,chk_appr_cm,ceo_cm,chk_ceo_cm,
       chk_fi_cm,fi_cm,dept_cfm,bu_cfm,final_cfm,null,null,null,cfm_dt,cfm_no,cust_cd,
       site_cd,wh_cd,chk_set,disburser_no,sig_disburser,disburser_dt,
       s_disburser_dt,rec_tel,rec_nm,closed,co_appr_no,sig_co_appr,
       co_appr_dt,co_appr_cm,exp_dt,bank_ty,pay_no,
       cid,cdt,mid,mdt
  from hbg300
 where pay_no=@pay_no


 if exists (select 1 from hbg310 where pay_no=@n_payno)
begin
return
end 

insert into hbg310 
       (bu_cd,pay_no,seq,yymm,st_yymm,pc_code,ac_code,ac_group,need_ar,
        curr,req_amt,act_amt,serial_no,supp,pay_desc,rmks,usd_rate,set_rate,usd_amt,
        ref0,ref1,ref2,ref3,ref4,ref5,ref6,fi_cfm,chk_cl,chk_ty,cid,cdt,mid,mdt,ar_dt
       )
select bu_cd,pay_no=@n_payno,seq,yymm=convert(varchar(7),@_dt,120),st_yymm,pc_code,ac_code,ac_group,need_ar,
       curr,req_amt,act_amt,serial_no,supp,pay_desc,rmks,usd_rate,set_rate,usd_amt,
       ref0,ref1,ref2,ref3,ref4,ref5,ref6,fi_cfm,chk_cl,chk_ty,cid,cdt,mid,mdt,ar_dt
  from hbg310
 where pay_no=@pay_no
  and ac_code='AA010'


end 

```
## stoer procedure work
```
USE [ERP]
GO
/****** Object:  StoredProcedure [dbo].[BCV122_WORK]    Script Date: 08/06/2024 10:25:17 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
/************************************************************************************************  
***** Function Name : [거래처 계좌 승인]                                         		      *****  
***** Creater       : 황주용                                                                 *****  
***** Work Date     : 2016.10.24															*****  
***** ModIfier	    :																		*****  
***** ModIfy Date   :																		*****  
***** Description   :																		*****  
***** Argument	    :																		*****  
*****																						*****  
**************************************************************************************************/  
-- exec [bcv122_Work] '01', 'A01231' ,'123123123', '1',10

ALTER	PROCEDURE [dbo].[BCV122_WORK]			
				@co_cd		VARCHAR(10),	-- Company Code
				@cust_cd    VARCHAR(30),    -- Cust Code
                @acct_no    VARCHAR(50),    -- Account Number
				@apr_yn     char(1),        -- Approval Y/N
                @reg_id		INT				-- Approval User
                
AS
			
------------------------------------------------- 필수			
SET NOCOUNT ON			
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED			
-------------------------------------------------	

/******************************************************************/
-- 1. 승인처리
/******************************************************************/
declare @seq INT
set @seq = isnull((select   top 1 a.sq_no 
                    from    bcv120 a
                    where   a.co_cd = @co_cd
                    and     a.cust_cd = @cust_cd
                    order by a.sq_no desc ), 0)

if @apr_yn = '1' begin

    update  a
    set     a.apr_yn = '1', a.apr_dt = getdate(),
    a.apr_rid = @reg_id
    from    bcv121 a
    where   a.cust_cd = @cust_cd
    and     a.acct_no = @acct_no
    
    insert into bcv120
                (co_cd, cust_cd, sq_no, acct_no, acct_nm, bank_cd, bank_nm, 
                branch, acct_bc, bas_yn, use_yn, adm_yn, adm_dt, adm_uid, 
				rmks, fr_dt, cont_no,
                cid, cdt, mid, mdt)
    select  @co_cd, @cust_cd, @seq + 1, @acct_no, a.acct_nm, a.bank_cd, '' as bank_nm, -- 수정필요
            a.branch, a.acct_bc, a.bas_yn, '1', '1', getdate(), @reg_id, 
			a.rmks, a.fr_dt, a.cont_no,
            @reg_id, getdate(), @reg_id, getdate()
    from    bcv121 a     
    where   a.cust_cd = @cust_cd
    and     a.acct_no = @acct_no
     
        
end

else begin
/******************************************************************/
-- 2. 승인취소처리
/******************************************************************/
    update  a
    set     a.apr_yn = '0', a.apr_dt = null,
            a.apr_rid = null
    from    bcv121 a
    where   a.cust_cd = @cust_cd
    and     a.acct_no = @acct_no

    delete
    from    bcv120
    where   co_cd = @co_cd
    and     cust_cd = @cust_cd
    and     acct_no = @acct_no
end
```
## store procedure query
```
USE [ERP]
GO
/****** Object:  StoredProcedure [dbo].[BCV300_Query]    Script Date: 08/06/2024 10:26:36 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER  procedure [dbo].[BCV300_Query] (
	@co_cd			varchar(10),
	@cust_cd		varchar(30)
)
as
begin

	set nocount on
	set transaction isolation level read uncommitted

	--****************************************************************************************************
	-- Writer	: KS.Choi
	-- Date		: 13.Sep.2016
	-- Reg.		: -
	--****************************************************************************************************
	

	-- Step 1. Declare
	--====================================================================================================
	declare @msg as nvarchar(1000), @msg_no as varchar(20), @tmp_no as varchar(10), @row_cnt as int
	declare @sys_dt as datetime = convert(varchar(10), getdate(), 120)


	-- Setp 2. Validation
	--====================================================================================================
	-- MS-SQL RegExp Error Escape
	/*
	if charindex('[', isnull(@itm_cd,''), 1)  > 0
		set @itm_cd = replace(@itm_cd, '[', '[[]')
	if charindex('[', isnull(@itm_nm,''), 1)  > 0
		set @itm_nm = replace(@itm_nm, '[', '[[]')
	*/
	

	-- Step 3. Reconsolidation
	--====================================================================================================

	
	-- Step 4. Retuan
	--====================================================================================================

	select	a.co_cd, a.cust_cd, b.cust_nm, a.sq_no, a.acct_no, a.acct_nm, a.bank_cd, a.bank_nm, a.bas_yn, a.use_yn, a.rmks, 
			a.cdt, z1.nm as cnm, a.mdt, z2.nm as mnm
	from BCV310 a
		left join BCV100 b on a.cust_cd = b.cust_cd and a.co_cd = b.co_cd
		left join scu100 z1 on a.cid = z1.reg_id
		left join scu100 z2 on a.mid = z2.reg_id
	where 1 = 1
	  and a.co_cd = @co_cd
	  and a.cust_cd = @cust_cd
	  and a.reg_dt = @sys_dt
	order by a.sq_no

	return;

exception:

end
```
## Procedure Sum
```
USE [ERP]
GO
/****** Object:  StoredProcedure [dbo].[ESJ100_SUM]    Script Date: 08/06/2024 10:28:41 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
/*

 ###################################################################################################
 제         목  :  구매 실적 집계
 사  용    D B  :  
 VB PROGRAM     :  ESJ100
 작   성   자   :  주세진
 작   성   일   :  2011.06.16
 수   정   자   :  
 수   정   일   :  
 개	     요     :  
 ###################################################################################################

EXEC ESJ100_SUM '01', '2011', 0
 select * from dmr100
*/

ALTER	PROCEDURE [dbo].[ESJ100_SUM]
						@co_cd			varchar(10),
						@std_mm			varchar(4),
						@reg_id			INT = 0
AS

set nocount on
set transaction isolation level read uncommitted



declare @cstd_mon		varchar(7),	-- 매출년월 
        @cgrp_bc      varchar(20),
        @ccust_cd	varchar(20),
		@camt			decimal(15,0)
		


--원래 실적 기록 모두 삭제
update	ESJ100
set	act = 0
where	co_cd = @co_cd
and	std_mon like @std_mm + '%'


declare ESJ100_cursor cursor for

	select z.std_mon, z.grp_bc, z.cust_cd, amt = sum(z.amt)
	from (
			select	std_mon = convert(varchar(7),d.set_dt,121), 
			
					case when e.itm_bc = 'DM100300' and e.grp1_cd = 'A100600' then 'ES301100'		--페인트
						 when e.itm_bc = 'DM100300' and e.grp1_cd = 'A100500' then 'ES301300'		--필름
						 when e.itm_bc = 'DM100300' and isnull(e.grp3_cd,'') in ('C100110', 'C100120', 'C100130')
																		  then 'ES301500'	--소재(원재료 and 소분류 = pnl or swhl)
						 when e.itm_bc = 'DM100200'	then 'ES301800'	-- 상품
					else 'ES301900' end grp_bc,		--기타
							
/*
case e.grp1_cd	when 'A100600' then 'ES301100'		--페인트
									when 'A100500' then 'ES301300'		--필름
					else	case when e.itm_bc = 'DM100300' 
									  and isnull(e.grp3_cd,'') in ('C100110', 'C100120', 'C100130')		--소재(원재료 and 소분류 = pnl or swhl)
										then 'ES301500'	
								 when e.itm_bc = 'DM100200'	
									  and h.model_cd <> '기타' then 'ES301800'	-- 상품
							
							else 'ES301900' end		--기타
*/							
							
													
					--end grp_bc,
									--when 'A100900' THEN 'ES301900'		--기타
									--ELSE 'ES301500' END AS GRP_BC ,		--소재(원재료 and 소분류 = pnl or swhl)
					cust_cd = isnull(d.cust_cd,''),  amt = isnull(d.set_amt,0)
				/* in_amt = sum(a.in_qty * dbo.fnPurchPrice (a.itm_id, a.fac_cd, d.cust_cd2, a.in_dt, ''))  */
			from lea100 a 

				left join bcc300 c on a.fac_cd = c.fac_cd
				join lea500 d on a.in_no = d.in_no and a.in_sq = d.in_sq	
				left join	dma100 e on a.itm_id = e.itm_id
				left join	dma110 h on a.itm_id = h.itm_id
				--left join mmv100 v on a.itm_id = v.itm_id and a.cust_cd = v.cust_cd
				join BCA250 v	on a.in_bc = v.base_cd and v.mng_sq = 2 and v.mng_val = '1'	--구매입고
				--join bca200v z on z.base_cd = a.in_bc and z.m2 = '1'			--> 구매입고인것만

			where 	year(d.set_dt) = @std_mm
			and 	c.co_cd = @co_cd) z
	
	group by  z.std_mon, z.grp_bc, z.cust_cd
	--group by  convert(varchar(7),d.set_dt,121), a.fac_cd, e.grp1_cd, d.cust_cd

open ESJ100_cursor		--선언된 커서열기

fetch next from ESJ100_cursor into @cstd_mon, @cgrp_bc, @ccust_cd, @camt
while (@@fetch_status = 0)
begin
	
		---------------------------------------------------------
		-- 기존 데이터 체크(계획)
		---------------------------------------------------------
		if exists (select 0  
					from	ESJ100 a 
					where	a.co_cd = @co_cd
					and	a.std_mon = @cstd_mon
					and	a.grp_bc = @cgrp_bc
					and	a.cust_cd = @ccust_cd )
		begin
		
			update	ESJ100
			set	act = @camt, mid = @reg_id, mdt = getdate()
			where	co_cd = @co_cd
			and	std_mon = @cstd_mon
			and	grp_bc = @cgrp_bc
			and	cust_cd = @ccust_cd 
						
		end
		else
		begin
			insert	into ESJ100(co_cd, std_mon, grp_bc, cust_cd,  pln, act, cid, cdt)
			values	        (@co_cd, @cstd_mon, @cgrp_bc,@ccust_cd, 0, @camt, @reg_id, getdate())
		
		end

   fetch next from ESJ100_cursor into 
			  @cstd_mon, @cgrp_bc, @ccust_cd, @camt

end
close ESJ100_cursor
deallocate ESJ100_cursor

```
## procedure Delete
```
USE [ERP]
GO
/****** Object:  StoredProcedure [dbo].[FAB100_Delete]    Script Date: 08/06/2024 10:29:29 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

/*
-- ###################################################################################################
-- 제         목  :  
-- 사  용    D B  :  
-- VB PROGRAM     :  
-- 작   성   자   :  
-- 개	     요   :  
-- ###################################################################################################
*/


ALTER  PROCEDURE [dbo].[FAB100_Delete]
				@doc_no		varchar(20),
				@reg_id		int
as

set nocount on
set transaction isolation level read uncommitted


	-----------------------------------------------------
	-- 자동승인관리 전표에 자동승인취소여부='1' 이면 승인취소작업
	-----------------------------------------------------
	

 --	if 1 = ( select isnull(a.del_apr_yn,0) 
	--		from FAA700 a
	--		join fab100 b on a.doc_bc = b.doc_bc and a.div_cd = b.div_cd and b.auto_yn = '1'
	--		where	b.doc_no= @doc_no )	
	--begin

 	if '1' = ( select isnull(a.auto_yn ,0) 
			from FAB100 a
			where	a.doc_no= @doc_no)	
	begin

		declare @doc_dt		date,
				@dept_cd	varchar(10)

		select @doc_dt= doc_dt,@dept_cd = dept_cd 
		from fab100 
		where doc_no=@doc_no 

		exec FAB120_Work  '0', @doc_no, @doc_dt,@reg_id, @dept_cd   --기표취소
		exec FAB120_Work2 '0', @doc_no, @doc_dt,@reg_id, @dept_cd   --승인취소

	end


--@frm_cd 찾는다
declare	@frm_cd   varchar(10)

select	@frm_cd = z.frm_cd
from	fab100 z
where	z.doc_no = @doc_no



if exists (select 0  
			from	fab100 a 
			where	a.doc_no = @doc_no
			and		a.conf_dt is not null)
begin
	exec PutMessage 'FAB100_02', '이미 승인되어 삭제할 수 없습니다'
	return
end


if exists (select 0  
			from	fab100 a 
			where	a.doc_no = @doc_no
			and		a.apr_dt is not null)
begin
	exec PutMessage 'FAB100_02_1', '이미 원장전기되어 삭제할 수 없습니다.'
	return
end

if exists (select 0  
			from	fab200 a 
			where	a.org_doc_no = @doc_no)
begin
	exec PutMessage'FAB100_03', '이미 반제처리되어 삭제할 수 없습니다 (회계팀에 문의하세요)'
	return
end

declare		@tran_yn	char(1) = case when @@trancount > 0 then '1' else '0' end

BEGIN TRY

	if @tran_yn = '0'
		begin tran



	--전표중계테이블
if [dbo].[fnExists]('FAB400',null) = '1' 
	update	FAB400
	set		doc_no = null
	where	doc_no = @doc_no   

	--AS  INVOICE
if [dbo].[fnExists]('CSA100',null) = '1' 
	begin
		update	CSA100
		set		slip_no = null
		where	slip_no = @doc_no    

		update	CSA100
		set		cancel_slip_no = null
		where	cancel_slip_no = @doc_no   
	end


	--Parts INVOICE
if [dbo].[fnExists]('SPA100',null) = '1' 
	begin
		update	SPA100
		set		slip_no = null
		where	slip_no = @doc_no    

		update	SPA100
		set		cancel_slip_no = null
		where	cancel_slip_no = @doc_no   
 	end

	--AS/Parts Collection  
if [dbo].[fnExists]('CSA120',null) = '1' 
		update	CSA120
		set		slip_no = null
		where	slip_no = @doc_no    

        
	--Purchase Local
if [dbo].[fnExists]('MMA100',null) = '1' 
	update	MMA100
	set		slip_no = null
	where	slip_no = @doc_no   
    
	--General Purchase
if [dbo].[fnExists]('MMA300',null) = '1' 
	update	MMA300
	set		slip_no = null
	where	slip_no = @doc_no   
    
	--Daehan Purchase
if [dbo].[fnExists]('MMD800',null) = '1' 
	update	MMD800
	set		slip_no = null
	where	slip_no = @doc_no   

	--Vehicle Collection
if [dbo].[fnExists]('SDB370',null) = '1' 
	update	SDB370
	set		slip_no = null
	where	slip_no = @doc_no   

	 
	--Vehicle Contract Approval
if [dbo].[fnExists]('SDB100',null) = '1' 
	update	SDB100
	set		cancel_slip_no = null
	where	cancel_slip_no = @doc_no   

	--Vehicle Contract Approval
if [dbo].[fnExists]('SDB100',null) = '1' 
	update	SDB100
	set		slip_no = null
	where	slip_no = @doc_no   

	--T&B Purchase 
if [dbo].[fnExists]('SDF200',null) = '1' 
	update	SDF200
	set		slip_no = null
	where	slip_no = @doc_no   

	--T&B Contract Reg  
if [dbo].[fnExists]('SDR500',null) = '1' 
	update	SDR500
	set		slip_no = null
	where	slip_no = @doc_no   

	--T&B Contract Collection 
if [dbo].[fnExists]('SDR530',null) = '1' 
	update	SDR530
	set		slip_no = null
	where	slip_no = @doc_no   

	--T&B Take In
if [dbo].[fnExists]('SDR220',null) = '1' 
	update	SDR220
	set		slip_no = null
	where	slip_no = @doc_no   

	--입출금전표등록
if [dbo].[fnExists]('FAB170',null) = '1' 
	update	FAB170
	set		doc_no = null
	where	doc_no = @doc_no   

	--입출금전표등록2
if [dbo].[fnExists]('FAE300',null) = '1' 
	update	FAE300
	set		doc_no = null,
	        mapping_slip_no = null
	where	doc_no = @doc_no   

	--T&B Disposal
if [dbo].[fnExists]('SDR600',null) = '1' 
	update	SDR600
	set		slip_no = null
	where	slip_no = @doc_no   


--	--급여전표
if [dbo].[fnExists]('PAB108',null) = '1' 
	update	PAB108 
	set		doc_no = null
	where	doc_no = @doc_no
		


	--기간비용 결산대체
if [dbo].[fnExists]('FAR250',null) = '1' 
	update	FAR250 
	set		doc_no2 = null,
			doc_sq2 = null
	where	doc_no2 = @doc_no  
	  
	--선급이자 결산대체
if [dbo].[fnExists]('FAL110',null) = '1' 
	update	FAL110 
	set		doc_no2 = null,
			doc_sq2 = null
	where	doc_no2 = @doc_no  


	--외화평가
if [dbo].[fnExists]('FAR301',null) = '1' 
	begin
		update	FAR301
		set		doc_no = null	
		where	doc_no = @doc_no  

			--외화평가
		update	FAR301
		set		rvs_doc_no = null		
		where	rvs_doc_no = @doc_no  
	end		
	  
	--미수수익 결산대체
if [dbo].[fnExists]('FAE110',null) = '1' 
	update	FAE110 
	set		doc_no2 = null,
			doc_sq2 = null
	where	doc_no2 = @doc_no    
	  

	--자산취득전표
if [dbo].[fnExists]('FAS100',null) = '1' 
	update	FAS100
	set		doc_no = null,
			doc_sq = null
	where	doc_no = @doc_no   
	and		@frm_cd = 'FAS100'		--고정자산등록의 자산등록분


    --자본적 지출
if [dbo].[fnExists]('FAS110',null) = '1' 
	update	FAS110
	set		doc_no = null,
			doc_sq = null,
			sel_yn = '0'
	where	doc_no = @doc_no 
	and		@frm_cd = 'FAS100_2'		--고정자산등록의 자본적지출등록분


	--자산상각전표
if [dbo].[fnExists]('FAS500',null) = '1' 
	update	FAS500
	set		doc_no = null
	where	doc_no = @doc_no    

  	--자산 폐기 
if [dbo].[fnExists]('FAS120',null) = '1' 
	update	FAS120
	set		doc_no = null,
			sel_yn = '0'
	where	doc_no = @doc_no 
	and     @frm_cd = 'FAS120'	--고정자산모듈에서 매각처리시
 
    --SDB100 매출정보에서 고정자산 매출처리시
 	Delete	FAS120
	where	doc_no = @doc_no 
	and     @frm_cd = 'FAS120_1'
 
 	--고정자산(타계정 대체)
if [dbo].[fnExists]('FAS180',null) = '1' 
	update	FAS180
	set		doc_no = null,
			doc_sq = null,
			chk	 = '0'
	where	doc_no = @doc_no

if [dbo].[fnExists]('FAS180',null) = '1' 
	update	FAS180
	set		fr_doc_no = null,
			fr_doc_sq = null,
			chk	 = '0'
	where	fr_doc_no = @doc_no

	--건설중인자산 취득
if [dbo].[fnExists]('FAS140',null) = '1' 
	update	FAS140
	set		doc_no = null,
			doc_sq = null,
			sel_yn = '0'
	where	doc_no = @doc_no
	and		ISNULL(@frm_cd,'') not in ('FAB110','FAB300')	--전표등록분, CASH IN/OUT 처리가 아닌 것 나머지는 아래에서 처리




	--예금가수금전표
if [dbo].[fnExists]('FAI100',null) = '1' 
	begin
		update	FAI100
		set		doc_no = null,
				doc_sq = null
		where	doc_no = @doc_no    

		update	FAI100
		set		doc_no2 = null,
				doc_sq2 = null
		where	doc_no2 = @doc_no   
	end

	--예금가수금 반제전표
if [dbo].[fnExists]('FAI150',null) = '1' 
		delete  FAI150
		where	doc_no = @doc_no    

	
	--수표발행
if [dbo].[fnExists]('FAG200',null) = '1' 
	begin
		update	FAG200
		set		doc_no = null,
				doc_sq = null
		where	doc_no = @doc_no  

		--수표결제
		update	FAG200
		set		doc2_no = null,
				doc2_sq = null
		where	doc2_no = @doc_no  
	end


	--월년 대체처리 전표 
if [dbo].[fnExists]('FAR350',null) = '1' 
	delete	FAR350
	where	doc_no = @doc_no


	
if [dbo].[fnExists]('FAP100',null) = '1' 
	begin
		--가지급신청전표
		update	FAP100
		set		doc_no = null
		where	doc_no = @doc_no  

		--가지급정산전표' 
		update	FAP100
		set		doc_no2 = null
		where	doc_no2 = @doc_no  
	end

	--선급비용(보험료,임차료) 전표_발생
if [dbo].[fnExists]('FAR500',null) = '1' 
	update	FAR500
	set		doc_no = null,
			adv_stat = 'FI100100'	--등록
	where	doc_no = @doc_no
	and		ISNULL(@frm_cd,'') <> 'FAB110'	--전표등록분 --> FAB300-Save_Koalo_Sub 에서 처리


if [dbo].[fnExists]('FAR510',null) = '1' 
	begin
		--선급비용(보험료) 전표_기간인식
		update	FAR510
		set		exp_doc_no = null
		where	exp_doc_no = @doc_no

		--선급비용 유동성대체
		update	FAR510
		set		liq_doc_no = null
		where	liq_doc_no = @doc_no
	end


	--선수임대료 업데이트_발생
if [dbo].[fnExists]('FAR550',null) = '1' 
	update	FAR550
	set		doc_no = null,
			adv_stat = 'FI100100'	--등록
	where	doc_no = @doc_no
	and		ISNULL(@frm_cd,'') <> 'FAB110'	--전표등록분 --> FAB300-Save_Koalo_Sub 에서 처리

	--선수임대료 업데이트_기간인식
if [dbo].[fnExists]('FAR560',null) = '1' 
	update	FAR560
	set		rec_doc_no = null
	where	rec_doc_no = @doc_no

	--자금조달등록
if [dbo].[fnExists]('FAL100',null) = '1' 
	update	FAL100
	set		doc_no = null,
			doc_sq = null,
			stat_bc = 'FA756100'	--등록
	where	doc_no = @doc_no
	and		ISNULL(@frm_cd,'') <> 'FAB110'		--전표등록분 --> FAB300-Save_Koalo_Sub 에서 처리


	--자금조달 경과이자 업데이트
if [dbo].[fnExists]('FAL111',null) = '1' 
	begin
		update	FAL111
		set		doc_no = null
		where	doc_no = @doc_no

		--자금조달 경과이자 업데이트(역분개)
		update	FAL111
		set		rvs_doc_no = null
		where	rvs_doc_no = @doc_no
	end
	
	--Mr.Ko
if [dbo].[fnExists]('FAP200',null) = '1' 
	begin
		update	FAP200
		set		doc_no = null
		where	doc_no = @doc_no
	end

	
	--자금조달 경과이자 등
if [dbo].[fnExists]('FAL600',null) = '1' 
	begin
		update	FAL600
		set		doc_no = null,
				pay_flag = '0',
				sel_yn = '0'
		where	doc_no = @doc_no

			--자금조달 경과이자 등
		update	FAL600
		set		liq_doc_no = null,
				sel_yn = '0'
		where	liq_doc_no = @doc_no
	end


	--대손충당금 
if [dbo].[fnExists]('FAR711',null) = '1' 
	begin
		update	FAR711
		set		doc_no = null
		where	doc_no = @doc_no
		
		--역분개
		update	FAR711
		set		rvs_doc_no = null,
				rvs_doc_dt = null
		where	rvs_doc_no = @doc_no
	end

	--기초생성전표
if [dbo].[fnExists]('FAB950',null) = '1' 
		update	FAB950
		set		doc_no = null
		where	doc_no = @doc_no 


if exists(select 0
			from FAB100 
			where	doc_no = @doc_no)
begin
	--전표삭제이력
	insert into fab105
			(doc_no, doc_dt, doc_bc, title, title2, co_cd, div_cd, doc_rid, dept_cd, cid, cdt, mid, mdt, 
			did, ddt, dtm)
	select	 doc_no, doc_dt, doc_bc, title, title2, co_cd, div_cd, doc_rid, dept_cd, cid, cdt, mid, mdt, 
			@reg_id, convert(varchar(10), getdate(), 120), getdate()
	from	fab100 
	where	doc_no = @doc_no

	--행삭제이력
	insert into fab106
			(doc_no, doc_sq, acc_cd, dept_cd, dsc, dsc2, pcury_bc, pex_rt, pamt1, pamt2, cury_bc, ex_rt, famt1, famt2, amt1, amt2, tax_no, cid, cdt, mid, mdt)
	select	 doc_no, doc_sq, acc_cd, dept_cd, dsc, dsc2, pcury_bc, pex_rt, pamt1, pamt2, cury_bc, ex_rt, famt1, famt2, amt1, amt2, tax_no, cid, cdt, mid, mdt
	from	fab200 
	where	doc_no = @doc_no


	---------------------------------------
	----고정자산 삭제한다
	--fab300,fab200,fab100  이전에 삭제처리한다.
	---------------------------------------

	declare		@doc_sq	int

	if @frm_cd in ('FAS100_3', 'FAS750','FAJ200')   --타계정대분(고정자산에서 타계정, 건설중인자산에서 타계정, 프로젝트 비용)
	begin
				
			----------------------------------------------------------------
			--고정자산번호 doc_sq를 찾는다
			----------------------------------------------------------------	
			select a.doc_sq
			into	#temp
			from	fab200 a
					join fas100 b on b.doc_no = a.doc_no and b.doc_sq = a.doc_sq
					join fab100 c on c.doc_no = a.doc_no
			where	b.doc_no = @doc_no   
			order by a.doc_sq   

			declare fab110_cur cursor fast_forward local for       

				select doc_sq 
				from #temp
      
			open fab110_cur    
			fetch next from fab110_cur into @doc_sq
      
       
			while @@fetch_status = 0      
			begin  
	
				exec	FAB300_Save_Trans_Asset	'D', @frm_cd, @doc_no, @doc_sq, null, NULL, @reg_id

				fetch next from fab110_cur into @doc_sq
			end      
			close fab110_cur      
			deallocate fab110_cur  
	
			drop table #temp 			
	
	end
	
	if @frm_cd = 'FAB300'   --전표등록분(신규, 자본적지출) --FAB170(FI_Bank_Out, Cash Out 정보)
	begin
	        exec	FAB300_Save_Kolao_Asset	'D', @frm_cd, @doc_no, 1, null, null, @reg_id
	end



	/*
	else if @frm_cd = 'FAB110'   --전표등록분(신규, 자본적지출, 타계정)
	begin

			----------------------------------------------------------------
			--고정자산번호 doc_sq를 찾는다
			----------------------------------------------------------------
	
				select a.*
				into	#temp2	
				from
				(
					--고정자산등록
					select a.doc_sq
					from	fab200 a
							join fas100 b on b.doc_no = a.doc_no and b.doc_sq = a.doc_sq
							join fab100 c on c.doc_no = a.doc_no
					where	b.doc_no = @doc_no   

					union all

					--자본적지출
					select a.doc_sq
					from	fab200 a
							join fas110 b on b.doc_no = a.doc_no and b.doc_sq = a.doc_sq
							join fab100 c on c.doc_no = a.doc_no
					where	b.doc_no = @doc_no 


					union all

					--건술중인자산
					select a.doc_sq
					from	fab200 a
							join fas140 b on b.doc_no = a.doc_no and b.doc_sq = a.doc_sq
							join fab100 c on c.doc_no = a.doc_no
					where	b.doc_no = @doc_no 


					union all
					
					--선급비용(보험료,임차료)
					select a.doc_sq
					from	fab200 a
							join far500 b on b.doc_no = a.doc_no and b.doc_sq = a.doc_sq
							join fab100 c on c.doc_no = a.doc_no
					where	b.doc_no = @doc_no 

					union all
					
					--선급비용(임대료)
					select a.doc_sq
					from	fab200 a
							join far550 b on b.doc_no = a.doc_no and b.doc_sq = a.doc_sq
							join fab100 c on c.doc_no = a.doc_no
					where	b.doc_no = @doc_no 

					union all
					
					--자금조달
					select a.doc_sq
					from	fab200 a
							join FAL100 b on b.doc_no = a.doc_no and b.doc_sq = a.doc_sq
							join fab100 c on c.doc_no = a.doc_no
					where	b.doc_no = @doc_no 
				) a
				order by a.doc_sq  

			declare fab110_cur2 cursor fast_forward local for       

				select doc_sq 
				from #temp2
      
			open fab110_cur2    
			fetch next from fab110_cur2 into @doc_sq
      
       
			while @@fetch_status = 0      
			begin  
	
				exec FAB300_Save_Kolao_Sub	'D', 'FAB110', @doc_no, @doc_sq, null, null, @reg_id

				fetch next from fab110_cur2 into @doc_sq
			end      
			close fab110_cur2      
			deallocate fab110_cur2  
	
			drop table #temp2 
	end	
	*/


	delete from fab300
	where	doc_no = @doc_no

	delete from fab210
	where	doc_no = @doc_no

	delete from fab200
	where	doc_no = @doc_no

	delete from fab161
	where	doc_no = @doc_no

	--전표삭제
	delete from fab100
	where	doc_no = @doc_no

end

	if @tran_yn = '0'
	commit

END TRY
BEGIN CATCH

	if @tran_yn = '0' and @@trancount > 0
		rollback

	declare @err nvarchar(500)
	set @err = ERROR_MESSAGE()

	raiserror (@err, 16, 1)

--    SELECT
--        ERROR_NUMBER() AS ErrorNumber,
--        ERROR_SEVERITY() AS ErrorSeverity,
--        ERROR_STATE() AS ErrorState,
--        ERROR_PROCEDURE() AS ErrorProcedure,
--        ERROR_LINE() AS ErrorLine,
--        ERROR_MESSAGE() AS ErrorMessage;
END CATCH;

```
## Procedure Save
```
USE [ERP]
GO
/****** Object:  StoredProcedure [dbo].[FAB300_Save_Kolao_Asset]    Script Date: 08/06/2024 10:30:23 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


/* ===========================================================================
   내용:	전표 > 고정자산등록, 기간비용등록
   작성일	2017-11-15
   작성자:	PARK
   입력값:	하단참조
   출력값:	
   사용SP:	
   사용Func:
   설  명: 	
   =========================================================================== */


ALTER procedure [dbo].[FAB300_Save_Kolao_Asset]	
					@work_ty		char(1),		--'I', 'U', 'D', 'A'		-> Insert, Update, delete, Approval
					@frm_cd			varchar(10),	--입력구분(Form 별)
					@doc_no			varchar(20),
					@doc_sq			int,
					@doc_no_old		varchar(20),
					@doc_sq_old		int,
					@reg_id			int
                    
as
------------------------------------------------- 필수
set nocount on
set transaction isolation level read uncommitted
-------------------------------------------------

	declare		
			@chg_ty			varchar(10),
			@ast_no			nvarchar(50),
			@prj_no			nvarchar(50),
			@new_ast_no		nvarchar(50),
			@ast_nm			nvarchar(100),
			@prj_nm			nvarchar(100),
			@get_dt			date,
			@qty			decimal(21,5),
			@unit			varchar(10),
			@mng_dept		varchar(20),
			@cost_kd		varchar(10),
			@cust_cd		varchar(30),
			@out_acc_cd		varchar(20)


	declare
			@co_cd			varchar(10),
			@div_cd			varchar(10),
			@std_cd			varchar(10),
			@prj_std_cd		varchar(10),
			@sys_cd			varchar(10),
			@acc_bc			varchar(10),
			@acc_cd			varchar(20),
			@dept_cd		varchar(20),
			@doc_dt			date,
			@apr_dt			date,
			@pcury_bc		varchar(10),
			@pex_rt			decimal(18,5),
			@cury_bc		varchar(10),
			@ex_rt			decimal(18,5),
			@pamt1			decimal(21,5),
			@pamt2			decimal(21,5),
			@amt1			decimal(21,5),
			@amt2			decimal(21,5),
			@famt1			decimal(21,5),
			@famt2			decimal(21,5),
			@grp1_id		int,
			@mid			int


	select	@co_cd = c.co_cd,
			@div_cd = c.div_cd,
			@sys_cd = d.sys_cd, 
			@acc_bc = e.acc_bc,
			@acc_cd = b.acc_cd,
			@dept_cd = b.dept_cd,
			@doc_dt = c.doc_dt,
			@apr_dt = c.apr_dt,
			@pcury_bc = b.pcury_bc, 
			@pex_rt = b.pex_rt,
			@cury_bc = b.cury_bc, 
			@ex_rt = b.ex_rt,
			@pamt1 = b.pamt1, 
			@pamt2 = b.pamt2,
			@amt1 = b.amt1, 
			@amt2 = b.amt2,
			@famt1 = b.famt1, 
			@famt2 = b.famt2,
			@mid = b.mid
	from	FAB200 b
			join		FAB100	c
								on		c.doc_no = b.doc_no
			join		BCC100	d
								on		c.co_cd  = d.co_cd
			left join	FAA100	e							
								on		e.acc_cd = e.acc_cd
	where	b.doc_no = @doc_no
	and		b.doc_sq = @doc_sq

	/*
		--기준계정등록
		F500 : 고정자산1
		F501 : 고정자산2
		F510 : 건설중인자산
	*/

	--고정자산.건설중인자산.경과이자 관련

	select	@std_cd = std_cd 
	from	faa500
	where	std_cd  in ('F500', 'F501', 'F510')
	and		sys_cd	= @sys_cd
	and		@acc_cd between acc_cd and isnull(acc2_cd, acc_cd)




	-- 전표 -> 고정자산 입력
	if @std_cd  in ('F500', 'F501')
	begin

		if not exists (select 0	from fab300 c
						join	faa200 z on z.mng_no = c.mng_no
						where	c.doc_no = @doc_no
						and		c.doc_sq = @doc_sq
						and		z.col_nm = 'chg_ty')	--> 증가구분
		begin	
			return
		end

		select	@chg_ty		= max(case when z.col_nm = 'chg_ty'		then a.mng_val end),
				@ast_no		= max(case when z.col_nm = 'ast_no'		then a.mng_val end),
				@new_ast_no = max(case when z.col_nm = 'new_ast_no' then a.mng_val end),
				@ast_nm		= max(case when z.col_nm = 'ast_nm'		then a.mng_val end),
				@get_dt		= max(case when z.col_nm = 'get_dt'		then a.mng_val end),
				@out_acc_cd	= max(case when z.col_nm = 'acc_cd'		then a.mng_val end),
				@qty		= max(case when z.col_nm = 'qty'		then case when a.mng_val = '' then 0 else convert(decimal(21,5), a.mng_val) end end),
				@unit		= max(case when z.col_nm = 'unit'		then a.mng_val end),
				@mng_dept	= max(case when z.col_nm = 'mng_dept'	then a.mng_val end),
				@cust_cd	= max(case when z.col_nm = 'cust_cd'	then a.mng_val end)
		from 	fab300 a
				join fab200 b
							on	a.doc_no = b.doc_no 
							and	a.doc_sq = b.doc_sq
				join fab100 c
							on	c.doc_no = a.doc_no 
				join faa100 d 
							on  d.acc_cd  = b.acc_cd
							and d.sys_cd  = @sys_cd
							and d.acc_bc  = 'FA200100'	--차변인 경우에만
				join faa200 z
							on  z.mng_no = a.mng_no	
		where 	a.doc_no = @doc_no 
		and		a.doc_sq = @doc_sq
		group by a.doc_no, a.doc_sq

		if isnull(@chg_ty, '') = ''
		begin	
			return
		end	

		--취득계정 존재여부
		if not exists ( select 0
						from	fas900 z
						where	z.co_cd = @co_cd
						and		z.get_acc_cd = @acc_cd)
		begin
			exec PutMessage 'FAS510_05', '취득계정코드가 정의되지 않았습니다.'
			return
		end
		 
		--감가상각 처리된 월 체크
		if exists ( select 0
						from	fas500
						where 	de_mon = dbo.fnMonth(@get_dt)
	 					and		div_cd = @div_cd) 
		begin
			exec PutMessage 'FAS100_04', '감가상각정보가 등록되어 있습니다.'
			return
		end

				--자산삭제시 적용
		if @work_ty = 'D' 
		begin
				 if @chg_ty = 'FA832200'	--자본적지출
				 begin
						--exec fas100_delete	@ast_no, @reg_id
						delete	a
						from	fas110 a
						where	a.doc_no = @doc_no
						and		a.doc_sq = isnull(@doc_sq, a.doc_sq)

				 end
				 else if @chg_ty ='FA832100'		--> 신규
				 begin
						exec fas100_delete	@ast_no, @reg_id
				 end
			return
		end	

		--------------------------------------------------
		--전표등록에서 처리한 자료는 일단 삭제 후 처리한다
		--------------------------------------------------
		if @work_ty = 'U' 
		begin

			--exec fas100_Check @doc_no, @reg_id
			delete	a
			from	FAS110 a
			where	a.doc_no = @doc_no_old
			and		a.doc_sq = @doc_sq_old


			--부서별 비율
			delete	a
			from	FAS160 a
					join fas100	b on b.ast_no = a.ast_no
			where	a.ast_no = @new_ast_no	


			--자산항목
			delete	a
			from	fas100 a
			where	a.doc_no = @doc_no
			and		a.doc_sq = isnull(@doc_sq, a.doc_sq)

		end


		----자산삭제시 적용
		--if @work_ty = 'D' 
		--begin
		--		 if @chg_ty = 'FA832200'	--자본적지출
		--		 begin
		--				--exec fas100_delete	@ast_no, @reg_id
		--				delete	a
		--				from	fas110 a
		--				where	a.doc_no = @doc_no
		--				and		a.doc_sq = isnull(@doc_sq, a.doc_sq)

		--		 end
		--		 else if @chg_ty ='FA832100'		--> 신규
		--		 begin
		--				exec fas100_delete	@new_ast_no, @reg_id
		--		 end
		--	return
		--end	


		if @chg_ty ='FA832100'		--> 신규
		begin
			
			if @work_ty = 'I'
			begin

					if exists (select 0 from	fas100
								where	doc_no = @doc_no
								and		doc_sq = @doc_sq)
					begin
						exec PutMessage 'FAS100_21', '이미 동일한 자산번호가 존재합니다.'
						return
					end

					--if exists (select 0 from	fas100
					--			where	doc_no = @doc_no
					--			and		doc_sq = @doc_sq)
					--begin
					--	exec PutMessage 'FAS100_22', '이미 동일한 전표번호,순번을 가진 자산이 존재합니다.'
					--	return
					--end


					select	@grp1_id = z.grp1_id
					from	fas900 z
					where	z.co_cd = @co_cd
					and		z.get_acc_cd = @acc_cd

					set @new_ast_no = dbo.fnAssetNo ('FAS100',@grp1_id, @div_cd)


					insert into fas100
							(ast_no, org_ast_no,
							ast_nm, doc_stat_bc, qty, unit, div_cd, co_cd, cust_cd, tran_bc,
							grp1_id,  mng_dept, get_dt, get_famt, get_amt, vat_amt, cury_bc, ex_rt, 
							life_year, fr_mon, 
							to_mon, 
							de_bc, de_rt, 
							prog_bc, 
							doc_no, doc_sq, resd_amt,
							mid, mdt, cid, cdt)

					select	@new_ast_no, @new_ast_no,
							@ast_nm, 'FA833100', @qty, @unit, @div_cd, @co_cd, @cust_cd, 'FA832100',	--(FA833100 : 전표저장상태) ,FA832100 ; 대체구분-신규
							z.grp1_id,  @mng_dept, @get_dt, @famt1, @amt1, 0,  @cury_bc, @ex_rt,
							z.life_year, dbo.fnMonth(@get_dt), 
							dbo.fnMonth(DateAdd(Month, -1, DateAdd(Month, isnull(z.life_year,0) * 12, @get_dt))),	--상각종료월
							z.de_bc, z.std_de_rt, 
							case when z.life_year > 0 then
									'FA830100'	--상각중
							else	
									'FA830500'	--상각안함
							end,
							@doc_no, @doc_sq, 0,	--잔존가액은 일단 0
							@reg_id, getdate(),@reg_id, getdate()
					from	fas900 z
					where	z.co_cd = @co_cd
					and		z.get_acc_cd = @acc_cd


					update	fab300 
					set		mng_val = @new_ast_no,
					        mng_dsc = @ast_nm
					where	doc_no = @doc_no	
					and		doc_sq = @doc_sq
					and		mng_no in ('530','700')

			end


			if @work_ty = 'U'
			begin
					--업데이트 처리시 자산정보가 없으면 Insert (자본적 지출 --> 신규 전환시)
					if not exists (select 0 from fas100
									where doc_no = @doc_no_old
									and	 doc_sq = @doc_sq_old)
					begin

							select	@grp1_id = z.grp1_id
							from	fas900 z
							where	z.co_cd = @co_cd
							and		z.get_acc_cd = @acc_cd

							set @new_ast_no = dbo.fnAssetNo2 ('FAS100_1',@grp1_id, @div_cd)


							insert into fas100
									(ast_no, org_ast_no,
									ast_nm, doc_stat_bc, qty, unit, div_cd, co_cd, cust_cd, tran_bc,
									grp1_id,  mng_dept, get_dt, get_famt, get_amt, vat_amt, cury_bc, ex_rt, 
									life_year, fr_mon, 
									to_mon, 
									de_bc, de_rt, 
									prog_bc, 
									doc_no, doc_sq, resd_amt,
									mid, mdt, cid, cdt)

							select	@new_ast_no, @new_ast_no,
									@ast_nm, 'FA833100', @qty, @unit, @div_cd, @co_cd, @cust_cd, 'FA832100',	--(FA833100 : 전표저장상태) ,FA832100 ; 대체구분-신규
									z.grp1_id,  @mng_dept, @get_dt, @famt1, @amt1, 0,  @cury_bc, @ex_rt,
									z.life_year, dbo.fnMonth(@get_dt), 
									dbo.fnMonth(DateAdd(Month, -1, DateAdd(Month, isnull(z.life_year,0) * 12, @get_dt))),	--상각종료월
									z.de_bc, z.std_de_rt, 
									case when z.life_year > 0 then
											'FA830100'	--상각중
									else	
											'FA830500'	--상각안함
									end,
									@doc_no, @doc_sq, 0,	--잔존가액은 일단 0
									@reg_id, getdate(),@reg_id, getdate()
							from	fas900 z
							where	z.co_cd = @co_cd
							and		z.get_acc_cd = @acc_cd


							update	fab300 
							set		mng_val = @new_ast_no
							where	doc_no = @doc_no	
							and		doc_sq = @doc_sq
							and		mng_no in ('530','700')

					end
					else
					begin
							update	a
							set		a.ast_no = @new_ast_no, a.ast_nm = @ast_nm, a.tran_bc = 'FA832100',
									a.qty = @qty, a.unit = @unit, a.div_cd = @div_cd, a.co_cd = @co_cd, a.cust_cd = @cust_cd,
									a.mng_dept = @mng_dept, a.get_dt = @get_dt, a.get_famt = @famt1, 
									a.get_amt = @amt1, a.cury_bc = @cury_bc, a.ex_rt = @ex_rt, 
									fr_mon = dbo.fnMonth(@get_dt), 
									to_mon = dbo.fnMonth(DateAdd(Month, -1, DateAdd(Month, isnull(z.life_year,0) * 12, @get_dt))),
									life_year = z.life_year, de_bc = z.de_bc, de_rt = z.std_de_rt, 
									prog_bc = case when z.life_year > 0 then 
											'FA830100'	--상각중
									else
											'FA830500'	--상각안함
									end,
									doc_no = @doc_no, doc_sq = @doc_sq,
									a.mid = @reg_id, a.mdt = getdate()
							from	fas100 a
									join fas900 z	on		z.co_cd = @co_cd
													and		z.get_acc_cd = @acc_cd
							where	a.doc_no = @doc_no
							and		a.doc_sq = @doc_sq
					end
			end

		end


		if @chg_ty = 'FA832200'		-- 자본적지출
		begin

				if @work_ty = 'I' 
				begin

					insert into fas110
							(ast_no, sq_no, doc_stat_bc,
							iss_dt, cury_bc, ex_rt, chg_famt, iss_amt, vat_amt, --docu, rmks, mng_rid, 
							cust_cd, doc_no, doc_sq, cid, cdt, mid, mdt)
					select	@ast_no, 
							(select isnull(max(sq_no),0) + 1 from fas110 where ast_no = @ast_no),
							'FA833100',
							@get_dt, @cury_bc, @ex_rt, @famt1, @amt1, 0,--round(@amt1 * 0.1,0,1),
							@cust_cd, @doc_no, @doc_sq, @reg_id, getdate(), @reg_id, getdate()
				end


				if @work_ty = 'U'
				begin
					--자본적지출이 없으면 Insert (신규 --> 자본적지출 변경시)
					if not exists (select 0 from fas110
								   where doc_no = @doc_no_old
								   and	 doc_sq = @doc_sq_old)
					begin

						insert into fas110
								(ast_no, sq_no, doc_stat_bc,
								iss_dt, cury_bc, ex_rt, chg_famt, iss_amt, vat_amt, --docu, rmks, mng_rid, 
								cust_cd, doc_no, doc_sq, cid, cdt, mid, mdt)
						select	@ast_no, 
								(select isnull(max(sq_no),0) + 1 from fas110 where ast_no = @ast_no),
								'FA833100',
								@get_dt, @cury_bc, @ex_rt, @famt1, @amt1, 0,--round(@amt1 * 0.1,0,1),
								@cust_cd, @doc_no, @doc_sq, @reg_id, getdate(), @reg_id, getdate()
					end
					else
					begin

						update	a
						set		a.ast_no = @ast_no, 
								a.iss_dt = @get_dt, a.cury_bc = @cury_bc, ex_rt = @ex_rt, chg_famt = @famt1, iss_amt = @amt1, 
								--vat_amt, --docu, rmks, mng_rid, 
								cust_cd = @cust_cd, doc_no = @doc_no, doc_sq = @doc_sq, mid = @mid, mdt = getdate()
						from	fas110 a
						where	a.doc_no = @doc_no_old
						and		a.doc_sq = @doc_sq_old
					end

				end

		end

	end


	-- 건설중인 자산 입력
	-- select * from fas140
	else if @std_cd  in ('F510')
	begin
			if not exists (select 0	from fab300 c
									join	faa200 z on z.mng_no = c.mng_no
							where	c.doc_no = @doc_no
							and		c.doc_sq = @doc_sq
							and		z.col_nm = 'chg_ty')	--> 증가구분
			begin	
				return
			end


			select	@chg_ty		  = max(case when z.col_nm = 'chg_ty'		  then a.mng_val end),
					@prj_no		  = max(case when z.col_nm = 'prj_no'		  then a.mng_val end),
					@prj_nm		  = max(case when z.col_nm = 'ast_nm'		  then a.mng_dsc end),
					@get_dt		  = max(case when z.col_nm = 'get_dt'		  then a.mng_val end),
					@acc_cd		  = max(case when z.col_nm = 'acc_cd'		  then a.mng_val end),
					@qty		  = max(case when z.col_nm = 'qty'		      then case when a.mng_val = '' then 0 else convert(decimal(21,5), a.mng_val) end end),
					@unit		  = max(case when z.col_nm = 'unit'		      then a.mng_val end),
					@mng_dept	  = max(case when z.col_nm = 'mng_dept'	      then a.mng_val end),
					@cust_cd	  = max(case when z.col_nm = 'cust_cd'	      then a.mng_val end)
			from 	fab300 a
					join fab200 b
								on	a.doc_no = b.doc_no 
								and	a.doc_sq = b.doc_sq
					join fab100 c
								on	c.doc_no = a.doc_no
					join faa100 d 
								on  d.acc_cd  = b.acc_cd
								and d.sys_cd  = @sys_cd
								and d.acc_bc  = 'FA200100'	--차변인 경우에만								 
					join faa200 z
								on  z.mng_no = a.mng_no
			where 	a.doc_no = @doc_no 
			and		a.doc_sq = @doc_sq
			group by a.doc_no, a.doc_sq

			if isnull(@chg_ty, '') = ''
			begin	
				return
			end	


			if @chg_ty <> 'FA832100'	--신규가 아니면 체크
			begin	
				exec PutMessage  'FAS750_02', '신규등록만 선택할 수 있습니다.'
				return
			end


			--건설중인 자산 프로젝트 체크
			if not exists ( select 0
							from	fas100
							where 	ast_no = @prj_no)
			begin
				exec PutMessage 'FAS750_01', '건설중인자산에 해당 프로젝트가 존재하지 않습니다.'
				return
			end

			--자산삭제시 적용
			--select * from fas140
			if @work_ty = 'D' 
			begin
					delete	a
					from	FAS140 a
					where	a.doc_no = @doc_no
					and		a.doc_sq = @doc_sq

				return
			end	


			if @chg_ty in  ('FA832100')		--> 신규
			begin
			
					if @work_ty = 'I'
					begin

							if exists (select 0 from	fas140
										where	doc_no = @doc_no
										and		doc_sq = @doc_sq)
							begin
								exec PutMessage 'FAS100_21', '이미 동일한 자산번호가 존재합니다.'
								return
							end

							declare	@_sq_no int = 1
							if exists (select 0 from fas140 where ast_no = @prj_no)
							begin
								select @_sq_no = isnull(max(sq_no),0) + 1 from fas140 where ast_no = @prj_no
							end


							insert into fas140
										(ast_no, sq_no, doc_stat_bc, dsc, 
										iss_dt, cury_bc, ex_rt, iss_famt, iss_amt, vat_amt, tax_bc, cust_cd, mng_rid, docu,
										 doc_no, doc_sq, cid,cdt, mid, mdt)

							select      @prj_no, @_sq_no, 'FA833100', 'Asset Under Construction Acquisition', 
										@get_dt, @cury_bc, @ex_rt, @famt1, @amt1, 0, null, @cust_cd, @reg_id, null,
										@doc_no, @doc_sq, @reg_id, getdate(),@reg_id, getdate()

					end

					if @work_ty = 'U' 
					begin
						update	a
						set		a.ast_no   = @prj_no,
								a.dsc      = 'Asset Under Construction Acquisition',
								a.iss_dt   = @get_dt,
								a.cury_bc  = @cury_bc,
								a.ex_rt    = @ex_rt,
								a.cust_cd  = @cust_cd, 
								a.iss_famt = @famt1, 
								a.iss_amt  = @amt1, 
								a.mid      = @reg_id, 
								a.mdt      = getdate()
						from	FAS140 a
						where	a.doc_no = @doc_no_old
						and		a.doc_sq = @doc_sq_old
					end

					----자산삭제시 적용
					--if @work_ty = 'D' 
					--begin
					--		delete	a
					--		from	FAS140 a
					--		where	a.doc_no = @doc_no
					--		and		a.doc_sq = @doc_sq

					--	return
					--end	
			 end
	end

/*
	-- 기간비용등록(보험료, 임차료)
	else if @std_cd  in ('R300') and @frm_cd = 'FAB300'
	begin
			
		if not exists (select 0	from fab300 c
						join	faa200 z on z.mng_no = c.mng_no
						where	c.doc_no = @doc_no
						and		c.doc_sq = @doc_sq
						and		z.col_nm = 'exp_cd')	--> 비용구분(기간비용)
		begin	
			return
		end


		select	@exp_cd		= max(case when z.col_nm = 'exp_cd'		then a.mng_val end),
				@exp_acc_cd	= max(case when z.col_nm = 'exp_acc_cd'	then a.mng_val end),
				@fr_dt		= max(case when z.col_nm = 'fr_dt'		then a.mng_val end),
				@to_dt		= max(case when z.col_nm = 'to_dt'		then a.mng_val end),
				@dsc		= max(case when z.col_nm = 'dsc'		then a.mng_val end),
				@cust_cd	= max(case when z.col_nm = 'cust_cd'	then a.mng_val end)
		from 	fab300 a
				join fab200 b
							on	a.doc_no = b.doc_no 
							and	a.doc_sq = b.doc_sq
				join fab100 c
							on	c.doc_no = a.doc_no 
				join faa100 d 
							on  d.acc_cd  = b.acc_cd
							and d.sys_cd  = @sys_cd
							and d.acc_bc  = 'FA200100'	--차변인 경우에만
				join faa200 z
							on  z.mng_no = a.mng_no

		where 	a.doc_no = @doc_no 
		and		a.doc_sq = @doc_sq
		group by a.doc_no, a.doc_sq

		if isnull(@exp_cd, '') = ''
		begin	
			return
		end	

	
		--선급비용계정 설정 존재여부
		if not exists ( select 0
						from	faa500 z
						where	std_cd  in ('R300')
						and		sys_cd	= @sys_cd)
		begin
			exec PutMessage 'FAR500_01', '선급비용계정이 기준계정등록 메뉴에 없습니다.'
			return
		end
 

		if exists (select 0 from FAR510 a
							join far500 b on b.adv_no = a.adv_no
				   where	b.doc_no = @doc_no
				   and		b.doc_sq = @doc_sq)
		begin
			exec PutMessage 'FAR510_03', '이미 기간인식이 진행된 자료입니다.'
		end


		if @exp_cd in  ('FI110100','FI110200')		--> 보험료, 임차료(유동성대체는 제외)
		begin
			
			if @work_ty = 'I'
			begin

					--if exists (select 0 from	FAR500
					--			where	doc_no = @doc_no
					--			and		doc_sq = @doc_sq)
					--begin
					--	exec PutMessage 'FAS100_21', '이미 동일한 자산번호가 존재합니다.'
					--	return
					--end


					select  @adv_no = [dbo].[fnCodeNo]( 'FAR500', convert(varchar(10), GETDATE(), 121))

			
					insert into FAR500
							(adv_no, co_cd, div_cd, exp_cd, doc_dt, doc_no, doc_sq, fr_dt, to_dt, adv_stat, 
							pre_acc_cd, exp_acc_cd, exp_dept_cd, cust_cd, cury_bc, famt, ex_rt, amt, 
							dsc, doc_cdt, cid, cdt, mid, mdt)
					select	@adv_no, @co_cd, @div_cd, @exp_cd, @doc_dt, @doc_no, @doc_sq, @fr_dt, @to_dt, 'FI100200', 
							@acc_cd, @exp_acc_cd, @dept_cd, @cust_cd, @cury_bc, @famt1, @ex_rt, @amt1, 
							@dsc, @doc_dt, @reg_id, getdate(),@reg_id, getdate()									
			end

			if @work_ty = 'U' 
			begin
				update	a
				set		a.adv_no = @adv_no, a.co_cd = @co_cd, a.div_cd = @div_cd, 
						a.exp_cd = @exp_cd, a.doc_dt = @doc_dt, 
						a.doc_no = @doc_no, a.doc_sq = @doc_sq, a.fr_dt = @fr_dt, a.to_dt = @to_dt, 
						a.exp_acc_cd = @exp_acc_cd, a.pre_acc_cd = @acc_cd, 
						a.exp_dept_cd = @dept_cd, a.cust_cd = @cust_cd, 
						a.cury_bc = @cury_bc, a.famt = @famt1, a.ex_rt = @ex_rt, a.amt = @amt1, 
						a.dsc = @dsc, a.doc_cdt = @doc_dt,
						a.mid = @reg_id, a.mdt = getdate()
				from	FAR500 a
				where	a.doc_no = @doc_no_old
				and		a.doc_sq = @doc_sq_old
			end

			--삭제시 적용
			if @work_ty = 'D' 
			begin
					delete	a
					from	far500 a
					where	a.doc_no = @doc_no
					and		a.doc_sq = @doc_sq
				return
			end	
		end

	end

	-- 기간비용등록(임대료)
	else if @std_cd  in ('R400')
	begin
			
		if not exists (select 0	from fab300 c
						join	faa200 z on z.mng_no = c.mng_no
						where	c.doc_no = @doc_no
						and		c.doc_sq = @doc_sq
						and		z.col_nm = 'rec_cd')	--> 비용구분(기간비용)
		begin	
			return
		end


		select	@rec_cd		= max(case when z.col_nm = 'rec_cd'			then a.mng_val end),
				@rec_dt		= max(case when z.col_nm = 'rec_dt'			then a.mng_val end),
				@rec_acc_cd = max(case when z.col_nm = 'exp acc_cd'		then a.mng_val end),
				@recacc_famt= max(case when z.col_nm = 'recacc_famt'	then case when a.mng_val = '' then 0 else convert(decimal(21,5), a.mng_val) end end),
				@fr_dt		= max(case when z.col_nm = 'fr_dt'			then a.mng_val end),
				@to_dt		= max(case when z.col_nm = 'to_dt'			then a.mng_val end),
				@dsc		= max(case when z.col_nm = 'dsc'			then a.mng_val end),
				@cust_cd	= max(case when z.col_nm = 'cust_cd'		then a.mng_val end)
		from 	fab300 a
				join fab200 b
							on	a.doc_no = b.doc_no 
							and	a.doc_sq = b.doc_sq
				join fab100 c
							on	c.doc_no = a.doc_no 
				join faa100 d 
							on  d.acc_cd  = b.acc_cd
							and d.sys_cd  = @sys_cd
							and d.acc_bc  = 'FA200200'	--대변인 경우에만(선수임대료)
				join faa200 z
							on  z.mng_no = a.mng_no
		where 	a.doc_no = @doc_no 
		and		a.doc_sq = @doc_sq
		group by a.doc_no, a.doc_sq

	
		if isnull(@rec_cd, '') = ''
		begin	
			return
		end	


		--선급비용계정 설정 존재여부
		if not exists ( select 0
						from	faa500 z
						where	std_cd  in ('R400')
						and		sys_cd	= @sys_cd)
		begin
			exec PutMessage 'FAR500_01', '선급비용계정이 기준계정등록 메뉴에 없습니다.'
			return
		end
 

		if exists (select 0 from FAR560 a
							join far550 b on b.adv_no = a.adv_no
				   where	b.doc_no = @doc_no
				   and		b.doc_sq = @doc_sq)
		begin
			exec PutMessage 'FAR510_03', '이미 기간인식이 진행된 자료입니다.'
		end


		if @rec_cd in  ('FI200100')		--> 임대료
		begin
			
			if @work_ty = 'I'
			begin

					select  @adv_no = [dbo].[fnCodeNo]( 'FAR550', convert(varchar(10), GETDATE(), 121))

			
					insert into FAR550
							(adv_no, co_cd, div_cd, rec_cd, rec_dt, doc_dt, doc_no, doc_sq, fr_dt, to_dt, adv_stat, 
							pre_acc_cd, rec_acc_cd, rec_dept_cd, cust_cd, cury_bc, famt, ex_rt, amt, 
							dsc, recacc_famt, doc_cdt, cid, cdt, mid, mdt)
					select	@adv_no, @co_cd, @div_cd, @rec_cd, @rec_dt, @doc_dt, @doc_no, @doc_sq, @fr_dt, @to_dt, 'FI100200', 
							@acc_cd, @rec_acc_cd, @dept_cd, @cust_cd, @cury_bc, @famt1, @ex_rt, @amt1, 
							@dsc, @recacc_famt, @doc_dt, @reg_id, getdate(),@reg_id, getdate()									
			end

			if @work_ty = 'U'	
			begin
				update	a
				set		a.adv_no = @adv_no, a.co_cd = @co_cd, a.div_cd = @div_cd, 
						a.rec_cd = @rec_cd, a.doc_dt = @doc_dt, a.rec_dt = @rec_dt,
						a.doc_no = @doc_no, a.doc_sq = @doc_sq, a.fr_dt = @fr_dt, a.to_dt = @to_dt, 
						a.rec_acc_cd = @rec_acc_cd, a.pre_acc_cd = @acc_cd,  a.recacc_famt = @recacc_famt,
						a.rec_dept_cd = @dept_cd, a.cust_cd = @cust_cd, 
						a.cury_bc = @cury_bc, a.famt = @famt1, a.ex_rt = @ex_rt, a.amt = @amt1, 
						a.dsc = @dsc, a.doc_cdt = @doc_dt,
						a.mid = @reg_id, a.mdt = getdate()
				from	FAR550 a
				where	a.doc_no = @doc_no_old
				and		a.doc_sq = @doc_sq_old
			end


			--삭제시 적용
			if @work_ty = 'D' 
			begin
					delete	a
					from	far550 a
					where	a.doc_no = @doc_no
					and		a.doc_sq = @doc_sq
				return
			end	
		end

	end

*/
```
## 
```
```
## Procedure Calculate Fee
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_calculate_vat_free]    Script Date: 08/06/2024 10:34:45 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		<Author,,Name>
-- Create date: <Create Date,,>
-- Description:	<Description,,>
-- =============================================
ALTER PROCEDURE [dbo].[sp_calculate_vat_free]
	(
	@inv_no nvarchar(30),
	@cust_dealer nvarchar(20),
	@roundid nvarchar(20)
	)
AS
BEGIN

   Declare @amt decimal(18, 0) = (select isnull(receive_amt,0) from LIV100 where inv_no =@inv_no and cust_dealer =@cust_dealer and roundid =@roundid and receive_amt > 0 )
   Declare @fee_amt decimal(18, 0) = case when (isnull(@amt,0)) between 1 and 2000000.9 then 1000
             when (isnull(@amt,0)) between 2000001 and 3000000.9 then 1500
             when (isnull(@amt,0)) between 3000001 and 4000000.9  then 2500
             when (isnull(@amt,0)) between 4000001 and 5000000.9 then 3500
             when (isnull(@amt,0)) between 5000001 and 7000000.9 then 4500
             when (isnull(@amt,0)) between 7000001 and 10000000.9 then 7500
             when (isnull(@amt,0)) between 10000001 and 30000000.9 then 12000
             when (isnull(@amt,0)) between 30000001 and 50000000.9 then 15500
             when (isnull(@amt,0)) between 50000001 and 100000000.9 then 20000
             when (isnull(@amt,0)) between 100000001 and 120000000.9 then 25000
             when (isnull(@amt,0)) between 120000001 and 150000000.9 then 30000
             when (isnull(@amt,0)) >= 150000001 then 40000 else 0
        end 

   Update LIV100 set transfer_fee = @fee_amt where inv_no =@inv_no and cust_dealer =@cust_dealer and roundid =@roundid and cust_dealer not in 
   ('BcelOne','IBCools','LaoQR','LottoPoints','MMoney','UMoney' )                             
  
 

 end

```
## Procedure Check Call back
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_check_calblack_invoice_ib]    Script Date: 08/06/2024 10:35:41 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
--exec dbo.sp_appr_plan @co_cd,@req_no,@emp_no,@coments
ALTER procedure [dbo].[sp_check_calblack_invoice_ib] 
       (
         @terminalLabel  nvarchar(150)
       )
as 
begin

  if exists (select 1 from liv110 where terminalLabel=@terminalLabel and txnStatus='1' and message='success')
    begin 
      select instId, txnAmount, txnRefId, additionalInfo, paymentAccount, paymentAccountCcy, paymentAccountName, 
        callbackRegDate, feeAmount, txnStatus, message, storeLabel, terminalLabel
      from liv110 
      where terminalLabel=@terminalLabel and txnStatus='1' and message='success'
      --select code='SC01',resual='PAYMENT SUCCESS'
    end
  else 
    begin 
      select instId='', txnAmount=0, txnRefId='', additionalInfo='', paymentAccount='', paymentAccountCcy='', paymentAccountName='', 
        callbackRegDate=getdate(), feeAmount=0, txnStatus='0', message='failed', storeLabel='', terminalLabel=''
      --select code='NF01',resual='PAYMENT NOT FOUND'
  end
end
  

```
## Procedure issue invoice
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_check_issue_invoice]    Script Date: 08/06/2024 10:36:11 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


ALTER PROCEDURE [dbo].[sp_check_issue_invoice]	
      
	     @cust_dealer varchar(20)
				
AS   
 begin
  
  if exists (select 1 from liv100 where cust_dealer=@cust_dealer and roundid=(select MAX (roundId) from LTM300 where dt <=getdate()))
    select code=200, status='Issue Invoiced'
  else 
    select code=404, status='Issue Invoiced not yet'
   
 end  
```
## Procedure Login APP
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_dealer_login_app]    Script Date: 08/06/2024 10:37:11 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
--exec dbo.sp_appr_plan @co_cd,@req_no,@emp_no,@coments
ALTER procedure [dbo].[sp_dealer_login_app] 
       (@user_id  nvarchar(100),
        @user_pwd nvarchar(250)
       )
as 
begin
if not exists(select 1 from LTC100 where cust_cd=@user_id and convert(nvarchar(4000),DECRYPTBYPASSPHRASE('Key',pass))=@user_pwd
              union all
              select 1 from erp.dbo.scu100 where emp_no=@user_id and pwd= erp.dbo.fnPwdEncrypt(@user_pwd) and use_yn ='1'
   --and pwd=dbo.fnPwdEncrypt(@user_pwd)
 )
 begin 
  print ('User name or password wrong')
  return;
end 

select user_id=cust_cd,user_nm=cust_nm,user_lnm=last_nm,tel_p=LEFT (tel1, 3),tel1=substring(tel1,4, 8),
       tel_p2=LEFT (tel2, 3),tel2=substring(tel2,4, 8),gen_bc, gen_nm=case when gen_bc='M' then 'Mis' when gen_bc='F' then 'Mr' else '' end,
       birth_dt,nat_bc, nat_nm=KPlaza.dbo.fcCodeName(nat_bc),cust_ty, cust_ty_nm=KPlaza.dbo.fcCodeName(cust_ty),
       email,addr1,addr2,province,province_nm=KPlaza.dbo.fcCodeName(province),
       distic, distic_nm=KPlaza.dbo.fcCodeName(distic),cust_cls, cust_cls_nm=KPlaza.dbo.fcCodeName(cust_cls),
       sub_ty=sp_ty, sub_ty_nm=KPlaza.dbo.fcCodeName(sp_ty),bank_acc=sp_acc,
       approve_transfer='0', invoice_dealer=case when cust_ty='KPC200140' and cust_cls='KPCT100300' then '1' else '0' end, 
       invoice_own=case when cust_ty='KPC200160' then '1' else '0' end, pay_wining=case when cust_ty='KPC200140' then '1' else '0' end, 
       pay_wining_his=case when cust_ty='KPC200140' then '1' else '0' end, saller_info=case when cust_ty='KPC200140' then '1' else '0' end
  from LTC100
where 1=1
 and cust_cd=@user_id
 and convert(nvarchar(4000),DECRYPTBYPASSPHRASE('Key',pass))=@user_pwd

 union all

 select user_id=a.emp_no,user_nm=emp_nm,user_lnm='',tel_p='',tel1=mobile,
       tel_p2='',tel2='',gen_bc='', gen_nm='',
       birth_dt=birthday,nat_bc=nat_cd, nat_nm=nation,cust_ty='KPC200140', cust_ty_nm='',
       email,addr1=address,addr2='',province,province_nm=KPlaza.dbo.fcCodeName(province),
       distic=district, distic_nm=erp.dbo.fccodename(a.district),cust_cls='KPCT100130', cust_cls_nm=KPlaza.dbo.fcCodeName('KPCT100130'),
       sub_ty='', sub_ty_nm='',bank_acc='', 
       approve_transfer=isnull(c.approve_transfer,'0'), invoice_dealer=isnull(c.invoice_dealer,'0'), invoice_own=isnull(c.invoice_own,'0'), 
       pay_wining=isnull(c.pay_wining,'0'), pay_wining_his=isnull(c.pay_wining_his,'0'), saller_info=isnull(c.saller_info,'0')
 from [ERP95].[ERP].[dbo].uv_staff a
 join erp.dbo.scu100 b on a.emp_no=b.emp_no
 join ltc110 c on a.emp_no=c.user_id
 where a.emp_no=@user_id 
   and b.use_yn ='1'
   and b.pwd=erp.dbo.fnPwdEncrypt(@user_pwd)

end
  
--declare @decryptedValue nvarchar(4000)
--declare @encryptedValue varbinary(8000)

--SET @encryptedValue  = ENCRYPTBYPASSPHRASE('Key',N'1234') 
--Set @decryptedValue = DECRYPTBYPASSPHRASE('Key',@encryptedValue)

--select @encryptedValue , @decryptedValue

--select convert(nvarchar(4000),DECRYPTBYPASSPHRASE('Key',pass)),*
--from ltc100 
--where cust_cd='KVTE017'
--
--update ltc100 set pass=ENCRYPTBYPASSPHRASE('Key',N'1234') where cust_cd='KVTE071'



```
## Procedure update Password
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_dealer_update_pass]    Script Date: 08/06/2024 10:37:48 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
--exec dbo.sp_appr_plan @co_cd,@req_no,@emp_no,@coments
ALTER procedure [dbo].[sp_dealer_update_pass] 
       (@user_id  nvarchar(100),
        @old_pwd nvarchar(250),
        @new_pwd nvarchar(250)
       )
as 
begin

 update a
   set a.pass=ENCRYPTBYPASSPHRASE('Key',@new_pwd)
 from ltc100 a
where 1=1
 and cust_cd=@user_id
 and convert(nvarchar(4000),DECRYPTBYPASSPHRASE('Key',pass))=@old_pwd
end
  
--declare @decryptedValue nvarchar(4000)
--declare @encryptedValue varbinary(8000)

--SET @encryptedValue  = ENCRYPTBYPASSPHRASE('Key',N'1234') 
--Set @decryptedValue = DECRYPTBYPASSPHRASE('Key',@encryptedValue)

--select @encryptedValue , @decryptedValue

--select convert(nvarchar(4000),DECRYPTBYPASSPHRASE('Key',password)),*
--from tl200 
--where cust_no='lar@kolaogroup.com'
--
--update tl200 set password=ENCRYPTBYPASSPHRASE('Key',N'1234') where cust_no='maxsoudpouk@gmail.com'

```
## Procedure notification list
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_get_notification_list]    Script Date: 08/06/2024 10:38:12 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
---old use --
ALTER procedure [dbo].[sp_get_notification_list] 
       (
         @user_id varchar(50)
       )
as 
begin

  select distinct a.inv_no, a.cust_dealer,round_id, a.appr_status
  from lco410 a,ltc110 b join ltc120 c on b.user_id=c.user_id 
  where b.approve_transfer ='1' 
  --and isnull(a.appr_status,'0') ='0'
  and b.user_id=@user_id

end
  
```
## Procedure sent Notification list
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_get_sending_notification_list]    Script Date: 08/06/2024 10:39:17 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
---old use --
ALTER procedure [dbo].[sp_get_sending_notification_list] 
as 
begin

  --if exists (select 1 from lco410 where isnull(appr_status,'0') ='0')
  --  begin 
      select fcm_token from ltc120 where user_id in ('K2211024','K1709016')
  --  end
  --else 
  --begin 
  --  select fcm_token=''
  --end


end
  
```
## Procedure exec invoice list
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_invoice_master_list_web]    Script Date: 08/06/2024 10:44:24 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

ALTER procedure [dbo].[sp_invoice_master_list_web] 
       (
         @round_id varchar(50),
         --@search  nvarchar(50),
         --@f_dt date,
         --@t_dt date,
         @cust_dealer varchar(30),
         @offset int,
         @limit int
       )
as 
begin

---dealer amt---
  select cust_cd,cntr_no,persen=( case when  a.rent_ty='LTT100100' then 27
                                        when  a.rent_ty='LTT100200' then 28
								        when  a.rent_ty='LTT100300' then 20
								        when  a.rent_ty='LTT100400' then 30
								        else  0
								 end )
  into #dealer_amt_presen
   from LTS100 a 
   where  cust_cd=@cust_dealer
     and cntr_no=(select max(cntr_no) from LTS100 where cust_cd=@cust_dealer)

  ---***---
  select a.co_cd,b.inv_no, inv_dt,a.cust_dealer,a.roundId,round_dt=c.dt,
     Digit1=isnull(a.Digit1,0),
	    Digit2=isnull(a.Digit2,0),
	    Digit3=isnull(a.Digit3,0),
	    Digit4=isnull(a.Digit4,0),
	    Digit5=isnull(a.Digit5,0),
	    Digit6=isnull(a.Digit6,0),
     a.inv_amt,
     issue_qty =isnull(b.issue_qty, dbo.fc_pos_qty (a.cust_dealer, a.roundid )),
     active_qty =isnull(active_qty, dbo.fc_Active_qty (a.cust_dealer , a.roundid)),
     dealer_amt=isnull(a.inv_amt,0)*isnull(e.persen,0)/100, --dealer_amt2=[dbo].[fc_dealer_amt] (@_cust_dealer, a.roundid ),
     ontime_amt=case when isnull(b.pay_dt,getdate())<=c.ont_dt then a.inv_amt*2/100 else 0 end, --ontime_amt2=dbo.fc_ontime_amt(@_cust_dealer, a.roundid),
     --target_avg_amt=isnull(b.target_avg,g.target_avg_amt),
     --target_avg_amt=case when a.inv_amt/dbo.fc_pos_qty (@cust_dealer, a.roundid) > = 300000 and dbo.fc_Active_qty (@cust_dealer, a.roundid)>=isnull(b.issue_qty,dbo.fc_pos_qty (@cust_dealer, a.roundid))*70/100 then a.inv_amt * 1/100 
     --else 0 end, --target_avg_amt2=dbo.fc_target_avg_amt(@_cust_dealer,a.roundid),
     pay_win_amt=isnull(f.pay_win_amt,0), pay_win_amt1=isnull(f.pay_win_amt,0),  --pay_win_amt2=dbo.fc_win_amt(@_cust_dealer,a.roundid),
     percen_pay_win_amt=isnull(f.pay_win_amt,0)/100,
     --data_pkg=isnull(b.data_pkg,g.data_pkg_amt),
     --data_pkg =isnull(b.data_pkg, dbo.fc_data_pkg (@cust_dealer ,SUBSTRING(CONVERT(nvarchar(6), DATEADD(month, -1, (select top 1 pay_dt from liv100 where cust_dealer=@cust_dealer and roundid=a.roundid)), 112),1,6) )),
     coll_amt=isnull(d.coll_amt,0),
     receive_amt=b.receive_amt,
     transfer_fee=b.transfer_fee
   into #tem
   from vs_LTU100 a       
   left join LIV100 b on a.cust_dealer=b.cust_dealer and a.roundId=b.roundid
   join ltm300 c on a.roundid=c.roundId
   left join (select coll_amt=isnull(sum(tot_amt),0), cust_dealer, roundid FROM [KKL].[dbo].[LCO400] where cust_dealer=@cust_dealer group by roundid, cust_dealer) d on a.cust_dealer=d.cust_dealer and a.roundid=d.roundid
   left join #dealer_amt_presen e on a.cust_dealer=e.cust_cd
   left join (Select pay_win_amt=isnull(sum(amt),0), pay_by, pay_on_round
								      from KKL.dbo.LTU110 
              group by pay_by, pay_on_round 
              ) f on a.cust_dealer=f.pay_by and a.roundid=f.pay_on_round
   --left join @target_avg_data_avg g on a.roundid=g.round_id and a.cust_dealer=g.cust_dealer
  where 1=1
   --and a.upload_dt between @f_dt and @t_dt
   --and b.inv_no like '%'+@search+'%' 
   and a.roundId like '%'+@round_id+'%' 
   and a.cust_dealer =@cust_dealer 
  --order by c.dt desc
  --OFFSET @offset ROWS
  --FETCH NEXT @limit ROWS ONLY

---*target_avg_amt || data_pkg_amt*---
  select a.co_cd,inv_no, inv_dt,a.cust_dealer,a.roundId,round_dt,
     Digit1,Digit2,Digit3,Digit4,Digit5,Digit6,
     a.inv_amt,coll_amt,dealer_amt,ontime_amt,pay_win_amt,percen_pay_win_amt,receive_amt, transfer_fee,
     target_avg_amt = ( case when a.inv_amt/a.issue_qty > = 300000 and a.active_qty>=a.issue_qty*70/100 then a.inv_amt * 1/100 else 0 end ),
     data_pkg=b.data_pkg_amt
  into #tem2
  from #tem a 
  left join (select a.cust_dealer, a.roundId,data_pkg_amt=case when sum (amt)/count(*)>3600000 then 20000 else 0 end
                --count(*), sum (amt)/count(*)
             from (select up_month=SUBSTRING(CONVERT(nvarchar(6), b.dt, 112),1,6), 
                     amt=sum(inv_amt),
                     --data_pkg=case when sum (amt)>3600000 then 20000 else 0 end,
                     cust_dealer, a.roundId
                   --into #tem
                   from #tem a
                   left join ltm300 b on a.roundId=b.roundId
                   where cust_dealer=@cust_dealer
                    and SUBSTRING(CONVERT(nvarchar(6), round_dt, 112),1,6)= SUBSTRING(CONVERT(nvarchar(6), DATEADD(month, -1, a.round_dt), 112),1,6)
                   group by SUBSTRING(CONVERT(nvarchar(6), b.dt, 112),1,6), cust_dealer, a.roundId
                  )a group by cust_dealer, roundId) b on a.cust_dealer=b.cust_dealer and a.roundid=b.roundId
  where 1=1

---*receive_amt || transfer_fee*---
  select a.co_cd,inv_no, inv_dt,a.cust_dealer,a.roundId,round_dt,
     Digit1,Digit2,Digit3,Digit4,Digit5,Digit6,
     a.inv_amt,coll_amt,
     dealer_amt, ontime_amt, target_avg_amt, pay_win_amt, percen_pay_win_amt, data_pkg,
     receive_amt=isnull(a.receive_amt,isnull(a.inv_amt,0)
                                             - isnull(a.dealer_amt,0)
                                             - isnull(ontime_amt,0)
                                             - isnull(target_avg_amt,0)
                                             - isnull(pay_win_amt,0) 
                                             - isnull(percen_pay_win_amt,0)
                                             - isnull(data_pkg,0)),
    transfer_fee=isnull(transfer_fee, dbo.fc_dl_fransfer_fee_amt (isnull(a.inv_amt,0)
                                                                   - isnull(a.dealer_amt,0)
                                                                   - isnull(ontime_amt,0)
                                                                   - isnull(target_avg_amt,0)
                                                                   - isnull(pay_win_amt,0) 
                                                                   - isnull(percen_pay_win_amt,0)
                                                                   - isnull(data_pkg,0)))
  into #tem3
  from #tem2 a

---*balance_amt*---
  select a.co_cd,inv_no, inv_dt,a.cust_dealer,a.roundId,round_dt,
    Digit1,Digit2,Digit3,Digit4,Digit5,Digit6,
    a.inv_amt,coll_amt,
    dealer_amt, ontime_amt, target_avg_amt, pay_win_amt, percen_pay_win_amt, data_pkg,
    receive_amt,transfer_fee,
    balance_amt=receive_amt-coll_amt-transfer_fee
  from #tem3 a
  order by round_dt desc
  OFFSET @offset ROWS
  FETCH NEXT @limit ROWS ONLY
end
  
```
### exec
```
USE [KKL]
GO

DECLARE	@return_value int

EXEC	@return_value = [dbo].[sp_invoice_master_list_web]
		@round_id = N'24063',
		@cust_dealer = N'ibbank13',
		@offset = 1,
		@limit = 10

SELECT	'Return Value' = @return_value

GO

```
## Procedure save from web
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_invoice_save_web]    Script Date: 08/06/2024 10:44:40 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
---New use---
ALTER procedure [dbo].[sp_invoice_save_web] 
       (
         @inv_no varchar(30),
         @seq int,
         @cust_dealer varchar(30),
         @roundid varchar(20),
         @inv_amt decimal(22,2),
         @inv_dt date,
         @dealer_amt decimal(22,2),
         @ontime_amt decimal(22,2),
         @target_avg decimal(22,2),
         @luky_amt decimal(22,2),
         @dl_pay_amt decimal(22,2),
         @unit_avg decimal(22,2),
         @receive_amt decimal(22,2),
         @pr_rd_amt decimal(22,2),
         @data_pkg decimal(22,2),
         @issue_qty int,
         @disable_qty int,
         @active_qty int,
         @rmk nvarchar(1000),
         @device_id nvarchar(50),
         @transfer_fee decimal(22,2)
         --@reg_id int
       )
as 
begin

  IF EXISTS (SELECT inv_no FROM LIV100 WHERE inv_no = @inv_no and roundid=@roundid)
    BEGIN 
      update LIV100 set 
         inv_amt=@inv_amt
        ,inv_dt=@inv_dt
        ,dealer_amt=@dealer_amt
        ,ontime_amt=@ontime_amt
        ,target_avg=@target_avg
        ,luky_amt=@luky_amt
        ,dl_pay_amt=@dl_pay_amt
        ,unit_avg=@unit_avg
        ,receive_amt=@receive_amt           
        ,pr_rd_amt=@pr_rd_amt
        ,data_pkg=@data_pkg
        ,issue_qty=@issue_qty
        ,disable_qty=@disable_qty
        ,active_qty=@active_qty
        ,rmk=@rmk
        --,mid =@reg_id
        --,mdt = getdate()          
      where inv_no =@inv_no and roundid=@roundid
       
      end
      else begin 
      INSERT INTO LIV100 (inv_no,seq,cust_dealer,roundid,inv_amt,inv_dt,dealer_amt,ontime_amt,
          target_avg,luky_amt,dl_pay_amt,unit_avg,receive_amt,pr_rd_amt,data_pkg,
          issue_qty,disable_qty,active_qty,rmk,cur_bc,cdt,insert_fr, pay_dt, device_id, transfer_fee)
      VALUES(@inv_no,@seq,@cust_dealer ,@roundid,@inv_amt ,getdate(),@dealer_amt,@ontime_amt,
          @target_avg ,@luky_amt,@dl_pay_amt,@unit_avg,@receive_amt,@pr_rd_amt,@data_pkg,
          @issue_qty,@disable_qty,@active_qty,@rmk,'BC400LAK',GETDATE(),'INSERT FROM WEB', getdate(), @device_id, @transfer_fee)
    
    end

    ---update luky amt---
    update LIV100 set luky_amt=[dbo].[fc_bill_lucky_test] (cust_dealer,@roundid)
    where inv_no =@inv_no and roundid=@roundid and cust_dealer=@cust_dealer

end
  
```
## Procedure invoice search
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_invoice_details]    Script Date: 08/06/2024 10:49:42 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

---Update final---
ALTER procedure [dbo].[sp_invoice_details] 
       (
         @round_id varchar(50),
         @cust_dealer varchar(30)
       )
as 
begin
  ---master invoice new round--- 
  select 
    b.inv_no,
	a.cust_dealer, 
	a.roundid, 
	round_dt=c.dt,
    inv_amt=isnull(a.inv_amt, 0), 
	inv_dt= isnull(b.inv_dt,getdate()),
	pay_dt= isnull(b.pay_dt,getdate()),	
	dealer_amt=isnull(b.dealer_amt, 
	a.dealer_amt),	
    ontime_amt=isnull(b.ontime_amt,case when isnull(b.pay_dt,getdate())<=c.ont_dt then a.inv_amt*2/100 else 0 end),	
    target_avg=isnull(b.target_avg,(case when a.inv_amt/a.issue_qty > = 300000 and a.active_qty>=a.issue_qty*70/100 then a.inv_amt * 1/100 else 0 end )),	
    luky_amt=isnull(a.luky_amt,0),	pr_rd_amt=isnull(b.pr_rd_amt, isnull(f.pay_win_amt,0)/100),	
	unit_avg=isnull(b.unit_avg,a.inv_amt/a.issue_qty),	
    issue_qty=isnull(b.issue_qty,a.issue_qty),	
	active_qty=isnull(b.active_qty,a.active_qty),	
	disable_qty=isnull(b.disable_qty,a.disable_qty),	
    data_pkg=isnull(b.data_pkg,a.data_pkg),	
    f_coll_amt=isnull(d.coll_amt,0),	
    pay_win_amt=isnull(pay_win_amt,0),	
	rmk
  into #master_invoice_new_round
  from liv101 a 
  left join LIV100 b on a.cust_dealer=b.cust_dealer and a.roundId=b.roundid
  join ltm300 c on a.roundid=c.roundid
  left join (select coll_amt=isnull(sum(tot_amt),0), cust_dealer, roundid FROM [KKL].[dbo].[LCO400] where cust_dealer=@cust_dealer group by roundid, cust_dealer) d on a.cust_dealer=d.cust_dealer and a.roundid=d.roundid
  left join (Select pay_win_amt=isnull(sum(amt),0), pay_by, pay_on_round
								      from KKL.dbo.LTU110 
              group by pay_by, pay_on_round 
              ) f on a.cust_dealer=f.pay_by and a.roundid=f.pay_on_round
  where 1=1 
  and a.roundid=@round_id
  and a.cust_dealer=@cust_dealer


  ---master invoice new round2--- 
  select a.inv_no,
    a.cust_dealer, 
	a.roundid, 
	round_dt,
    inv_amt, 
	a.inv_dt,
	a.pay_dt,	
	dealer_amt,	
	ontime_amt,	
	target_avg,	
	pr_rd_amt,	
	unit_avg,	
	issue_qty,	
    active_qty,	
	disable_qty,	
	data_pkg,	
    receive_amt=isnull(a.inv_amt,0)- isnull(a.dealer_amt,0)- isnull(ontime_amt,0)- isnull(target_avg,0)- isnull(pay_win_amt,0) - isnull(pr_rd_amt,0)- isnull(data_pkg,0),
    transfer_fee=dbo.fc_dl_fransfer_fee_amt (isnull(a.inv_amt,0)- isnull(a.dealer_amt,0)- isnull(ontime_amt,0)- isnull(target_avg,0)- isnull(pay_win_amt,0) - isnull(pr_rd_amt,0)- isnull(data_pkg,0)),
    f_coll_amt, 
	win_amt=pay_win_amt, 
	a.luky_amt,
	rmk
  into #master_invoice_new_round2
  from #master_invoice_new_round a
  where 1=1 


  ---final---
  ---actual invoice old round--- 
  select 
    a.inv_no,
	a.cust_dealer, 
	a.roundid, 
	round_dt=c.dt,Digit1=0,Digit2=0,Digit3=0,Digit4=0,Digit5=0,Digit6=0,
	inv_amt=isnull(a.inv_amt, 0), 
	a.inv_dt,	
	a.pay_dt,
	dealer_amt=isnull(a.dealer_amt, 0),	
    ontime_amt=isnull(a.ontime_amt,0),	
    target_avg=isnull(a.target_avg,0),	
	pr_rd_amt=isnull(a.pr_rd_amt,0),	
	unit_avg=isnull(unit_avg,inv_amt/issue_qty),	
    issue_qty=isnull(issue_qty,0),	
	active_qty=isnull(active_qty,0),	
	disable_qty=isnull(disable_qty,0),	
    data_pkg=isnull(data_pkg,0),	
	receive_amt=isnull(receive_amt,0),
	transfer_fee=isnull(a.transfer_fee,0),
    qr_amt=isnull(receive_amt,0)-isnull(transfer_fee,0), 
	f_coll_amt=isnull(d.coll_amt,0),	
    f_balance =isnull(a.receive_amt,0)-isnull(coll_amt,0),	-- use old
    balance=isnull(a.receive_amt,0)-isnull(coll_amt,0)- isnull(transfer_fee,0),--use new
    win_amt=isnull(pay_win_amt,0), a.luky_amt,	rmk,
	ont_dt =(select ont_dt from LTM300 where roundId =a.roundId )
  from liv100 a 
  join ltm300 c on a.roundid=c.roundid
  left join (select coll_amt=isnull(sum(tot_amt),0), cust_dealer, roundid FROM [KKL].[dbo].[LCO400] where cust_dealer=@cust_dealer group by roundid, cust_dealer) d on a.cust_dealer=d.cust_dealer and a.roundid=d.roundid
  left join (Select pay_win_amt=isnull(sum(amt),0), pay_by, pay_on_round
								      from KKL.dbo.LTU110 
              group by pay_by, pay_on_round 
              ) f on a.cust_dealer=f.pay_by and a.roundid=f.pay_on_round
  where 1=1 
  and a.roundid=@round_id
  and a.cust_dealer=@cust_dealer
  and c.dt<'2024-03-01'

  union all

  ---master invoice new round3--- 
  select 
    a.inv_no,
	a.cust_dealer, 
	a.roundid, 
	round_dt,
	Digit1=0,Digit2=0,Digit3=0,Digit4=0,Digit5=0,Digit6=0,
    inv_amt, 
	a.inv_dt,
	a.pay_dt,	
	dealer_amt,	
	ontime_amt,	
	target_avg,	
	pr_rd_amt,	
	unit_avg,	
	issue_qty,	
    active_qty,	
	disable_qty,	
	data_pkg,	
    receive_amt,
    transfer_fee=dbo.fc_dl_fransfer_fee_amt (receive_amt),
    qr_amt=isnull(receive_amt,0)-isnull(transfer_fee,0), 
    f_coll_amt,	
    f_balance =isnull(a.receive_amt,0)-isnull(f_coll_amt,0),	-- use old
    balance=isnull(a.receive_amt,0)-isnull(f_coll_amt,0)- isnull(transfer_fee,0),--use new
    win_amt, a.luky_amt,	rmk,
	ont_dt =(select ont_dt from LTM300 where roundId =a.roundId )
  from #master_invoice_new_round2 a
  where 1=1

    union all

  ---bank partner 
  select 
    inv_no=b.inv_no,cust_dealer=a.partner_id, a.roundid, round_dt='',
    Digit1=0,Digit2=0,Digit3=0,Digit4=0,Digit5=0,Digit6=0,
    inv_amt=a.tot_amt, 
	inv_dt=getdate(),
	pay_dt=getdate(),	
	dealer_amt = tot_amt*com_amt/100 ,
    ontime_amt=0,	
	target_avg=0,	
	pr_rd_amt=( case when a.partner_id in('BcelOne','IBCool','LottoPoints') then 0 else  isnull(a.lucky_amt,0)*1/100 end ) ,	
	unit_avg=0,	
	issue_qty=0,	
    active_qty=0,	
	disable_qty=0,	
	data_pkg=0,	
    receive_amt = ( case when a.partner_id in('BcelOne','IBCool','LottoPoints') then  tot_amt-(tot_amt*com_amt/100) else tot_amt-(tot_amt*com_amt/100)- (isnull(a.lucky_amt,0))-(isnull(a.lucky_amt,0)*1/100) end ) ,
    transfer_fee=0,
    qr_amt=0, 
    f_coll_amt=0,	
    f_balance =0,	
    balance=( case when a.partner_id in('BcelOne','IBCool','LottoPoints') then  tot_amt-(tot_amt*com_amt/100) else tot_amt-(tot_amt*com_amt/100)- (isnull(a.lucky_amt,0))-(isnull(a.lucky_amt,0)*1/100) end ) ,--use new
    win_amt=0, 
	lucky_amt =( case when a.partner_id in('BcelOne','IBCool','LottoPoints') then 0 else isnull(a.lucky_amt,0) end ) ,
	rmk=b.rmk,
	ont_dt =(select ont_dt from LTM300 where roundId =a.roundId )
  from LCO300 a
  left join LIV100 b on a.partner_id = b.cust_dealer and a.roundid=b.roundid
  where 1=1
	and a.roundid =@round_id 
	and a.partner_id =@cust_dealer 
	

end
  

```
## Procedure histrory list
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_payment_history_list]    Script Date: 08/06/2024 10:51:20 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
--exec dbo.sp_appr_plan @co_cd,@req_no,@emp_no,@coments
ALTER procedure [dbo].[sp_payment_history_list] 
       (
         @round_id  nvarchar(100),
         --@stat_cd varchar(15),
         @search nvarchar(100),
         @cust_dealer nvarchar(20), 
         @offset int,
         @limit int
       )
as 
begin

 declare @_rid varchar(10) = @round_id

 select a.bill_no, total=sum(amt), 
   win_lot=sum(case when pay_ty='LTW200100' then amt else 0 end),
   win_ticket=sum(case when pay_ty='LTW200200' then amt else 0 end),
   barcode=a.bill_no,a.cust_dealer,roundId=pay_on_round,
   pay_id='',dess='',stat_cd='',bill_pay='',amt_digit1=0,amt_digit2=0,
   amt_digit3=0,amt_digit4=0,amt_digit5=0,amt_digit6=0, pay_dt=NULL, pay_no='', vat=0, 
   pay_amt=sum(amt)
 from ltu110 a
  where a.pay_by=@cust_dealer
  and a.pay_on_round=@_rid
 group by a.bill_no, cust_dealer, pay_on_round
 order by a.pay_on_round,a.bill_no
 OFFSET @offset ROWS
 FETCH NEXT @limit ROWS ONLY

 --declare @_rs varchar(6) = (select result from LTM300 where roundId = @_rid)

 -----Master bill---
 --select bill_no, barcode, cust_dealer, roundId, pay_dt,
 --      pay_id,dess,stat_cd,bill_pay
 -- into #tbl_bill_master
 -- from LTU100 a
 --where 1=1 
 --  --and roundId=@_rid
 --  and a.stat_cd='LTS300200'
 --  --and a.cust_dealer=@cust_dealer
 --  and (a.bill_no like '%'+@search+'%' or a.barcode like '%'+@search+'%')
 --  and (digit =RIGHT(@_rs, 1) or
 --       digit =RIGHT(@_rs, 2) or
	--       digit =RIGHT(@_rs, 3) or
	--       digit =RIGHT(@_rs, 4) or
	--       digit =RIGHT(@_rs, 5) or
	--       digit =RIGHT(@_rs, 6) ) 
 --group by bill_no, barcode, cust_dealer, roundId, pay_dt,pay_id,dess,stat_cd,bill_pay


 -----Group by digit---
 --select bill_no, barcode, cust_dealer, roundId,
 --    amt_digit1 = sum(case when len(digit) = 1 and digit =RIGHT(@_rs, 1) then amt*5 else 0 end),
 --    amt_digit2 = sum(case when len(digit) = 2 and digit =RIGHT(@_rs, 2) then amt*60 else 0 end),
 --    amt_digit3 = sum(case when len(digit) = 3 and digit =RIGHT(@_rs, 3) then amt*500 else 0 end),
 --    amt_digit4 = sum(case when len(digit) = 4 and digit =RIGHT(@_rs, 4) then amt*5000 else 0 end),
 --    amt_digit5 = sum(case when len(digit) = 5 and digit =RIGHT(@_rs, 5) then amt*40000 else 0 end),
 --    amt_digit6 = sum(case when len(digit) = 6 and digit =RIGHT(@_rs, 6) then amt*400000 else 0 end),
	--    total = --sum(case when len(digit) = 1 and digit =RIGHT(@_rs, 1) then amt*5 else 0 end)
 --         sum(case when len(digit) = 2 and digit =RIGHT(@_rs, 2) then amt*60 else 0 end)
	--         + sum(case when len(digit) = 3 and digit =RIGHT(@_rs, 3) then amt*500 else 0 end)
	--         + sum(case when len(digit) = 4 and digit =RIGHT(@_rs, 4) then amt*5000 else 0 end)
	--         + sum(case when len(digit) = 5 and digit =RIGHT(@_rs, 5) then amt*40000 else 0 end)
	--         + sum(case when len(digit) = 6 and digit =RIGHT(@_rs, 6)  then amt*400000 else 0 end)

 -- into #tbl_summary_by_digit
 -- from LTU100 a
 --where 1=1
 --  --and roundId=@_rid
 --  and a.stat_cd='LTS300200'
 --  --and a.cust_dealer=@cust_dealer
 --  and (a.bill_no like '%'+@search+'%' or a.barcode like '%'+@search+'%')
 --  and (digit =RIGHT(@_rs, 1) or
 --       digit =RIGHT(@_rs, 2) or
	--       digit =RIGHT(@_rs, 3) or
	--       digit =RIGHT(@_rs, 4) or
	--       digit =RIGHT(@_rs, 5) or
	--       digit =RIGHT(@_rs, 6) ) 
 --group by bill_no, barcode, cust_dealer, roundId

 -----summary---
 --select a.bill_no,a.barcode,a.cust_dealer,a.roundId,
 --  a.pay_id,a.dess,a.stat_cd,a.bill_pay,amt_digit1,amt_digit2,
 --  amt_digit3,amt_digit4,amt_digit5,amt_digit6,total, d.pay_dt, d.pay_no, d.vat, 
 --  pay_amt=d.amt, win_lot=1000, win_ticket=1000
 --from #tbl_bill_master a
 --join #tbl_summary_by_digit b on a.bill_no=b.bill_no and a.cust_dealer=b.cust_dealer
 --join ltu110 d on a.bill_no=d.bill_no and a.cust_dealer=d.cust_dealer
 --where d.pay_by=@cust_dealer
 -- and d.pay_on_round=@_rid
 --order by a.roundId,a.cust_dealer,a.bill_no
 --OFFSET @offset ROWS
 --FETCH NEXT @limit ROWS ONLY

end
  
```
## Procedure save photo 
```
USE [KKL]
GO
/****** Object:  StoredProcedure [dbo].[sp_save_bill_photo_upload]    Script Date: 08/06/2024 10:52:20 AM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

ALTER PROCEDURE [dbo].[sp_save_bill_photo_upload]
(
	@bill_no nvarchar(50), 
	@fid nvarchar(200), 
	@fext nvarchar(10), 
	@fname nvarchar(200)
)
AS
begin
 
if exists(select 1 from ltu120 where bill_no=@bill_no)
begin 
   update a
     set fid=@fid,
	        fext=@fext,
	        fname=@fname,
	        mdt=getdate()
   from ltu120 a 
   where bill_no=@bill_no
end 
else
begin 
  insert into ltu120 (bill_no, fid, fext, fname, cdt)
  values (@bill_no, @fid, @fext, @fname, getdate())
end

end


```
