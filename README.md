# AdvDBMSProject
#Project execution starts after settingup Hadoop
#For the code provided, we have installed Hadoop through Cloudera and tested the code

#copy file from local to HDFS by specifying where to save

sudo -u hdfs hadoop fs -put Real_Estate_S.csv /Advdbmspro

#We can see if got copied correctly

hdfs dfs -ls /Advdbmspro

#Change the owner to cloudera to give unrestricted access (User is 'cloudera' because we are using hadoop through cloudera)

sudo -u hdfs hadoop fs -chown -R cloudera /Advdbmspro/Real_Estate_S.csv

#give 777 access to file (Read Write Execute)

hdfs dfs -chmod 777 /Advdbmspro/Real_Estate_S.csv

#setup hive session and create a database and create a table structure to handle csv file data in that database

create database advdbmspro;
use advdbmspro;

create table realestate (development_start_date string, purchased_date string, city string, locality string, property_type string, developer_name string, yoe int, ceo string, number_of_employees int, service_amount int, rating string, investor_name string, city_investors string, sq_ft_cost int, purchase_amount int, area_purchased int, developer_id int, property_id int, user_id int) row format delimited fields terminated by ',' lines terminated by '\n' tblproperties("skip.header.line.count"="1");

#load csv file data to table created

load data inpath '/Advdbmspro/Real_Estate_S.csv' into table realestate;

#


#

create table dim_developer as select distinct row number() over() as developer_id, developer name, yoe, ceo,
number _of_employees, service_amount, rating from realestate;


create table dim_investment as select distinct row number() over() as investment_id, city_investors, investor name from
realestate;


create table dim property as select distinct row number() over() as property_ id, city, locality, property_ type from
realestate;



create table dim developmentstartdate as select row number() over() as time_id, year(from unixtime(unix_timestamp
(Development _Start Date, `dd-MMM-yy'))) as year, ceil(month(from unixtime(unix timestamp(Development start Date, 'dd-MM-
yy'))) /.3.0) as quarter, month(from unixtime(unix_timestamp(Development_Start_Date, "dd-MMM-yy'))) as month, from_unixtime
(unix_timestamp(Development Start_ Date, 'dd-MMM-yy'),'EEEE') as day_of_week, from_unixtime(unix_timestamp
(Development Start_Date, 'dd-MMM-yy')) as time fld, development start date as date_fld from realestate;




create table dim purchaseddate as select row number|) over() as time_id, year(from unixtime(unix_timestamp(Purchased_Date
dd-MMM-yy'))) as year, ceil(month(from_ unixtime(unix timestamp(Purchased_Date, 'dd-MMM-yy'))) / 3.0) as quarter, month
(from_unixtime(unix_timestamp(Purchased_Date, 'dd-MMM-yy'))) as month, from_unixtime(unix_timestamp(Purchased_Date, 'dd-MM1
yy'), 'EEEE') as day_of_week, from unixtime(unix timestamp(Purchased Date, 'dd-MMM-yy')) as time fld, purchased _date as
date fld from realestate;






create table facttable as select distinct row number() over() as fact id, dim developmentstartdate.time id as
developed_on_id, dim_purchaseddate.time_ id as bought _on_ id,dim property.property_Id as property_id,
dim developer.developer_id as developer_ id, dim investment.investment id as investment id, realestate.sq_ft_cost as
sq_ft_cost, realestate.purchase_ amount as purchase amount, realestate.area purchased as area_ purchased from realestate,
dim _property, dim developer, dim investment, dim developmentstartdate, dim purchaseddate where dim property.city
realestate.city and dim property.locality = realestate.locality and dim property.property_type = realestate.property_type
and dim developer.developer name = realestate.developer_name and dim developer.yoe = realestate.yoe and dim developer.ceo
realestate.ceo and dim developer.number_of_ employees = realestate.number_of employees and dim developer .service_amount
realestate.service_amount and dim developer.rating = realestate.rating and dim_ investment.city_investors
realestate.city_ investors and dim investment.investor name realestate. investor name and dim developmentstartdate. dat _fld
realestate.development start_dat and dim purchaseddate.date _fld = realestate.purchased date;
