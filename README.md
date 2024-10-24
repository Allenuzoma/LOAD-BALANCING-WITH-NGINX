# LOAD-BALANCING-WITH-NGINX

![image](https://github.com/user-attachments/assets/d5c0b8ee-4983-480d-a17e-6d588ab6a8c7)

Load Balancer Solution With Nginx and SSL/TLS


Part 1 - Configure Nginx As A Load Balancer

We will create a new load balancer

1. We will create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx-LB.

2. Ensure that the following ports are open by creating incoming rules for them in our security group: Port 80 for http connections and Port 443 for https connections to the 




3. Next, we have to update the /etc/hosts file. This file is used for the local DNS for storing the Web Servers' names(e.g. Web1 and Web2) and their local IP addresses.

  sudo nano etc/hosts

 

 ![etc hosts file](https://github.com/user-attachments/assets/e66258f7-d114-4c0b-946e-9cddb9411cc8)


3. Next we will Install and configure Nginx as a load balancer to point the traffic to the resolvable DNS names of the webservers.
 
 Update the instance and Install Nginx: 

    #update the apt repository
    sudo apt update
    #install nginx
    sudo apt install nginx

4. Verify that the nginx is running on the cli using the command:

    sudo systemctl status nginx

   ![nginx running on cli](https://github.com/user-attachments/assets/fd0ddad2-4592-4e1c-80fd-be35505f2aa6)

5. Confirm that nginx is running on the browser by using the IP address of the load balancer

![verify nginx browser](https://github.com/user-attachments/assets/891972b3-a870-4951-bb2b-ed20cf958933)

        

4. Open the default nginx configuration file


   sudo nano /etc/nginx/nginx.conf





    #insert following configuration into http section
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
#comment out this line
# include /etc/nginx/sites-enabled/*;

Restart Nginx and make sure the service is up and running:


(e.g. Web1 and Web2) and their local IP addresses

sudo vi /etc/nginx/nginx.conf
#insert following configuration into http section
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
#comment out this line
# include /etc/nginx/sites-enabled/*;


Restart Nginx and make sure the service is up and running
sudo systemctl restart nginx
sudo systemctl status nginx


    sudo systemctl restart nginx
    sudo systemctl status nginx



Part 2 - Register a new domain name and confi gure secured
connection using SSL/TLS certifi cates
Let us make necessary configurations to make connections to our Tooling
Web Solution secured!
In order to get a valid SSL certificate - you need to register a new domain
name, you can do it using any Domain name registrar - a company that
manages reservation of domain names. The most popular ones
are: Godaddy.com, Domain.com, Bluehost.com.
1. Register a new domain name with any registrar of your choice in any
domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)
2. Assign an Elastic IP to your Nginx LB server and associate your
domain name with this Elastic IP
48% COMPLETE
 Previous Lesson
You might have noticed, that every time you restart or stop/start your EC2
instance - you get a new public IP address. When you want to associate
your domain name - it is better to have a static IP address that does not
change after reboot. Elastic IP is the solution for this problem, learn how to
allocate an Elastic IP and associate it with an EC2 server on this page.
3. Update A record in your registrar to point to Nginx LB using Elastic IP
address
Learn how associate your domain name to your Elastic IP on this page.
Side Self Study: Read about different DNS record types and learn what
they are used for.
Check that your Web Servers can be reached from your browser using new
domain name using HTTP protocol - http://<your-domain-name.com>
4. Configure Nginx to recognize your new domain name
Update your nginx.conf with server_name www.<your-domain-name.com> instead
of server_name www.domain.com
5. Install certbot and request for an SSL/TLS certificate
Make sure snapd service is active and running
sudo systemctl status snapd
Install certbot
sudo snap install --classic certbot
Request your certificate (just follow the certbot instructions - you will need
to choose which domain you want your certificate to be issued for, domain
name will be looked up from nginx.conf file so make sure you have updated
it on step 4).
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
Test secured access to your Web Solution by trying to reach https://<your￾domain-name.com>
You shall be able to access your website by using HTTPS protocol (that
uses TCP port 443) and see a padlock pictogram in your browser's search
string. Click on the padlock icon and you can see the details of the
certificate issued for your website.
6. Set up periodical renewal of your SSL/TLS certificate
By default, LetsEncrypt certificate is valid for 90 days, so it is
recommended to renew it at least every 60 days or more frequently.
You can test renewal command in dry-run mode
sudo certbot renew --dry-run
Best pracice is to have a scheduled job that to run renew command
periodically. Let us configure a cronjob to run the command twice a day.
To do so, lets edit the crontab file with the following command:
crontab -e
Add following line:
* */12 * * * root /usr/bin/certbot renew > /dev/null 2>&1
You can always change the interval of this cronjob if twice a day is too
often by adjusting schedule expression.
Side Self Study: Refresh your cron configuration knowledge by
watching this video.
You can also use this handy online cron expression editor.
Congratulations!
You have just implemented an Nginx Load Balancing Web Solution with
secured HTTPS connection with periodically updated SSL/TLS certificates.**

