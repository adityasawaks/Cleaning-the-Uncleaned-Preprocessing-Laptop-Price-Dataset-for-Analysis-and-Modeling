
# The Uncleaned Laptop Price dataset is a collection of laptop product listings scraped from an online e-commerce website. The dataset includes information about various laptop models, such as their brand, screen size, processor, memory, storage capacity, operating system, and price. However, the dataset is uncleaned, meaning that it contains missing values, inconsistent formatting, and other errors that need to be addressed before the data can be used for analysis or modeling.https://www.kaggle.com/datasets/ehtishamsadiq/uncleaned-laptop-price-dataset

The SQL script performs various operations on a table called "laptopdata" in a database schema named "sqlproject".

First, the script creates a backup of the laptopdata table and drops an unnecessary column. Then, it modifies several columns, cleaning up and standardizing their values using string functions, and adding new columns for the parsed values.

The script also performs conditional updates on the "OpSys" column to standardize operating system names and adds new columns for GPU and CPU brand and model information parsed from existing columns. The script also adds new columns for screen resolution, touchscreen capability, and memory type parsed from existing columns.

Finally, the script updates the "memory_type" column based on the "Memory" column values, classifying them into different types such as "HDD", "SSD", "Hybrid", "Flash Storage", or null.

SELECT COUNT(*) FROM sqlproject.laptopdata;


CREATE TABLE BACKUP LIKE laptopdata;


INSERT INTO BACKUP
SELECT * FROM LAPTOPDATA;

select * from information_schema.tables
where TABLE_SCHEMA='sqlproject'
AND table_NAME='LAPTOPDATA';

ALTER TABLE laptopdata DROP column `Unnamed: 0`;

select * FROM laptopdata;


delete FROM laptopdata WHERE Price=0;

select COUNT(*) FROM laptopdata;

select * FROM laptopdata;

ALTER TABLE laptopdata modify column inches decimal(10,1);

select * FROM laptopdata;

update laptopdata A2
SET Ram = ( select replace(Ram,'GB','') FROM laptopdata A1 WHERE A1.index=A2.index) ;


ALTER TABLE laptopdata modify COLUMN RAM INTEGER ;

select * FROM laptopdata;

update laptopdata A2
SET Weight = ( select replace(Weight,'KG','') FROM laptopdata A1 WHERE A1.index=A2.index) ;

select * FROM laptopdata;

update laptopdata A2
SET Weight = ( select replace(Weight,'kg',' ') FROM laptopdata A1 WHERE A1.index=A2.index) ;

select * from laptopdata;

ALTER TABLE laptopdata modify COLUMN Weight INTEGER;

select * from laptopdata;

update laptopdata A2
SET Price = ( select round(Price) FROM laptopdata A1 WHERE A1.index=A2.index) ;


select * from laptopdata;

alter table laptopdata modify Price integer;

update laptopdata
set OpSys=case
    when OpSys like '%mac%' then 'macos'
    when OpSys like 'window%' then 'windows'
    when OpSys like'%linux%' then 'linux'
    when OpSys='no os' then 'N/A'
    ELSE 'other'
END;

select * from laptopdata;


alter table laptopdata
add column gpu_brand VARCHAR(255) after Gpu,
add column gpu_name VARCHAR(255) after gpu_brand;

select * from laptopdata;

update laptopdata a1
set gpu_brand = ( select substring_index(Gpu,' ',1) from laptopdata a2 WHERE a2.index=a1.index );

select * from laptopdata;

update laptopdata a1
set gpu_name = ( select replace(Gpu,gpu_brand,' ') from laptopdata a2 WHERE a2.index=a1.index );

select * from laptopdata;

ALTER TABLE laptopdata DROP column `Gpu`;

select * from laptopdata;

alter table laptopdata
add column cpu_brand VARCHAR(255) after Cpu,
add column cpu_name VARCHAR(255) after cpu_brand,
add column cpu_speed DECIMAL(10,1) after Cpu_name;

select * from laptopdata;

update laptopdata a1
set cpu_brand = ( select substring_index(Cpu,' ',1) from laptopdata a2 WHERE a2.index=a1.index );

