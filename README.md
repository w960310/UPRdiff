# UPRdiff

想看保費入賬分布:綜合繳別，起效日來判斷
select distinct modx,count(modx)
from polf
where insurance_type = 'H'
and left(po_issue_date,3) in ('111','112')
group by 1

select distinct substr(co_issue_date,5,2),count(substr(co_issue_date,5,2))
from arco_a
where insurance_type = 'H'
and co_sts_code in ('42')
group by 1

select distinct substr(co_issue_date,5,2),count(substr(co_issue_date,5,2))
from upr_com
group by 1



select a.plan_code, a.rate_scale, a.plan_term_ind ,b.policy_no, b.co_sts_code, 

-------------------------------------------

drop table if exists a1;
drop table if exists a2;

select value_date,plan_abbr_code
from upr_all
where left(value_date,3) in ('109','110','111')
and plan_abbr_code in ('JAD')
order by 1,2
into temp a1;

select a.*, 
amt_1,
amt_2,
amt_3
from a1 a, acti b
where left(a.value_date,3) = left(b.value_date,3)
and a.plan_abbr_code = b.plan_abbr_code
order by 1,2
into temp a2;

unload to 'JAD_amt.txt'
select * from a2


drop table if exists a1;
select value_date, plan_abbr_code,sum(cnt) as cnt
from upr_all
where left(value_date,3) in ('109','110','111')
and plan_abbr_code in ('JAD')
group by value_date
order by 1
into temp a1;
unload to 'JAD_cnt.txt'
select * from a1
--------------------------------------
drop table if exists a1;
drop table if exists a2;

select value_date, plan_abbr_code,
case when dur = '1'
     then 'FY'
     else 'RY' end as FY_RY,
sum(cnt) as cnt,
sum(agp_upr) as agp,
sum(upr) as upr
from upr_all
where left(value_date,3) in ('111','112')
and right(value_date,5) in ('12/31','11/30','10/31','09/30','08/31','07/31',
'06/30','05/31','04/30','03/31','02/28','01/31')
and plan_abbr_code in ('JAD','KAD')
group by 1,2,3
order by 1,2,3
into temp a1;

select a.*, 
case when FY_RY = 'FY'
     then sum(amt_1+amt_2)
     else sum(amt_3) end as prem
from a1 a, acti b
where left(a.value_date,3) = left(b.value_date,3)
and a.plan_abbr_code = b.plan_abbr_code
group by 1,2,3,4,5,6
order by 1,2,3,4,5,6
into temp a2;

unload to 'KAD_JAD_111~112.txt'
select * from a2
-------------------------------------------------

團體傷害險即便保收尚未入帳，仍屬於有效單，故與個人險有些不同，
在即將滿期時，UPR提存比率很低，但此時如果才剛入帳，就會導致未滿期保費佔率降低。

而個人險有入帳才會提存UPR，所以單純看入帳時間分布就可以，不會有上述原因，
且因為大部分保單繳別都屬於年繳，因此可以用起效日粗略估計其入賬時間，
雖然會有部分慢一個月才入帳的單，導致實際提存UPR時間點在後面一個月。
但大致上可看出保費收入的浮動與起效日有一定程度關係。

團險學保部分，會因為學期關係，越接近學期末UPR提存率越低，將會影響未滿期保費佔率。
