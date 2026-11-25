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

<img width="1562" height="557" alt="Screenshot 2026-11-24 165607" src="https://github.com/user-attachments/assets/19261388-a8df-4287-804f-5222685b4eb4" />


#### Internet Gateway and NAT Gateway Connections

<img width="1566" height="547" alt="Screenshot 2026-11-24 165620" src="https://github.com/user-attachments/assets/e7763a97-1fa7-4fd1-8e36-f0ac8a2d8078" />

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

 

The `student_id` column is the primary key and is configured with
auto-increment.

#### Database Table Structure


<img width="731" height="226" alt="Screenshot 2026-11-25 102621" src="https://github.com/user-attachments/assets/1bffe91c-b351-4ed5-9dc1-4d85a3fc7ae6" />

#### Student Records in MariaDB

<img width="1185" height="147" alt="Screenshot 2026-11-25 102650" src="https://github.com/user-attachments/assets/24f8cbc7-c227-4d8c-81e3-bd08fb53c698" />


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

<img width="622" height="350" alt="Screenshot 2026-11-25 202109" src="https://github.com/user-attachments/assets/4686fb8f-44ca-414e-b83e-f5e837e55992" />


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

<img width="1917" height="1020" alt="Screenshot 2026-11-25 102124" src="https://github.com/user-attachments/assets/6ff9b019-aeb1-4d1d-a46b-a4440d1233b9" />


#### Students List

<img width="1917" height="1020" alt="Screenshot 2026-11-25 102107" src="https://github.com/user-attachments/assets/c2dde6f8-d97a-40e8-8e69-5b8bf65b9465" />



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


