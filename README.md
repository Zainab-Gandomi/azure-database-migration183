# azure-database-migration183

# Table of Contents
1. [Set up the Production Environment](#set-up-the-production-environment)
2. [Migrate to Azure SQL Database](#migrate-to-azure-sql-database)
    - [Creating a Migration Project](#creating-a-migration-project)
        - [Schema Migration](#schema-migration)
        - [Data Migration](#data-migration)
        - [Validate Migration Success](#validate-migration-success)




## Set up the Production Environment

In this project, a cloud-based database system on Microsoft Azure is architected and implemented.

GitHub use to track changes to code and save them online in a GitHub repo.

Different services running on Azure: 
first Set up a Windows Virtual Machine (VM) to serve as the cornerstone of production environment,and then Install SQL Server and SQL Server Management Studio (SSMS) on the VM. These tools are essential for proficient database management within production environment, mirroring the capabilities of a corporate database server.

Download the SQL Server Developer installer from the[Microsoft Download Center](https://colab.research.google.com/corgiredirector?site=https%3A%2F%2Fgo.microsoft.com%2Ffwlink%2Fp%2F%3Flinkid%3D2215158%26clcid%3D0x809%26culture%3Den-gb%26country%3Dgb)

The production database create by restoring it from backup file. 


## Migrate to Azure SQL Database


The imperative migrate of databases from on-premise to the cloud, driven by businesses embracing cloud computing for innovation and scalability, necessitates a powerful tool like Microsoft's Azure Data Studio, designed to simplify database management and streamline migration processes for a seamless transition.


To install Azure Data Studio for previously created Windows Azure VM, follow this link [Aure Data Studio](https://colab.research.google.com/corgiredirector?site=https%3A%2F%2Fgo.microsoft.com%2Ffwlink%2F%3Flinkid%3D2242848).


Using Azure Data Studio establish a connection to the newly created Azure SQL Database. This connection servers as the conduit for schema and data migration between the two databases.


### Creating a Migration Project

In Azure Data Studio, the initiation of the database migration process involves creating a migration project using the SQL Server Schema Compare extension, a powerful tool facilitating the comparison and application of schema changes between the local database and the SQL Database project within Azure Data Studio.


#### Schema Migration

Migration start by migrating the schema of the local database to the Azure-hosted database. To do this, first click on the Extensions icon in the Activity Bar. Search for and install the **SQL Server Schema Compare** extension if it is not already installed.

To compare schema the Source corresponds to local SQL Server and the desired local database, while the Target corresponds to the Azure SQL Server and the desired Azure SQL Database.Click on the Compare button at the top of the Schema Compare page

Select the specific schema changes that want to apply to the Azure SQL Database project (in our case all the changes). Apply the selected schema changes to synchronize the schema between the local database and the Azure SQL Database project by clicking on the Apply button at the top of the Schema Compare page. 

If everything works as expected will received the following message *Apply schema compare changes succeeded* , and should be able to see the new schema in the established Azure SQL connection but still no data in it.

#### Data Migration

With the successful execution of the schema migration, it is time to move forward with the data migration phase. In Azure Data Studio, data migration plays a crucial role in moving data from a local SQL Server database to an Azure-hosted database. To facilitate this process, **Azure SQL Migration** extension, a powerful tool is designed to streamline and simplify the migration of data between these environments.

After install extention, Click on the Server icon and select local SQL Server database.Right-click on the server and select *Manage*.Under *General* click on *Azure SQL Migration* and then click on the Migrate to Azure SQL button.

In Azure SQLMigration wizard following steps to set sourse and target and then need to create Azure Database Migration Service at Azure portal, which will orchestrate the database migration. 

To connect Database Migration Service, first, download and install integration runtime by following the provided link. Once the installation is completedby using one of the two keys provided in Azure Data Studio to register the integration runtime.

Before proceeding to the final step click the Run validation button to run and validate the migration settings. The validation step ensures that all the migration settings are correct and helps identify and resolve any potential issues that could arise during the migration process. Once the validation is successful, press Start migration to migrate the data from the local to the cloud-hosted database.

#### Validate Migration Success

To confirm the success of the database migration process, carry out a comprehensive validation. Right-click on any table and pressing *Select Top 1000*. And now be able to see the data stored in the Azure SQL Database.


![Screenshot (633)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/d0aeaca1-59c2-4d07-90dc-2670584c9f7e)

