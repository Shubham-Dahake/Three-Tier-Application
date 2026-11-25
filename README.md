# Three-Tier Web Application Deployment on AWS

## Project Overview

This project demonstrates the deployment of a **Three-Tier Web
Application on AWS** using a custom VPC, public and private subnets, EC2
instances, Nginx Reverse Proxy, Apache Tomcat 9, Java, and
MariaDB/Amazon RDS.

The Proxy Server receives browser requests, the Application Server runs
the Java web application, and the database layer stores student records.

The project covers AWS networking, private server access, NAT Gateway,
Internet Gateway, Elastic IP, reverse proxy configuration, application
deployment, and database connectivity.

## Architecture

> **Architecture diagram:** Add your architecture diagram to the
> `screenshots` folder and name it `architecture.png`. The image will
> then appear below.

![Three-Tier AWS Architecture](screenshots/architecture.png)

### Three-Tier Components

  -----------------------------------------------------------------------
  Tier              Server            Technology        Responsibility
  ----------------- ----------------- ----------------- -----------------
  Presentation      Proxy Server      Nginx             Receives HTTP
                                                        requests and
                                                        forwards them

  Application       App Server        Java, Tomcat 9    Runs and
                                                        processes
                                                        application
                                                        requests

  Database          Database Layer    MariaDB / Amazon  Stores student
                                      RDS               records
  -----------------------------------------------------------------------

## Technologies Used

-   **Cloud Platform:** Amazon Web Services (AWS)
-   **Networking:** VPC, Subnets, Route Tables
-   **Internet Connectivity:** Internet Gateway, NAT Gateway
-   **Public IP:** Elastic IP
-   **Compute:** Amazon EC2
-   **Web Server:** Nginx
-   **Application Server:** Apache Tomcat 9
-   **Application:** Java-based Student Registration Application
-   **Database:** MariaDB / Amazon RDS
-   **Database Connectivity:** JDBC, MySQL Connector
-   **Operating System:** Linux
-   **Deployment:** WAR File
-   **Remote Access:** SSH

------------------------------------------------------------------------

## AWS Infrastructure Setup

### 1. Create a Custom VPC

Created a custom VPC named `three-tier-vpc` with the following
configuration:

-   **IPv4 CIDR:** `10.0.0.0/16`
-   **IPv6:** Not enabled

The VPC provides an isolated networking environment for the project
resources.

### 2. Create Subnets

Created three subnets across three Availability Zones.

  Subnet Name          Type      Availability Zone   CIDR Block
  -------------------- --------- ------------------- ----------------
  `public-subnet`      Public    `us-east-1a`        `10.0.0.0/20`
  `private-subnet-1`   Private   `us-east-1b`        `10.0.16.0/20`
  `private-subnet-2`   Private   `us-east-1c`        `10.0.32.0/20`

**Purpose:**

-   Public Subnet: Hosts the Nginx Proxy Server.
-   Private Subnet 1: Hosts the Application Server.
-   Private Subnet 2: Hosts the DB Server.

### 3. Internet Gateway

Created an Internet Gateway and attached it to the custom VPC.

The Internet Gateway provides internet connectivity to resources in the
Public Subnet when public IP addressing and routing are configured
correctly.

### 4. NAT Gateway and Elastic IP

Created a NAT Gateway inside the Public Subnet and associated an Elastic
IP with it.

The NAT Gateway allows instances in the Private Subnets to initiate
outbound internet connections without requiring public IP addresses.

### 5. Route Tables

Created two route tables.

**Public Route Table (`public-rt`)**

-   Associated with the Public Subnet.
-   Contains the local VPC route.
-   Routes `0.0.0.0/0` to the Internet Gateway.

**Private Route Table (`private-rt`)**

-   Associated with both Private Subnets.
-   Contains the local VPC route.
-   Routes `0.0.0.0/0` to the NAT Gateway.

#### VPC and Subnet Resource Map

![VPC, subnets and route
tables](screenshots/01-vpc-subnets-route-tables.png)

#### Internet Gateway and NAT Gateway Connections

![Internet Gateway and NAT
Gateway](screenshots/02-internet-nat-gateway.png)

### 6. EC2 Instance Deployment

Launched three EC2 instances inside the custom VPC.

  EC2 Instance   Subnet             Purpose
  -------------- ------------------ -----------------------
  Proxy Server   Public Subnet      Nginx Reverse Proxy
  App Server     Private Subnet 1   Java + Apache Tomcat
  DB Server      Private Subnet 2   Database server setup

The Proxy Server is publicly accessible. The private instances are
accessed through the Proxy Server using SSH jump/bastion access.

### 7. Security Groups

Configured Security Groups to control communication between the servers.

  Port   Protocol   Service           Purpose
  ------ ---------- ----------------- ----------------------------------
  22     TCP        SSH               Remote server access
  80     TCP        HTTP              Web application access
  3306   TCP        MySQL / MariaDB   Database connectivity
  8080   TCP        Tomcat            Application server communication

Rules should be restricted to trusted IP addresses and the appropriate
source Security Groups. Avoid exposing SSH, Tomcat, or database ports to
the entire internet.

------------------------------------------------------------------------

## Server Configuration

### 8. Connect to EC2 Instances

Connected to the Proxy Server using SSH.

From the Proxy Server, accessed the private App Server and DB Server
using SSH jump/bastion access. This allowed administration of private
instances without assigning them public IP addresses.

### 9. Proxy Server Setup -- Nginx

Installed Nginx on the Proxy Server and configured it as a Reverse
Proxy.

Nginx receives incoming HTTP requests and forwards them to the private
Application Server.

**Example Nginx configuration:**

``` nginx
location / {
    proxy_pass http://APP_SERVER_PRIVATE_IP:8080/student/;
}
```

