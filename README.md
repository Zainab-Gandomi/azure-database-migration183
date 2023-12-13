# azure-database-migration183

# Table of Contents
1. [Set up the Production Environment](#set-up-the-production-environment)
2. [Migrate to Azure SQL Database](#migrate-to-azure-sql-database)
    - [Creating a Migration Project](#creating-a-migration-project)
        - [Schema Migration](#schema-migration)
        - [Data Migration](#data-migration)
        - [Validate Migration Success](#validate-migration-success)
3. [Data Backup and Restore](#data-backup-and-restore)
    - [Backup the On-Premise Database](#backup-the-on-premise-database)
    - [Upload Backup to Blob Storage](#upload-backup-to-blob-storage)
    - [Restore Databse on Development Environmet](#restore-databse-on-development-environmet)
    - [Automate backup for Development Database](#automate-backup-for-development-database)
        - [Create the SQL Server Credential](#create-the-sql-server-credential)
        - [Create Maintenance Plans](#create-maintenance-plans)
4. [Disaster Recovery Simulation](#disaster-recovery-simulation)
    - [Mimic Data Loss in Production Environment](#mimic-data-loss-in-production-environment)
    - [Restore Database from Azure SQL Database Backup](#restore-database-from-azure-sql-database-backup)
5. [Geo Replication and Failover](#geo-replication-and-failover)
    - [Set Up Geo-replication for Azure SQL Database](#set-up-geo-replication-for-azure-sql-database)
    - [Test Failover and Tailback](#test-failover-and-tailback)
6. [Microsoft Entra Directory Integration](#microsoft-entra-directory-integration)
    - [Configure Microsoft Entra ID for Azure SQL Database](#configure-microsoft-entra-id-for-azure-sql-database)
    - [Create DB Reader User](#create-db-reader-user)







## Set up the Production Environment


In this project, a cloud-based database system on Microsoft Azure is architected and implemented.

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



## Data Backup and Restore


Companies often have two types of databases: one for storing real customer data (production database) and another for testing and experimenting (development database). By provisioning a development database, will be able to test and experiment without worrying about affecting main production data. This separation is crucial for making changes and improvements while maintaining the integrity of live data.

Before creating a development environment for the database, be ensure the data stored in production database is securely stored on Azure. And create an automated backup solution for the development environment. This solution safeguards ongoing work, aids developers in efficiently testing changes, and enables swift recovery from errors or data loss.


### Backup the On-Premise Database


Now, the process involves backing up a local on-premise database and ensuring data protection through the implementation of best practices for managing backup files locally.

Open SQL Server Management Studio (SSMS) and right-click on the *AdvanturWorks* database you want to back up and select *Tasks > Back Up*...

![Screenshot (634)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/66850810-b85b-4a14-b05d-fcdd2d38f517)


In the Back Up Database dialog, choose a destination for the backup file, backup type and Database.

![Screenshot (635)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/2e32148e-03c0-4c45-b9c9-211e6e1725d9)


### Upload Backup to Blob Storage


By uploading the on-premise backup file to Azure Blob Storage,can secure backups in the cloud,and ensure data durability and availability. This enables to implement an off-site backup strategy, enhancing the resilience of data protection plan.

To upload backup in blob storage, first, set up a storage account and container. Then by click on the Upload button or drag and drop the on-premise backup file from local machine to the container.


![Screenshot (638)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/4f530064-db8a-4387-bab6-b962d0e8c916)


![Screenshot (639)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/3b5e36de-aec7-44a7-b38d-77e130654562)



### Restore Databse on Development Environmet


The ability to restore a database to a known good state is essential for handling various scenarios, such as accidental data deletion, hardware failures, or data corruption due to software bugs or malicious activities.

Think of the development environment as an area for controlled experimentation, much like a sandbox. In software development, a sandbox is a controlled and isolated environment where applications and software can be tested, developed, and experimented with, all without impacting the production systems.

To replicate this environment, provision a new Windows VM that mirrors the development setup. Install SQL Server on this VM to mimic the necessary database infrastructure.

A development environment is a controlled and isolated space where software developers and testers can work on applications, test new features, and identify and fix bugs without affecting the production environment

To restore database on development environment first create another Windows virtual machine. Then download the on-premise database backup file (AdvantureWorks) stored in Azure Blob Storage locally.

![Screenshot (643)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/ad70a60d-1b5e-4a50-8253-bf5a5b59feee)

Download SQL Server Developer and SQL Server Management Studio (SSMS) on the development VM like before.Open SQL Server Management Studio (SSMS) and connect to the SQL Server instance installed on the VM. And right-click on the Database node of the Server and select Restore Database... In the Restore Database dialog, select Device as the source for the backup file.

![Screenshot (645)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/7311eb8f-7948-4e90-abb2-e06a8d8c5b5d)


Now check the data and run tests to ensure the database is working correctly.


### Automate backup for Development Database


SQL Server Maintenance Plans are a powerful feature of SQL Server Management Studio (SSMS) that allow database administrators to automate routine maintenance tasks for SQL Server databases. These maintenance tasks include operations such as database backups, index maintenance, database integrity checks, and statistics updates.

Now start to create periodic backups for our development AdvantureWorks database and stored these backups in an Azure Storage Container.

In the SSMS go to Object Explorer, right-click on the SQL Server Agent node and then click Start. SQL Server Agent is a component of Microsoft SQL Server that enables the scheduling, automation, and management of various administrative tasks, jobs, and alerts within SQL Server.

Check Agent is started successfully by expand the SQL Server Agent node and see the subsection.


#### Create the SQL Server Credential


A SQL Server Credential is a security object that allows SQL Server to access external resources securely. It is particularly useful when you need to interact with external systems, such as Azure Blob Storage, to store or retrieve data.

To create a SQL Server Credential follow these steps:

Open SSMS and connect to your SQL Server instance. Right-click on the server name and select New Query to open a new query window. Then execute the following T-SQL command to create the credentials:

'''

        CREATE CREDENTIAL CredentialToAutomateBackup
        WITH IDENTITY = 'aicoreproject08122023',
        SECRET = 'Access Key';


'''
 
Access key can be found by navigating to the Storage Account in the Azure portal, under *Security + networking > Access key*.

After creating the SQL Server Credential, should be able to see this under the Security *node > Credentials*, with the name is provisioned the Credential with.

![Screenshot (646)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/82f56cc3-f046-450d-a146-2e45501fc874)

With the SQL Server Credential in place, SQL Server will be able to access the Azure Storage Account using the secret access key.


#### Create Maintenance Plans


To continue creating a management task in SSMS for periodic backups and configure it to store backups in Azure Blob Storage, in the Object Explorer, expand the *Management* node, right-click on *Maintenance Plans*, and select *Maintenance Plan Wizard*.

Configure Maintenance Plan Wizard,with provide a name for the plan, then change the schedule click *Change*, this will open a new window where you can set up the schedule: daily, weekly or monthly, start time and end time, etc. Configure a weekly backup schedule to ensure consistent protection for your evolving work and simplify recovery for your development environment if needed.

![Screenshot (651)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/f04bc38b-6933-4cf9-99d7-3d05cf323fe5)

Select Back Up Database (Full) to configure the backup task. Then In the Back Up Database Task dialog, select the databases, and choose URL as the backup destination.

In the Destination tab you will first have to select the SQL Server Credentials from a drop-down list. Here you should select the credentials you have provisioned before, that will allow you access to the Azure Storage Account.

![Screenshot (653)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/5b2922f2-e79b-4587-bbe5-113e0314bae2)

Under Azure storage container fill in the name of the container you want to upload the backups to. You can choose a unique container name that suits your project. For this example, I will use the container mycontainermaya

![Screenshot (654)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/94f7121a-501c-4e00-8997-f8b588220d76)

Following photo shows that the maintenance plan executes successfully.

![Screenshot (652)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/b5ab5174-88f7-41b3-87ec-1d0e0981420f)


## Disaster Recovery Simulation

Disaster recovery is a critical aspect of any organization's data strategy. It refers to the set of processes and tools implemented to safeguard data, applications, and systems, enabling rapid recovery and continuity of operations in the face of disruptive events. By having a robust disaster recovery plan, businesses can minimize downtime, ensure data integrity, and maintain their ability to serve customers without significant interruptions.


### Mimic Data Loss in Production Environment

To perform a disaster recovery simulation, intentionaly mimiking data loss in production environment.

Deliberately remove critical data from production database to replicate a scenario where data integrity is compromised.

first senario: There is *Production.ProductCostHistory* table and we want to update the price of the product with ProductID = 708 but by accident we update the price of the ProductID = 707.

![Screenshot (661)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/c63430be-052a-4faa-9fc6-6b18dd0a1857)

![Screenshot (662)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/6763a074-8b24-4ac0-b243-cfc287f9eb9d)

Other senario: We need to delete the 100 of top row data but instead we delete 1000 of them on *dbo.DatabaseLog* table.

![Screenshot (666)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/56ff4bc8-9b19-4191-84ae-0f353e86620c)



### Restore Database from Azure SQL Database Backup

Navigate to the Azure portal and from the Azure SQL Database dashboard, locate and select the target database that needs restoration. From the SQL Database Home Page select the *Restore* option at the top bar on the page.

Then choose the restore point that represents the point in time before the data loss occurred. select a restore point ( for this project is on the same day one hours before the data loss has occurred) as no data has been added to the database since then, however if this is an active database that gets updated real-time would have to choose the closest point in time before data loss has occurred for minimal data loss.

![Screenshot (667)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/4447135e-d0c7-4f3c-94c7-726b6097d73e)

Next, choose the new Database name, After that just click *Review + create* and finally click *Create*. Azure will begin restoring the selected backup to the specified destination. This will take a couple of minutes to complete.

To verify the database has been restored to a correct point in time, before the data loss has occurred, will establish a connection to it using Azure Data Studio.

![Screenshot (668)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/c948d87e-3a04-42af-9437-ad33fe064e64)


To verify that a database has been restored correctly, select the *dbo.DatabaseLog* table and confirm that the number of rows is 1596, matching the number of rows before they were deleted.


## Geo Replication and Failover

Geo-replication enhances data protection by establishing a synchronized copy of production Azure SQL Database in a secondary region. This strategic redundancy ensures continuous data availability and minimizes potential downtime during unforeseen disruptions.

A planned failover to the secondary region allows to assess the availability and consistency of the secondary database. This process provides valuable insights into the failover mechanism and its impact on data ecosystem.


### Set Up Geo-replication for Azure SQL Database

Follow these steps to configure and set up georeplication for Azure SQL Database:

 - Navigate to the Azure portal and from the Azure SQL Database dashboard, select the primary database (*restored production database*) to replicate.
 - In the left-hand menu of the primary database blade, click on *Replicas* under *Data Management*.
 - Click on *Create replica* to begin the setup process. In the Geo Replica menu, first have to create a new SQL Server and this server will should be located in a region geographically distant from the primary region to ensure data redundancy.
 - Select authentication method and create the server 
 - Click *Review + create* and then click on *Create* to initiate the replication process.

![Screenshot (671)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/3cfee1a9-08c3-43f6-aa61-cebec5fdbc83)


### Test Failover and Tailback

Failover is the process of switching the workload from the primary region to the secondary region in a georeplicated environment

It is typically performed during planned maintenance or in response to a disaster in the primary region. Failover ensures high availability and business continuity by allowing applications to continue running from the secondary region.

A test failover is a non-disruptive way to verify the functionality of the failover environment without impacting the production workload. Follow these steps to initiate a test failover to the secondary region:

 - Navigate to the Azure portal and from the Azure SQL Database dashboard, select the SQL server associated with the primary database *production-database-restored* that want to failover (In this case, the database on UK South).
 - Select *Failover groups* under the *Data management* pane, and then select Add group to create a new failover group.

 - On the Failover group page enter a name for the failover group. Then for the Server select secondary server, not the server on which the primary database resides in.

 - Now navigate to the Azure SQL Server in which your secondary/replication database resides in. The newly created failover group is under Failover groups.

  Access the failover group page, should be able to see the following page, which indicates the two servers in the failover group and which is the primary and the secondary one:

![Screenshot (676)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/e1cc4139-3ea7-4933-add0-c88732ef0124)


To initiate a planned failover (without data loss), select Failover from the task pane to fail over the failover group containing the database. Then receive a warning about switching the secondary database to a primary role click Yes to continue here.

Wait for the failover to complete and review which server is now primary and which server is secondary. If failover succeeded, the two servers should have swapped roles. Also can use the connection details of the secondary server to connect to the new primary database, and run tests to validate the database's functionality in the secondary region, ensuring that it is working as expected.

![Screenshot (677)](https://github.com/Zainab-Gandomi/azure-database-migration183/assets/79536268/8cdf0099-acda-4ff7-b215-209690518caa)


Tailback is the process of reverting the workload back to the primary region after a successful failover.

Once the primary region is restored, the workload is switched back from the secondary region to the primary region, ensuring that the original production environment is reinstated.

Select *Failover* again to fail the servers back to their original roles.


By following these steps, can safely perform a test failover to verify disaster recovery environment's functionality and then perform a tailback to revert to the primary region. Testing these processes regularly ensures the readiness of disaster recovery plan and boosts confidence in organization's ability to handle unexpected incidents.




## Microsoft Entra Directory Integration

### Configure Microsoft Entra ID for Azure SQL Database

### Create DB Reader User