update laptopdata a1
set cpu_speed = ( select CAST(replace(substring_index(Cpu,' ',-1),'GHz',' ') as Decimal(10,2)) from laptopdata a2 WHERE a2.index=a1.index );


update laptopdata a1
set cpu_name = ( select replace(replace(Cpu,cpu_brand,' '),substring_index(Cpu,' ',-1),' ') from laptopdata a2 WHERE a2.index=a1.index );


select * from laptopdata;

ALTER TABLE laptopdata DROP column `Cpu`;


select * from laptopdata;

select substring_index(screenResolution,' ',-1),substring_index(substring_index(screenResolution,' ',-1),'x',-1),substring_index(substring_index(screenResolution,' ',-1),'x',1) from laptopdata;

alter table laptopdata
add column resolution_width integer after screenResolution,
add column resolution_height integer after resolution_width;

update laptopdata a1
set resolution_width = ( select substring_index(substring_index(screenResolution,' ',-1),'x',1) from laptopdata a2 WHERE a2.index=a1.index ),
resolution_height = ( select substring_index(substring_index(screenResolution,' ',-1),'x',-1) from laptopdata a2 WHERE a2.index=a1.index );


select * from laptopdata;

alter table laptopdata
add column touchscreen INTEGER AFTER resolution_height;

select * from laptopdata;

select ScreenResolution LIKE '%Touch%' from laptopdata;

update laptopdata a1
set touchscreen = ( select ScreenResolution LIKE '%Touch%' from laptopdata a2 WHERE a2.index=a1.index );

select * from laptopdata;

alter table laptopdata
drop column ScreenResolution;

select * from laptopdata;

select cpu_name,substring_index(trim(cpu_name),' ',2) from laptopdata;

update laptopdata
set cpu_name = substring_index(trim(cpu_name),' ',2);

select * from laptopdata;

Alter table laptopdata
add column memory_type VARCHAR(255) AFTER Memory,
add column primary_storage INTEGER AFTER memory_type,
add column secondary_storage integer AFTER primary_storage;


select * from laptopdata;

SELECT Memory,
CASE
     WHEN Memory like '%SSD%' AND Memory like '%HDD%' THEN 'Hybrid' 
     WHEN Memory like '%SSD%'  THEN 'SSD' 
     WHEN Memory like '%HDD%' THEN 'HDD'
     WHEN Memory like '%Hybrid%' THEN 'Hybrid' 
     WHEN Memory like '%Flash Storage%' THEN 'Flash Storage'
     WHEN Memory like '%Flash Storage%' AND Memory like '%HDD%'  THEN 'Hybrid'
     else null
end as 'memory_type'
from laptopdata;

update laptopdata
set memory_type= CASE
     WHEN Memory like '%SSD%' AND Memory like '%HDD%' THEN 'Hybrid' 
     WHEN Memory like '%SSD%'  THEN 'SSD' 
     WHEN Memory like '%HDD%' THEN 'HDD'
     WHEN Memory like '%Hybrid%' THEN 'Hybrid' 
     WHEN Memory like '%Flash Storage%' THEN 'Flash Storage'
     WHEN Memory like '%Flash Storage%' AND Memory like '%HDD%'  THEN 'Hybrid'
     else null
end ;

select * from laptopdata;

select Memory,
substring_index(Memory,'+',1),
case when Memory like '%+%' then substring_index(Memory,'+',-1) else 0 
END FROM  laptopdata;

select * from laptopdata;

update laptopdata
set primary_storage=substring_index(Memory,'+',1),
secondary_storage=case when Memory like '%+%' then substring_index(Memory,'+',-1) else 0 
END;

select * from laptopdata;


select primary_storage,
case when primary_storage <=2 then primary_storage*1024 else  primary_storage end from laptopdata;

update laptopdata a1
set primary_storage=case when primary_storage <=2 then primary_storage*1024 else  primary_storage end;

select * from laptopdata;

select secondary_storage,
case when secondary_storage <=2 then secondary_storage*1024 else  secondary_storage end from laptopdata;

update laptopdata a1
set secondary_storage=case when secondary_storage <=2 then secondary_storage*1024 else  secondary_storage end;

select * from laptopdata;


alter table laptopdata Drop column Memory;

select * from laptopdata;

alter table laptopdata Drop column gpu_name;

select * from laptopdata;
