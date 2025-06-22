### **Task 1. Enable the Data Catalog API**
In the Google Cloud console, click on the Navigation menu (Navigation menu icon) > APIs & Services > Library.

In the search bar, enter Data Catalog, and select the Google Cloud Data Catalog API.

Click Enable.
![Screenshot 2025-06-21 5 20 42 PM](https://github.com/user-attachments/assets/26a2c72c-4610-48c9-aed7-c072df2082bf)

Note: If you run into an "Action Failed" error after trying to enable the Data Catalog API, try the following steps: click close, refresh your browser tab, then click Enable again.
Click Check my progress to verify the objective.

Enable the Data Catalog API
![Screenshot 2025-06-21 5 24 02 PM](https://github.com/user-attachments/assets/7b42a0df-0b61-4893-9585-11e553bff97a)


### **Task 2. Connect PostgreSQL to Dataplex Universal Catalog**
Create a variable for Project ID
In Cloud Shell, run the following command to set your Project ID as an environment variable:
```bash
export PROJECT_ID=$(gcloud config get-value project)
```
![Screenshot 2025-06-21 5 25 40 PM](https://github.com/user-attachments/assets/fc499b72-acba-4df1-97b4-8de7d2052606)

Create the PostgreSQL Database
Run the following command to clone the Github repository:
```bash
gsutil cp gs://spls/gsp814/cloudsql-postgresql-tooling.zip .
unzip cloudsql-postgresql-tooling.zip
```
Next, change your current working directory to the cloned repo directory:
```bash
cd cloudsql-postgresql-tooling/infrastructure/terraform
```
![Screenshot 2025-06-21 5 27 24 PM](https://github.com/user-attachments/assets/241ed8d3-6e76-437d-9e9b-cf56b4962672)

Run the following commands to change the region and zone from us-central1 and us-central1-a to your default assigned region and zone:
```bash
export REGION=us-west1
export ZONE=us-west1-a
sed -i "s/$REGION-a/$ZONE/g" variables.tf
```
Next, execute the init-db.sh script:

```bash
cd ~/cloudsql-postgresql-tooling
bash init-db.sh
```
This will create your PostgreSQL instance and populate it with a random schema. This can take around 10 to 15 minutes to complete.

![Screenshot 2025-06-21 5 28 15 PM](https://github.com/user-attachments/assets/96d4b10a-f3d8-4f05-babb-994af14c61a7)

![Screenshot 2025-06-21 5 29 33 PM](https://github.com/user-attachments/assets/88ddd2c8-9fb0-4c9f-8ea4-80e0981d5072)

Click Check my progress to verify the objective.

Create the PostgreSQL Database

![Screenshot 2025-06-21 5 31 02 PM](https://github.com/user-attachments/assets/58d89bf9-b932-4011-99b6-7a0995e4a761)


**Set up the Service Account**
Create a Service Account:
```bash
gcloud iam service-accounts create postgresql2dc-credentials \
--display-name  "Service Account for PostgreSQL to Data Catalog connector" \
--project $PROJECT_ID
```
Next, create and download the Service Account Key:
```bash
gcloud iam service-accounts keys create "postgresql2dc-credentials.json" \
--iam-account "postgresql2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com"
```
Next, add Data Catalog admin role to the Service Account:
```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member "serviceAccount:postgresql2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com" \
--quiet \
--project $PROJECT_ID \
--role "roles/datacatalog.admin"
```
Click Check my progress to verify the objective.
![Screenshot 2025-06-21 5 31 41 PM](https://github.com/user-attachments/assets/8f5fa8dc-fe10-4ab7-89e8-922be7b20ea8)

Create a Service Account for PostgreSQL
![Screenshot 2025-06-21 5 32 47 PM](https://github.com/user-attachments/assets/cf3ed40b-2e7e-48c9-b95b-c8adca16c328)


Execute PostgreSQL to Dataplex Universal Catalog connector
You can build the PostgreSQL connector yourself by going to this GitHub repository.

To facilitate its usage, this lab uses a docker image.
![Screenshot 2025-06-21 5 33 31 PM](https://github.com/user-attachments/assets/822e3e8d-6510-45d5-ab6b-7fba5c472b83)

The variables needed were output by the Terraform config.

Change directories into the location of the Terraform scripts:
```bash
cd infrastructure/terraform/
```
Grab the environment variables:
```bash
public_ip_address=$(terraform output -raw public_ip_address)
username=$(terraform output -raw username)
password=$(terraform output -raw password)
database=$(terraform output -raw db_name)
```
Change back to the root directory for the example code:
```bash
cd ~/cloudsql-postgresql-tooling
```
![Screenshot 2025-06-21 5 34 39 PM](https://github.com/user-attachments/assets/52be0526-6730-4008-ab16-e6efc5782105)

Execute the connector:
```bash
docker run --rm --tty -v \
"$PWD":/data mesmacosta/postgresql2datacatalog:stable \
--datacatalog-project-id=$PROJECT_ID \
--datacatalog-location-id=$REGION \
--postgresql-host=$public_ip_address \
--postgresql-user=$username \
--postgresql-pass=$password \
--postgresql-database=$database
```
Soon after you should receive the following output:

![Screenshot 2025-06-21 5 35 57 PM](https://github.com/user-attachments/assets/e15f8ee4-c45a-47cd-b1c6-4aa7c68317d4)

Click Check my progress to verify the objective.

Execute PostgreSQL to Data Catalog connector

![Screenshot 2025-06-21 5 36 34 PM](https://github.com/user-attachments/assets/e00c1a5a-d816-40d9-9a00-0cc83ba45688)

Check the results of the script
Navigate to Dataplex Universal Catalog in the Google Cloud console by clicking on the Navigation menu (Navigation menu icon) > View all products > Analytics > Dataplex Universal Catalog.

Click on Catalog then Aspect types & Tag Templates.

You should see the following postgresql Tag Templates:

Postgresql Table - Metadata and Postgresql Schema - Metadata

![Screenshot 2025-06-21 5 38 10 PM](https://github.com/user-attachments/assets/0a0fc72b-3c21-4be5-b21c-c66f7eee7e27)

Click on Entry groups.
You should see the following postgresql Entry Group:

postgresql

Now click on the postgresql Entry Group. Your console should resemble the following:
postgresql Entry Group entries

This is the real value of an Entry Group — you can see all entries that belong to postgresql using the UI.

Click on one of the warehouse entries. Look at the Custom entry details and tags:
Custom entry details tabbed page

This is the real value the connector adds—it allows you to have the metadata searchable in Dataplex Universal Catalog.

Clean up
To delete the created resources, run the following command to delete the PostgreSQL metadata:
```bash
./cleanup-db.sh
```
Now execute the cleaner container:
```bash
docker run --rm --tty -v \
"$PWD":/data mesmacosta/postgresql-datacatalog-cleaner:stable \
--datacatalog-project-ids=$PROJECT_ID \
--rdbms-type=postgresql \
--table-container-type=schema
Copied!
Finally, delete the PostgreSQL database:
./delete-db.sh
```
From the Dataplex Universal Catalog menu, under Discover, click on the Search page.

![Screenshot 2025-06-21 5 41 17 PM](https://github.com/user-attachments/assets/91098906-08de-45fc-939a-d9da20590557)

In the search bar, enter PostgreSQL and click Search.

You no longer see the PostgreSQL Tag Templates in the results:

Search results: 0 rows to display

Ensure you see the following output in Cloud Shell before you move on:

  Cloud SQL Instance deleted
  
Next, you learn how to do the same thing with a MySQL instance.

## **Task 3. Connect MySQL to Dataplex Universal Catalog**
Create the MySQL database
Run the following command in Cloud Shell to return to your home directory:
```bash
cd
```
Run the following command to download the scripts to create and populate your MySQL instance:
```bash
gsutil cp gs://spls/gsp814/cloudsql-mysql-tooling.zip .
unzip cloudsql-mysql-tooling.zip
```
Now change your current working directory to the cloned repo directory:
```bash
cd cloudsql-mysql-tooling/infrastructure/terraform
```
Run the following commands to change the region and zone from us-central1 and us-central1-a to your default assigned region and zone:
```bash
export REGION=us-west1
sed -i "s/us-central1/$REGION/g" variables.tf
```
```bash
export ZONE=us-west1-a
sed -i "s/$REGION-a/$ZONE/g" variables.tf
```
Next execute the init-db.sh script:
```bash
cd ~/cloudsql-mysql-tooling
bash init-db.sh
```
This creates your MySQL instance and populate it with a random schema. After a few minutes, you should receive the following output:

Note: If you get an Error: Failed to load "tfplan" as a plan file, re-run the init-db script.
Click Check my progress to verify the objective.
Create the MySQL Database

Set up the Service Account
Run the following to create a Service Account:
```bash
gcloud iam service-accounts create mysql2dc-credentials \
--display-name  "Service Account for MySQL to Data Catalog connector" \
--project $PROJECT_ID
```
Next, create and download the Service Account Key:
```bash
gcloud iam service-accounts keys create "mysql2dc-credentials.json" \
--iam-account "mysql2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com"
```
Next add Data Catalog admin role to the Service Account:
```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member "serviceAccount:mysql2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com" \
--quiet \
--project $PROJECT_ID \
--role "roles/datacatalog.admin"
```
Click Check my progress to verify the objective.
Create a Service Account for MySQL

Execute MySQL to Dataplex Universal Catalog connector
You can build the MySQL connector yourself by going to this GitHub repository.

To facilitate its usage, this lab uses a docker image.

The variables needed were output by the Terraform config.

Change directories into the location of the Terraform scripts:
```bash
cd infrastructure/terraform/
```
Grab the environment variables:
```bash
public_ip_address=$(terraform output -raw public_ip_address)
username=$(terraform output -raw username)
password=$(terraform output -raw password)
database=$(terraform output -raw db_name)
```
Change back to the root directory for the example code:
```bash
cd ~/cloudsql-mysql-tooling
```
Execute the connector:
```bash
docker run --rm --tty -v \
"$PWD":/data mesmacosta/mysql2datacatalog:stable \
--datacatalog-project-id=$PROJECT_ID \
--datacatalog-location-id=$REGION \
--mysql-host=$public_ip_address \
--mysql-user=$username \
--mysql-pass=$password \
--mysql-database=$database
```
Soon after you should receive the following output:

Click Check my progress to verify the objective.
Execute MySQL to Data Catalog connector

Check the results of the script
Navigate to Dataplex Universal Catalog in the Google Cloud console by clicking on the Navigation menu (Navigation menu icon) > View all products > Analytics > Dataplex Universal Catalog.

Click on Catalog then Aspect types & Tag Templates.

You should see the following mysql Tag Templates:

Mysql Table - Metadata and Mysql Database - Metadata

Click on Entry groups.
You should see the following mysql Entry Group:

mysql

Now click on the mysql Entry Group. Your console should resemble the following:
mysql Entry Group entries

This is the real value of an Entry Group — you can see all entries that belong to MySQL using the UI.

Click on one of the warehouse entries. Look at the Custom entry details and tags.
This is the real value the connector adds — it allows you to have the metadata searchable in Dataplex Universal Catalog.

Clean up
To delete the created resources, run the following command to delete the MySQL metadata:
```bash
./cleanup-db.sh
```
Now execute the cleaner container:
```bash
docker run --rm --tty -v \
"$PWD":/data mesmacosta/mysql-datacatalog-cleaner:stable \
--datacatalog-project-ids=$PROJECT_ID \
--rdbms-type=mysql \
--table-container-type=database
```
Finally, delete the PostgreSQL database:
```bash
./delete-db.sh
```
Ensure you see the following output in Cloud Shell before you move on:

  Cloud SQL Instance deleted
 
From the Dataplex Universal Catalog menu, under Discover, click on the Search page.

In the search bar, enter MySQL and click Search.

You no longer see the MySQL Tag Templates in the results.

![Screenshot 2025-06-21 5 53 41 PM](https://github.com/user-attachments/assets/bb828a5c-f579-44d0-b421-ced824117949)
