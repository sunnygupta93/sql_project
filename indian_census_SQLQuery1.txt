select * from projectsqlind_cen.dbo.Data1;
select * from projectsqlind_cen.dbo.Data2;

use projectsqlind_cen

select count(*) from dbo.Data2;
select * from projectsqlind_cen.dbo.Data1 where State in ('Jharkhand','Bihar');
select * from projectsqlind_cen.dbo.Data1 where State ='Bihar';

select sum(Population) from projectsqlind_cen.dbo.Data2;
select sum(Population) as pop from Data2;

---avg_growth
select state,AVG(Growth)*100 as avg_growth_percentage from projectsqlind_cen.dbo.Data1 group by State;

---avg_sex_ratio
select state, round(AVG(Sex_ratio),0) as Avg_sex_ratio from projectsqlind_cen.dbo.Data1 group by State order by Avg_sex_ratio  desc;

---avg literacy rate
select state, round(AVG(Literacy),0) as Avg_literacy from projectsqlind_cen.dbo.Data1 group by State having round(AVG(Literacy),0)>90 order by Avg_literacy  desc ;

---top 3 state showing highest growth ratio
select  top 3 state,AVG(Growth)*100 as avg_growth_percentage from projectsqlind_cen.dbo.Data1 group by State order by avg_growth_percentage desc ;

---bottom 3 state showing lowest sex ratio
select top 3 state, round(AVG(Sex_ratio),0) as Avg_sex_ratio from projectsqlind_cen.dbo.Data1 group by State order by Avg_sex_ratio  asc ;

---top and bottom 3 states in literacy state
create table #topstates
( state nvarchar(255),
    topstates float)
	insert into #topstates
	select top 3 state, round(AVG(Literacy),0) as Avg_literacy from projectsqlind_cen.dbo.Data1 group by State  order by Avg_literacy  desc

	create table #bottomstates
( state nvarchar(255),
    topstates float)
	insert into #bottomstates
	select top 3 state, round(AVG(Literacy),0) as Avg_literacy from projectsqlind_cen.dbo.Data1 group by State  order by Avg_literacy  asc

select * from #bottomstates
union all
select * from #topstates

--state starting with letter 'a'
select distinct state from projectsqlind_cen.dbo.Data1 where state like 'a%' or State like 'b%'

--state starting with letter 'a' or end with 'd'
select distinct state from projectsqlind_cen.dbo.Data1 where state like 'a%' or State like '%d'

----joing both table

select d.state, sum(d.males) as total_males, sum(d.females) as total_females from
(select c.district, c.state, round(c.population/(1+c.sex_rat),0) as males, round(c.population*c.sex_rat/(1+c.sex_rat),0) as females from
(select a.district,a.state,a.sex_ratio/1000 as sex_rat,b.population from projectsqlind_cen.dbo.Data1 a inner join projectsqlind_cen.dbo.Data2 b on
  a.district= b.district) c)
 d group by d.state

 --- total literacy rate

 select d.state, sum(d.total_literate_people) as total_literate_pop,sum(d.total_illiterate_people) as total_illiterate_pop from
 (select c.district, c.state, c.literacy_ratio,round(c.population*c.literacy_ratio,0) as total_literate_people,round((1-c.literacy_ratio)*population,0) as total_illiterate_people from
 (select a.district,a.state,a.Literacy/100 as literacy_ratio,b.population from projectsqlind_cen.dbo.Data1 a inner join projectsqlind_cen.dbo.Data2 b on
  a.district= b.district) c) d group by d.state 

  --population in previous census

  select '1' as keyy, n.* from(
  select sum(d.prev_pop) as prev_sum,sum(d.curr_pop) as curr_sum from(
  select c.state, sum(c.prev_census_pop) as prev_pop, sum(c.curr_census_pop)as curr_pop from
 ( select b.district, b.state,  round(b.population/(1+b.growth),0) as prev_census_pop, b.Population as curr_census_pop from
(select a.district,a.state,a.growth,b.population from projectsqlind_cen.dbo.Data1 a inner join projectsqlind_cen.dbo.Data2 b on
  a.district= b.district) b) c group by c.state
  ) d
  )n


----population vs area

select '1' as keyy, z.* from (
select SUM(Area_km2) as total_area from projectsqlind_cen.dbo.Data2
)z


-----vs

select g.total_area/g.prev_sum as area_pop_ratio_prev,g.total_area/g.curr_sum as area_pop_ratio_curr from(
select e.*,f.* from(
  select '1' as keyy, n.* from(
  select sum(d.prev_pop) as prev_sum,sum(d.curr_pop) as curr_sum from(
  select c.state, sum(c.prev_census_pop) as prev_pop, sum(c.curr_census_pop)as curr_pop from
 ( select b.district, b.state,  round(b.population/(1+b.growth),0) as prev_census_pop, b.Population as curr_census_pop from
(select a.district,a.state,a.growth,b.population from projectsqlind_cen.dbo.Data1 a inner join projectsqlind_cen.dbo.Data2 b on
  a.district= b.district) b) c group by c.state
  ) d
  )n
  )e inner join(

select '1' as keyy1, z.* from (
select SUM(Area_km2) as total_area from projectsqlind_cen.dbo.Data2
)z)f on e.keyy=f.keyy1
)g


---window
-----top 3 district from each state with highest literacy rate

select a.* from(
select district, state, literacy, rank() over(partition by state order by literacy desc ) as Rnk from projectsqlind_cen.dbo.Data1
) a where a.Rnk in (1,2,3) order by state 
