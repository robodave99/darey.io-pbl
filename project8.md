# CONFIGURE APACHE AS A LOAD BALANCER
1. Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this:
2. Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.
3. Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:
```javascript
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev
```
```javascript
#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic
```
```javascript
#Restart apache2 service
sudo systemctl restart apache2
```
Make sure apache2 is up and running
```javascript
sudo systemctl status apache2
```
Configure load balancing
```javascript
sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
```
Restart apache server
```javascript
sudo systemctl restart apache2
```
### Screenshots
![1](https://user-images.githubusercontent.com/52970510/166144106-3fbbe327-ff8d-450b-a7df-d9cfa9f444d9.jpeg)

![2](https://user-images.githubusercontent.com/52970510/166144119-f4684c3a-405c-492f-8dbc-d47883be4b2d.jpeg)

bytraffic balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by loadfactor parameter.

You can also study and try other methods, like: bybusyness, byrequests, heartbeat

4. Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:
```javascript
http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php
```
Note: If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.

Open two ssh/Putty consoles for both Web Servers and run following command:
```javascript
sudo tail -f /var/log/httpd/access_log
```
Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them.

If you have configured everything correctly – your users will not even notice that their requests are served by more than one server.
  
### Optional Step – Configure Local DNS Names Resolution
Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management.
What we can do, is to configure local domain name resolution. The easiest way is to use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well. So let us configure IP address to domain name mapping for our LB.
```javascript
#Open this file on your LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```
Now you can update your LB config file with those names instead of IP addresses.
```javascript
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```
You can try to curl your Web Servers from LB locally curl http://Web1 or curl http://Web2 – it shall work.

Remember, this is only internal configuration and it is also local to your LB server, these names will neither be ‘resolvable’ from other servers internally nor from the Internet.
### Screenshots
![Screenshot (112)](https://user-images.githubusercontent.com/52970510/166144062-71ad5f77-daa5-4c05-9569-c9b4f975689e.png)

![Screenshot (113)](https://user-images.githubusercontent.com/52970510/166144080-793f754e-6be9-48c1-90d3-5dc8d16ea2ac.png)
