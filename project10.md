# LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS
## CONFIGURE NGINX AS A LOAD BALANCER
You can either uninstall Apache from the existing Load Balancer server, or create a fresh installation of Linux for Nginx.
1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)
2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
Update the instance and Install Nginx
```javascript
sudo apt update
sudo apt install nginx
```
Configure Nginx LB using Web Servers’ names defined in /etc/hosts

Open the default nginx configuration file
```javascript
sudo vi /etc/nginx/nginx.conf
```
insert following configuration into http section
```javascript
upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }
```
comment out this line
```javascript
#include /etc/nginx/sites-enabled/*;
```
### Updating the Nginx configuration file
![2](https://user-images.githubusercontent.com/52970510/166324254-e3b38c87-532c-4491-a9e7-5ce2d205add0.jpg)

Restart Nginx and make sure the service is up and running
```javascript
sudo systemctl restart nginx
sudo systemctl status nginx
```
### Screenshot showing nginx restarted and then status check
![1](https://user-images.githubusercontent.com/52970510/166321423-9efcd60e-a515-4837-b418-dbb6127a1027.jpg)

## REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES
1. Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)
2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP
### Screesnhot showing Elastic IP assigned to Nginx LB server
![3](https://user-images.githubusercontent.com/52970510/166322574-d014e7ac-9801-49d7-840e-e837f18a757f.jpg)

3. Update A record in your registrar to point to Nginx LB using Elastic IP address
Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – http://<your-domain-name.com>
4. Configure Nginx to recognize your new domain name
Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com
5. Install certbot and request for an SSL/TLS certificate
Make sure snapd service is active and running
```javascript
sudo systemctl status snapd
```
Install certbot
```javascript
sudo snap install --classic certbot
```
Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step 4).
```javascript
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```
Test secured access to your Web Solution by trying to reach 
```javascript
https://<your-domain-name.com>
```
### Accessing your websolution
![4](https://user-images.githubusercontent.com/52970510/166323307-54d0aa0d-1c2c-4925-b542-a0f64017e8a8.jpg)

6. Set up periodical renewal of your SSL/TLS certificate
By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in dry-run mode
```javascript
sudo certbot renew --dry-run
```
Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the crontab file with the following command:
```javascript
crontab -e
```
Add following line:
```javascript
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```
You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.
