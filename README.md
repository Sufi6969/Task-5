# Task-5
Exploratory Data Analysis - Sports : Task 5 : By Sufi Sheikh


# Code

proc import datafile="/home/u63391350/TSF/deliveries.csv" out=Delivery  dbms=csv replace;
    getnames=yes;
run;
proc import datafile="/home/u63391350/TSF/matches.csv" out=Matches  dbms=csv replace;
    getnames=yes;
run;

/* Filter and keep rows where winner is not null */
proc sql;
  create table mat_filt as
  select *
  from matches
  where not missing(winner);
quit;

/* Delivery Filtered */
proc sql;
  create table Del_filt as
  select *
  from delivery
  where not missing(match_id);
quit;

/******************************MErging data**********************/
data Match_id;
set matches;
rename id=match_id;
run;

data Merged_data;
merge Delivery (in=d)
	  Match_id (in=m);
by match_id;
if m or d;
run;

/******************************* Top Team to Endorse ******************/
proc sql;
  create table team_wins as
  select winner, count(*) as total_wins
  from mat_filt
  group by winner
  order by total_wins desc;
quit;
proc print data=team_wins (obs=10);
title"Top Team to Endorse";
run;

/***************************** Top Player to Endrose *****************/
proc sql;
create table player_of_match as
select  player_of_match, count(*) as Best_Player
from matches 
where Win_by_runs >=1 or win_by_wickets >=1
group by player_of_match
order by Best_Player desc;
run;
proc print data=player_of_match;
 where monotonic() <= 5;
 title "Top 5 Players to Endorse";
quit;


 
/************************* Gayle till 2017 after 2017 ********************/

data Gayle17 Gayle18 Others;
set mat_filt;
if season > 2007 and season < 2018 and player_of_match = "CH Gayle" then output Gayle17;
else if season >=2018 and player_of_match = "CH Gayle" then output Gayle18;
else output Others;
run;
proc sql;
create table Gayle17 as
select distinct player_of_match, count(*)  as BestPlayer17
from Gayle17;
run;
proc sql;
create table Gayle18 as
select distinct player_of_match, count(*)  as BestPlayer18
from Gayle18;
run;

proc print data= gayle17 ;
title"Gayle best player title between 2008-2017";
run;
proc print data= gayle18 ;
title"Gayle best player title in 2018 & 2019";
run;

/******************************** Number of matches per season ************************************/
proc sql;
create table Season_matches as
select season,count(*) as num_matches
from matches
group by season
order by num_matches desc;
quit;
proc print data= Season_matches;
title"Matches Per Season";
run;


/*********************************** Played_per_Season_csk ******************************************/
proc sql;
create table Played_per_Season_csk as
select season, count(*) as played_csk 
from mat_filt
where team1= "Chennai Super Kings" or team2="Chennai Super Kings"
group by season;
run;
proc print data=Played_per_Season_csk;
title"Played_per_Season_Csk";
sum played_csk;
run;
/* Won_per_Season_csk */
proc sql;
create table Won_per_Season_csk as
select season, count(*) as num_wins 
from mat_filt
where winner="Chennai Super Kings" 
group by season;
run;

proc sql;
select mean(num_wins) as Average_Wins
from Won_per_Season_csk;
quit;

proc print data=Won_per_Season_csk;
title"Won_per_Season_Csk";
sum num_wins;
run;




/**************************************** Played_per_Season_MI **************************************/
proc sql;
create table Played_per_Season_mi as
select season, count(*) as played_mi 
from mat_filt
where team1= "Mumbai Indians" or team2="Mumbai Indians"
group by season;
run;
proc print data=Played_per_Season_mi;
title"Played_per_Season_MI";
sum played_mi;
run;
/* Won_per_Season_MI */
proc sql;
create table Won_per_Season_mi as
select season, count(*) as num_wins 
from mat_filt
where winner="Mumbai Indians" 
group by season;
run;
proc print data=Won_per_Season_mi;
title"Won_per_Season_MI";
sum num_wins;
run;


/********************************************* Top Batsman ***************************************/
proc sql;
create table Batting as
select distinct batsman, count(*) as Total_batting
from del_filt 
group by batsman
order by Total_batting desc;
quit;
proc print data=batting (obs=5 );
title"Top 5 Batsman";
run;


/*************************** Calculate total runs by player per season ************************/
proc sql;
    create table tot_r_season as
    select season, batsman, sum(total_runs) as total_runs
    from merged_data
    group by season, batsman
    order by season, total_runs desc;
quit;


/************************** Total Contribution to Overall Run per Season *****************/
proc sql;
    create table tot_r_by_season as
    select season, sum(total_runs) as total_runs
    from merged_data
    group by season
    order by season;
quit;
proc sql;
    create table ms_dhoni_percentage as
    select A.season, 
           A.batsman, 
           A.total_runs as dhoni_runs, 
           B.total_runs as total_runs_season,
           (A.total_runs / B.total_runs) * 100 as percentage_runs
    from tot_r_season A
    left join tot_r_by_season B
    on A.season = B.season
    where A.batsman = "MS Dhoni"
    order by A.season;
quit;
proc print data=ms_dhoni_percentage(keep=Season batsman percentage_runs);
title"MS Dhoni's Run Contribution of Total Run";
run;

proc means data=ms_dhoni_percentage mean;
var percentage_runs;
title "Average Run Contribution of MS_Dhoni from 2008-2019";
run;

/* R Sharma */
proc sql;
    create table rg_sharma_percentage as
    select A.season, 
           A.batsman, 
           A.total_runs as rg_runs, 
           B.total_runs as total_runs_season,
           (A.total_runs / B.total_runs) * 100 as percentage_runs
    from tot_r_season A
    left join tot_r_by_season B
    on A.season = B.season
    where A.batsman = "RG Sharma"
    order by A.season;
quit;
proc print data=rg_sharma_percentage(keep=Season batsman percentage_runs);
title"RG Sharma's Run Contribution of Total Run";
run;
proc means data=rg_sharma_percentage mean;
var percentage_runs;
title "Average Run Contribution of RG_Sharma from 2008-2019";
run;



/******************************* Top Team to Endorse ******************/

proc print data=team_wins (obs=1);
title" CASE I : Top Team to Endorse";
run;

/***************************** Top Player to Endrose *****************/

proc print data=player_of_match;
 where monotonic() <= 1;
 title " CASE II : Top Player to Endorse";
quit;

/********************************************* Top Batsman ***************************************/

proc print data=batting (obs=1 );
title" CASE III : Top Batsman to Endorse";
run;

/************************** Total Contribution to Overall Run per Season *****************/

proc print data=player_of_match;
where monotonic()=3  ;
 title " Case VI : Allrounder Player to Endorse";
quit;


