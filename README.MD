# MSSQL testing database and data encription #

## Reference articles ##

[How to Encrypt SQL Server using TDE on Docker - 2019 ](https://chlee.co/how-to-encrypt-sql-server-using-tde-on-docker/)

[How to install and host SQL Server on Linux with Docker - 2019](https://chlee.co/how-to-install-and-host-sql-server-on-linux-with-docker-for-10-dollars-a-month/)

## Scope ##

Testing and training the encription features in MSSQL .

## Run ##


### Run docker file to setup database ###
```
docker-compose -f .\src\Home\docker-compose-sql.yml up -d
```


### In MSSQL Studio ##

Connect to database . To find iternal docker ip just run ipconfig. You will find it quickly ;) 


Open a new query window and run the following statments.

```

USE master;  
GO  
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'PoBj9MZunVVVH3V6sZdc';  
GO

CREATE CERTIFICATE LocalServerCertificate WITH SUBJECT = 'Local Server Certificate';  
GO



create database "local-encrypt-db"
GO

USE [local-encrypt-db]
GO  
CREATE DATABASE ENCRYPTION KEY  
WITH ALGORITHM = AES_128  
ENCRYPTION BY SERVER CERTIFICATE LocalServerCertificate;  
GO  
ALTER DATABASE [local-encrypt-db]  
SET ENCRYPTION ON;  
GO  


---  Verify the certificate and key has been created
SELECT  [name], [principal_id], [algorithm_desc], [create_date]
from master.sys.symmetric_keys

--- Verify that the database is encrypted
SELECT *  
FROM sys.dm_database_encryption_keys  
WHERE encryption_state = 3;

--- Backup the certificate.
use master
GO

BACKUP CERTIFICATE LocalServerCertificate TO FILE = 'C:\sql\certs\local-server-certificate.cert'  
    WITH PRIVATE KEY ( FILE = 'C:\sql\certs\local-server-certificate.key' ,   
    ENCRYPTION BY PASSWORD = 'PoBj9MZunVVVH3V6sZdc' );  
GO  

```

### Check if files are inside container ###

```
docker exec -it sql2017 "bash"

cd /sql/certs

ls -la
```