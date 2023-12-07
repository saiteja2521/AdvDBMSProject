# AdvDBMSProject

#Project execution starts after settingup Hadoop

#For the code provided, we have installed Hadoop through Cloudera and tested the code

**#copy file from local to HDFS by specifying where to save**

sudo -u hdfs hadoop fs -put Real_Estate_S.csv /Advdbmspro

**#We can see if got copied correctly**

hdfs dfs -ls /Advdbmspro

**#Change the owner to cloudera to give unrestricted access (User is 'cloudera' because we are using hadoop through cloudera)**

sudo -u hdfs hadoop fs -chown -R cloudera /Advdbmspro/Real_Estate_S.csv

**#give 777 access to file (Read Write Execute)**

hdfs dfs -chmod 777 /Advdbmspro/Real_Estate_S.csv

**#setup hive session and create a database and create a table structure to handle csv file data in that database**

create database advdbmspro;
use advdbmspro;

create table realestate (development_start_date string, purchased_date string, city string, locality string, property_type string, developer_name string, yoe int, ceo string, number_of_employees int, service_amount int, rating string, investor_name string, city_investors string, sq_ft_cost int, purchase_amount int, area_purchased int, developer_id int, property_id int, user_id int) row format delimited fields terminated by ',' lines terminated by '\n' tblproperties("skip.header.line.count"="1");

**#load csv file data to table created**

load data inpath '/Advdbmspro/Real_Estate_S.csv' into table realestate;
#or
hdfs dfs -cp /Advdbmspro/Real_Estate_S.csv /user/hive/warehouse/advdbmspro.db/realestate

**#run queries on realestate table to check runtime**

select sum(area_purchased) as area,property_type, city from realestate group by property_type, city order area desc;

**# divide root file to fact and dimension tables**

create table dim_developer as select distinct row_number() over() as developer_id, developer_name, yoe, ceo, number_of_employees, service_amount, rating from realestate;

create table dim_investment as select distinct row_number() over() as investment_id, city_investors, investor_name from realestate;

create table dim_property as select distinct row_number() over() as property_id, city, locality, property_type from realestate;

create table dim_developmentstartdate as select row_number() over() as time_id, year(from unixtime(unix_timestamp(Development_Start_Date, 'dd-MMM-yy'))) as year, ceil(month(from unixtime(unix timestamp(Development_start_Date, 'dd-MMM-yy'))) /.3.0) as quarter, month(from unixtime(unix_timestamp(Development_Start_Date, 'dd-MMM-yy'))) as month, from_unixtime (unix_timestamp(Development_Start_Date, 'dd-MMM-yy'),'EEEE') as day_of_week, from_unixtime(unix_timestamp (Development_Start_Date, 'dd-MMM-yy')) as time_fld, development_start_date as date_fld from realestate;

create table dim_purchaseddate as select row_number() over() as time_id, year(from unixtime(unix_timestamp(Purchased_Date, dd-MMM-yy'))) as year, ceil(month(from_ unixtime(unix timestamp(Purchased_Date, 'dd-MMM-yy'))) / 3.0) as quarter, month(from_unixtime(unix_timestamp(Purchased_Date, 'dd-MMM-yy'))) as month, from_unixtime(unix_timestamp(Purchased_Date, 'dd-MMM-yy'), 'EEEE') as day_of_week, from unixtime(unix timestamp(Purchased Date, 'dd-MMM-yy')) as time fld, purchased _date as
date_fld from realestate;

create table facttable as select distinct row_number() over() as fact id, dim developmentstartdate.time_id as developed_on_id, dim_purchaseddate.time_ id as bought_on_id, dim_property.property_Id as property_id, dim_developer.developer_id as developer_id, dim_investment.investment_id as investment_id, realestate.sq_ft_cost as sq_ft_cost, realestate.purchase_amount as purchase_amount, realestate.area_purchased as area_ purchased from realestate, dim_property, dim_developer, dim_investment, dim_developmentstartdate, dim_purchaseddate where dim_property.city = realestate.city and dim_property.locality = realestate.locality and dim_property.property_type = realestate.property_type and dim_developer.developer_name = realestate.developer_name and dim developer.yoe = realestate.yoe and dim developer.ceo = realestate.ceo and dim_developer.number_of_employees = realestate.number_of_employees and dim_developer.service_amount = realestate.service_amount and dim_developer.rating = realestate.rating and dim_investment.city_investors = realestate.city_investors and dim_investment.investor_name = realestate.investor_name and dim_developmentstartdate.date_fld = realestate.development_start_date and dim_purchaseddate.date_fld = realestate.purchased date;


**#run queries after structuring data to analyse runtime**

select sum(area_purchased) as area,property_type, city from facttable a, dim_property b where a.property_id = b.property_id group by property_type, city order area desc;


**#Divide fact and dimension tables to partitioning and bucketing**

create table dim_property_partitioned_bucketed (property_id int, locality string, property_type string) partitioned by (city string) clustured by (property_type string) into 2 buckets;


**#check runtime after executing partitioning and bucketing**

select sum(area_purchased) as area,property_type, city from facttable a, dim_property_partitioned_bucketed b where a.property_id = b.property_id group by property_type, city order area desc;







#instructions to run queries through Hive

use advdbmspro;

select sum(area_purchased) as area,property_type, city from realestate group by property_type, city order area desc;
select sum(area_purchased) as area,property_type, city from facttable a, dim_property b where a.property_id = b.property_id group by property_type, city order area desc;
select sum(area_purchased) as area,property_type, city from facttable a, dim_property_partitioned_bucketed b where a.property_id = b.property_id group by property_type, city order area desc;



#instructions to run queries through Spark

spark-shell

import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.sql.hive.HiveContext

val conf = new SparkConf().setAppName("HiveWithSpark")
val sc = new SparkContext(conf)
val hiveContext = new HiveContext(sc)

val results = hiveContext.sql("SELECT * FROM your_hive_database.your_hive_table")
results.show()n
