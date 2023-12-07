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

