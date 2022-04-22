# PROJECT 5: CLIENT/SERVER ARCHITECTURE USING A MYSQL RELATIONAL DATABASE MANAGEMENT SYSTEM
## IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS).
TASK – Implement a Client Server Architecture using MySQL Database Management System (DBMS).

To demonstrate a basic client-server using MySQL Relational Database Management System (RDBMS), follow the below instructions
## STEP 1 - Create and configure two Linux-based virtual servers (EC2 instances in AWS).
```javascript
Server A name - `mysql server`
Server B name - `mysql client`
```
### Screenshot showing configured EC2 Instances running
![Screenshot (77)](https://user-images.githubusercontent.com/52970510/164568745-da2bc3a3-1334-41fe-a85a-d4e6a9a2d25e.png)

## STEP 2 - On mysql server Linux Server install MySQL Server software.
### Screenshot showing installation of mysql_server software
![Screenshot (79)](https://user-images.githubusercontent.com/52970510/164569082-3570928e-a869-45d1-9fdc-af4a4768579d.png)

## STEP 3 - On mysql client Linux Server install MySQL Client software.
### Screenshot showing installation of mysql_client software
![Screenshot (80)](https://user-images.githubusercontent.com/52970510/164569224-98bcb8ce-770f-4c63-b568-27b98d5dae12.png)

## STEP 4 - By default, both of your EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups. For extra security, do not allow all IP addresses to reach your ‘mysql server’ – allow access only to the specific local IP address of your ‘mysql client’.
### Screenshot showing TCP port 3306 open in the Security group for MyQL server connection 
![Screenshot (81)](https://user-images.githubusercontent.com/52970510/164569367-a4556344-8436-4755-bb88-b0283976fb12.png)

## STEP 5 - You might need to configure MySQL server to allow connections from remote hosts.
```javascript
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
Replace ‘127.0.0.1’ to ‘0.0.0.0’ like this:

### Screenshot showing bind address replacement in MySQL database server configuration file
![Screenshot (82)](https://user-images.githubusercontent.com/52970510/164569820-8cae3557-becb-4ff9-a08d-b718a18dfd1b.png)

### Database creation
![Screenshot (83)](https://user-images.githubusercontent.com/52970510/164570070-df85eed2-cfdc-445c-8854-1b29c785925e.png)

## STEP 6 - From mysql client Linux Server connect remotely to mysql server Database Engine without using SSH. You must use the mysql utility to perform this action.
### Screenshot showing client server connection to mysql server database 
![Screenshot (84)](https://user-images.githubusercontent.com/52970510/164570288-b369deda-e0ef-4b95-9fad-add73af86f15.png)

## STEP 7 - Check that you have successfully connected to a remote MySQL server and can perform SQL queries:
```javascript
Show databases;
```
### Screenshot showing databases
![Screenshot (85)](https://user-images.githubusercontent.com/52970510/164570461-bb479fa1-d602-483e-b9b7-7c0ad1fefebd.png)