Replace `APP_SERVER_PRIVATE_IP` with the actual private IP address of
the App Server. Use the location and application context path configured
for your deployment.

**Restart Nginx:**

``` bash
sudo systemctl restart nginx
```

### 10. Application Server Setup -- Java and Tomcat

Connected to the private App Server through the Proxy Server.

Installed Java and configured Apache Tomcat 9.

**Installed components:**

-   Java
-   Apache Tomcat 9

Tomcat runs the Java-based Student Registration Application.

### 11. Download Student WAR File

Downloaded the Student Registration Application WAR file from GitHub.

**Command template:**

``` bash
curl -L -O <GITHUB_WAR_FILE_URL>
```

Copied `student.war` into Tomcat's `webapps` directory. Tomcat deploys
the application when it starts.

### 12. Download MySQL Connector

Downloaded the MySQL Connector JAR file from GitHub.

**Command template:**

``` bash
sudo wget -O mysql-connector.jar <GITHUB_CONNECTOR_URL>
```

Copied the connector JAR file into Tomcat's `lib` directory. The JDBC
driver allows the Java application to connect to a MariaDB/MySQL
database.

### 13. Tomcat Database Configuration

Configured Tomcat's database connection by editing:

``` bash
conf/context.xml
```

Configured the JDBC Resource with the database name, endpoint, username,
password, and JDBC driver according to the application requirements.

**Security note:** Do not commit database passwords, private keys, or
other credentials to GitHub. Use placeholders in public documentation.

------------------------------------------------------------------------

## Database Configuration

### 14. Amazon RDS Setup

Created an Amazon RDS database using the MariaDB engine.

Configured:

-   MariaDB engine
-   Database instance
-   Database username and password
-   Database endpoint
-   Database connectivity

Configured connectivity between the Application Server and the RDS
database. The App Server uses the database endpoint for database
communication.

### 15. MariaDB Database and Table

Connected to MariaDB and used the `studentapp` database.

**Commands used:**

``` sql
SHOW DATABASES;
USE studentapp;
SHOW TABLES;
DESC students;
SELECT * FROM students;
```

#### Students Table Structure

  Column                  Description
  ----------------------- -----------------------
  `student_id`            Student ID
  `student_name`          Student Name
  `student_addr`          Student Address
  `student_age`           Student Age
  `student_qual`          Student Qualification
  `student_percent`       Student Percentage
  `student_year_passed`   Student Year Passed

The `student_id` column is the primary key and is configured with
auto-increment.

#### Database Table Structure

![MariaDB students table
structure](screenshots/03-mariadb-students-table-structure.png)

#### Student Records in MariaDB

![MariaDB student records](screenshots/04-mariadb-student-records.png)

------------------------------------------------------------------------

## Application Deployment and Testing

### 16. Start Apache Tomcat

Connected to the Application Server and navigated to Tomcat's `bin`
directory.

**Command:**

``` bash
./catalina.sh start
```

Started Apache Tomcat to deploy and run the Student Registration
Application.

#### Tomcat Server Screenshot

![Apache Tomcat server](screenshots/05-apache-tomcat.png)

### 17. Restart Nginx

After configuring the Reverse Proxy, restarted Nginx on the Proxy
Server.

**Command:**

``` bash
sudo systemctl restart nginx
```

Nginx forwards browser requests to the private Application Server on
port 8080.

### 18. Application Testing

Opened the Public IP address of the Proxy Server in a web browser.

Successfully accessed the Student Registration Application through the
Nginx Reverse Proxy.

**Application features:**

-   Student Registration
-   View Students List
-   Edit Student Details
-   Delete Student Details

#### Student Registration Form

![Student Registration
Form](screenshots/06-student-registration-form.png)

#### Students List

![Students List](screenshots/07-students-list.png)

------------------------------------------------------------------------

## Key Learning Outcomes

Through this project, I gained practical experience in:

-   Creating and configuring a custom AWS VPC.
-   Working with Public and Private Subnets.
-   Configuring Internet Gateway and NAT Gateway.
-   Associating an Elastic IP with a NAT Gateway.
-   Configuring Public and Private Route Tables.
-   Launching and managing multiple EC2 instances.
-   Configuring Security Groups for server communication.
-   Connecting to private EC2 instances using SSH jump access.
-   Installing and configuring Nginx as a Reverse Proxy.
-   Installing Java and Apache Tomcat 9.
-   Deploying Java applications using WAR files.
-   Configuring JDBC database connectivity.
-   Setting up MariaDB on Amazon RDS.
-   Testing a Three-Tier Web Application through a browser.

## Conclusion

This project provided hands-on experience deploying a Three-Tier Web
Application using AWS infrastructure. By combining VPC networking, EC2
instances, Nginx, Apache Tomcat, Java, and MariaDB/RDS, I practised
cloud networking, Linux server administration, application deployment,
and database connectivity.

**This project represents my practical learning journey in AWS and Cloud
Computing.**

------------------------------------------------------------------------

## Repository Screenshot Guide

All screenshot files are stored in the `screenshots` folder. If you
replace an image, keep the same filename so the README image link
continues to work.

  File                                        Screenshot
  ------------------------------------------- -----------------------------------------------
  `01-vpc-subnets-route-tables.png`           VPC, subnets and route tables
  `02-internet-nat-gateway.png`               Internet Gateway and NAT Gateway
  `03-mariadb-students-table-structure.png`   MariaDB table structure
  `04-mariadb-student-records.png`            Student records in MariaDB
  `05-apache-tomcat.png`                      Tomcat page
  `06-student-registration-form.png`          Registration form
  `07-students-list.png`                      Students list
  `architecture.png`                          Add your three-tier architecture diagram here
