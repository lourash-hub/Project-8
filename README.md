# Project-8
Darey.io_Project-8

## Awesome Documentation of Project-8

In this project we will enhance our Tooling Website solution by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.

#### Task
Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 intance. Make sure that users can be served by Web servers through the Load Balancer.

To simplify, let us implement this solution with 2 Web Servers, the approach will be the same for 3 and more Web Servers

#### Prerequisites
Make sure that you have following servers installed and configured within Project-7:

Two RHEL8 Web Servers
One MySQL DB Server (based on Ubuntu 20.04)
One RHEL8 NFS server

![Architecture](./Images/Architecture.png)

#### CONFIGURE APACHE AS A LOAD BALANCER
##### Configure Apache As A Load Balancer
1.  Create an Ubuntu Server 20.04 EC2 instance and name it LB.
2.  Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.
3.  Install Apache Load Balancer on LB server and configure it to point traffic coming to LB to both Web Servers:

```markdown
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
```

Make sure apache2 is up and running
```sudo systemctl status apache2```

Configure load balancing

```markdown 
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

#Restart apache server

sudo systemctl restart apache2
```

bytraffic balancing method will distribute incoming load between the Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by loadfactor parameter.

4.  Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from the browser:

![Testing](./Images/testing.png)


Open two ssh/Putty consoles for both Web Servers and run following command:
```sudo tail -f /var/log/httpd/access_log```

Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them.

If you have configured everything correctly – your users will not even notice that their requests are served by more than one server.

![Testing_1](./Images/Weserver-1.png)

![Testing_2](./Images/Weserver-2.png)

Optional Step – Configure Local DNS Names Resolution
Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management.
What we can do, is to configure local domain name resolution. The easiest way is to use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well. So let us configure IP address to domain name mapping for our LB.

```markdown
#Open this file on your LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2```

Now you can update your LB config file with those names instead of IP addresses.

```markdown
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```

![Final Architecture](./Images/Final%20Architecture.png)
