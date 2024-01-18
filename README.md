# Credit-card-transactions
CREATE TABLE credit_card_transcations
(
    index integer,
    City character(50),
    Date date,
    CardType character(30),
    ExpType character(20),
    Gender character(1),
    Amount integer
);

select * from credit_card_transcations;


1- write a query to print top 5 cities with highest spends and their percentage contribution of total credit card spends 
with cte1 as
(SELECT City, 
    sum(Amount) as total_spent_citywise
	FROM credit_card_transcations
	group by City
	order by total_spent_citywise desc
	limit 5),
cte2 AS (select 
	sum(Amount) as total_amount
    from credit_card_transcations)
	select c1.City,c1.total_spent_citywise, 
	round(cast(c1.total_spent_citywise as decimal) / c2.total_amount * 100 ,2)
	as percentage_contribution
	from cte1 c1
	inner join cte2 c2 on 1=1;
	
2- write a query to print highest spend month and amount spent in that month for each card type	
with cte1 as
(select 
 extract(month from Date)as m,
 extract(year from Date) as y,
 CardType,
 sum(Amount)as total_amount
 from credit_card_transcations
 group by extract(month from Date),
 extract(year from Date),
 CardType),
cte2 as (
	select *, 
	dense_rank() over(partition by
    CardType order by total_amount desc) as rnk 
	from cte1)
select * from cte2 where rnk =1 ;
									 
3- write a query to print the transaction details(all columns from the table) for each card type when	
with cte1 as
(select *,
 sum(Amount) 
 over(partition by CardType
 order by Date)as cumulative_sum 
 from credit_card_transcations),
cte2 as (
	select *, dense_rank() 
	over(partition by CardType 
    order by cumulative_sum) as rnk 
	from cte1 
	where cumulative_sum >=1000000)
select * from cte2 where rnk =1 ;

4- write a query to find city which had lowest percentage spend for gold card type	
with cte1 as
(select City,
  sum(Amount) as total_gold_amount
  from credit_card_transcations
  where CardType ='Gold' 
  group by City),
cte2 as (
	select City,
	sum(Amount) as trans_amount
    from credit_card_transcations 
	group by City),
cte3 as (
	select c1.City,
	c1.total_gold_amount,
	c2.trans_amount,
    round(cast(c1.total_gold_amount as decimal) / c2.trans_amount * 100,2) as per_contribution
    from cte1 c1 
	inner join cte2 c2 on c1.City = c2.City)
select * from cte3
order by per_contribution limit 1;

5- write a query to print 3 columns:  city, highest_expense_type , lowest_expense_type	
with cte1 as
(select city, 
 exptype,
 sum(amount) as total_amount 
 from credit_card_transcations
 group by city,exptype),
 cte2 as (
 select city, 
	 max(total_amount)  as highest_expense,
     min(total_amount) as lowest_expense 
	 from cte1 group by city)
select c1.city,
   max(case when total_amount = highest_expense then exptype end) as highest_expense_type,
   min(case when total_amount = lowest_expense then exptype end) as lowest_expense_type
   from cte1 c1
   inner join cte2 c2 on c1.city= c2.city
   group by c1.city
   order by c1.city;

6- write a query to find percentage contribution of spends by females for each expense type	
with cte1 as(
select exptype, sum(amount) as total 
from credit_card_transcations 
where gender='F'
group by exptype),
cte2 as (
select exptype, sum(amount) as total_amount
from credit_card_transcations
group by exptype)
select c1.exptype,c1.total,
c2.total_amount,round(cast(c1.total as decimal)/c2.total_amount * 100,2) as
percentage_spends from cte1 as c1
inner join cte2 as c2 on
c1.exptype = c2.exptype;

7- which card and expense type combination saw highest month over month growth in Jan-2014
with month_year_spend as (
	select 
	cardtype, 
	exptype, 
	Extract(month from date) as spend_month, 
	Extract(year from date) as spend_year,
	sum(amount) as spend
	from credit_card_transcations
	Group By cardtype, exptype, spend_month, spend_year
)	
,get_prev_spend as (
	select 
	*
	,lag(spend,1)over(partition by cardtype, exptype order by spend_year, spend_month) as lag_spend
	from month_year_spend
)	
select *, 
(spend-lag_spend) as growth
from get_prev_spend
where spend_month = 1 and spend_year = 2014 and(spend-lag_spend) > 0 
order by (spend-lag_spend) desc limit 1;


8- during weekends which city has highest total spend to total no of transcations ratio 
select city,
sum(amount) as total,
count(1) as total_no_of_trans,
sum(amount) / count(1) as ratio
from credit_card_transcations
where extract(DOW from date) in('6','0')
group by city
order by ratio desc
limit 1;


9- which city took least number of days to reach its 500th transaction after the first transaction in that city
with cte1  AS (
	select c1.city, c1.date from  
( select city, date, dense_rank()over(partition by city order by date asc) rank_txns
from credit_card_transcations) c1 where c1.rank_txns = 1
)
, cte2 AS (
	select c1.city, c1.date, c1.rns from
( select city, date, ROW_NUMBER()over(partition by city order by date asc) rns 
from credit_card_transcations) c1
where c1.rns = 500
)
select f.city as city, f.date as first_txn_date, l.date as txn_date_500th, l.date-f.date as days
from cte1 as f join cte2 as l on f.city = l.city
order by l.date-f.date limit 1;





