# LOAD-BALANCING-WITH-NGINX


Nginx is a high-performance, open-source web server known for its efficiency, scalability, and flexibility. It's widely used in various web applications, including three-tier architectures.

In a three-tier web application, Nginx can play multiple crucial roles:

Reverse Proxy: Nginx can act as a reverse proxy, load balancing requests across multiple application servers. This improves performance, fault tolerance, and scalability.

Static Content Serving: Nginx can efficiently serve static content like images, CSS, and JavaScript files, offloading this task from the application servers.

Caching: Nginx can cache frequently accessed content, reducing the load on the application servers and improving response times.

Web Socket Support: Nginx supports web sockets, enabling real-time communication between clients and servers, which is essential for many modern applications.

![nginx diagram](https://github.com/user-attachments/assets/5a61aa42-eaab-4d39-9ccb-55bc8607f1e9)


Load Balancer Solution With Nginx and SSL/TLS


**Part 1: Configure Nginx As A Load Balancer**


1. We will create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx-LB.
   
   ![instance creation](https://github.com/user-attachments/assets/9fce0a9c-e71c-4f74-b396-eed01f895d0f)


2. Ensure that the following ports are open by creating incoming rules for them in our security group: Port 80 for http connections and Port 443 for https connections to the 




3. Next, we have to update the /etc/hosts file. This file is used for the local DNS for storing the Web Servers' names(e.g. Web1 and Web2) and their local IP addresses.

       sudo nano /etc/hosts

 

   ![etc hosts file](https://github.com/user-attachments/assets/e66258f7-d114-4c0b-946e-9cddb9411cc8)


4. Next we will install and configure Nginx as a load balancer to point the traffic to the resolvable DNS names of the webservers.
 
 Update the instance and install Nginx using the following command: 

    #update the apt repository
    sudo apt update
    #install nginx
    sudo apt install nginx

5. Verify that the nginx is running on the cli using the command:

       sudo systemctl status nginx

   ![nginx running on cli](https://github.com/user-attachments/assets/fd0ddad2-4592-4e1c-80fd-be35505f2aa6)

6. Confirm that nginx is running on the browser by using the IP address of the load balancer with format


       http://<public-ip-add of nginx-lb>
      

  ![verify nginx browser](https://github.com/user-attachments/assets/891972b3-a870-4951-bb2b-ed20cf958933)

 We can see the default page for nginx on the browser.      

7. Now that the nginx is up and running, we will create a configuration file specific to our project, this configuration file will listen on port 80 and will direct the nginx file server to the domain which will be created shortly.

  To create the configuration file, enter the following command:

       sudo nano /etc/nginx/conf.d/nginx-lb.conf

   Paste the following configuration data in the newly created file:


    
          
          # Define the group of application servers
          upstream app_servers {
              server web1 weight=10;
              server web2 weight=10;
              server web3 weight=5;
          }
          
          server {
              listen 80;
              #server_name example.com; # Replace this with the public IP or domain name of the server
              server_name cloudchief.com.ng;  
          
              location / {
                  proxy_pass http://app_servers;
                  proxy_set_header Host $host;
                  proxy_set_header X-Real-IP $remote_addr;
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                  proxy_set_header X-Forwarded-Proto $scheme;
              }
          }
    


8. The config file in the step above was created so as to not tamper with the original nginx configuration file nginx.conf which can be accessed with the command:


         sudo nano /etc/nginx/nginx.conf


   ![default nginx conf edited to remove the site enabled](https://github.com/user-attachments/assets/43d55c26-7124-4497-9e05-00e40b9aa391)

        
  From this nginx.conf file I edited the code to include the .conf file I   
  created specially for my project. I also commented out the nginx/sites- 
 enabled line so as not to have the nginx server reading from or serving 
 the default webpage from the http part of the code.

  I added the line for the config file to refer to the newly created config 
  file:

      include /etc/nginx/conf.d/*.conf;

  And commented out:

      include /etc/nginx/sites-enabled/*;
    
    




  Now restart Nginx and make sure the service is up and running:

      sudo systemctl restart nginx
      sudo systemctl status nginx



**Note:**Ensure that the web servers listed in the /etc/nginx/conf.d/nginx-lb.conf file matches those in the /etc/hosts, else restarting nginx returns an error message. 



**Part 2 - Register a new domain name and configure secured connection using SSL/TLS certificates**

Let us make necessary configurations to make connections to our Tooling
Web Solution secured!
In order to get a valid SSL certificate - you need to register a new domain
name, you can do it using any Domain name registrar - a company that
manages reservation of domain names. The most popular ones
are: Godaddy.com, Domain.com, Bluehost.com.
1. Register a new domain name with any registrar of your choice in any
domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other.)

In my case I registered a domain **cloudchief.com.ng** on registrar GO54.com

  ![domain details go54](https://github.com/user-attachments/assets/52b2c612-0711-4d82-b2c5-b3926188e9b0)


2. I assigned an Elastic IP to my Nginx LB server, this way my nginx server will have a constant IP address which will not change after the server is rebooted.

  ![new cli details showing elastic ip add](https://github.com/user-attachments/assets/af96c838-bec2-47d6-abe9-35896382ad81)


3. Next, I associated the domain name with this Elastic IP.
   
  ![associating elastic ip](https://github.com/user-attachments/assets/555a90ef-cfd6-489e-8fd1-527f28af230e)


4. Update A record in your registrar to point to Nginx LB using Elastic IP
address

  ![editing A record on domain](https://github.com/user-attachments/assets/daadab74-37db-47d2-a217-4445e718398c)

  ![updating A record on domain](https://github.com/user-attachments/assets/c812d1dc-a20b-4a22-8b5c-3fc02478479a)


Check that your Web Servers can be reached from your browser using new
domain name using HTTP protocol - http://<your-domain-name.com>

5. We have to install certbot and request for an SSL/TLS certificate
   
    #First make sure snapd service is active and running:

      sudo systemctl status snapd
   
    #Install certbot
    
      sudo snap install --classic certbot
    
6. Request your certificate (just follow the certbot instructions - you 
 will need to choose which domain you want your certificate to be issued 
 for, domain name will be looked up from the .conf file.


      sudo ln -s /snap/bin/certbot /usr/bin/certbot
    
      sudo certbot --nginx
    
7. Test secured access to the Web Solution by trying to reach       

       https://<your￾domain-name.com>
   
    In my case,

       https://cloudchief.com.ng
  

   ![site showing ](https://github.com/user-attachments/assets/562f5f32-00f5-4ed0-9265-c9956902da11)


   ![site showing 2 ](https://github.com/user-attachments/assets/ebf6ad99-976b-4526-9c95-1ce9c1960e04)

   ![site certificate details](https://github.com/user-attachments/assets/d73077c6-50c1-4778-bbcf-d1b3d9b540b7)


You shall be able to access your website by using HTTPS protocol (that
uses TCP port 443) and see a padlock pictogram in your browser's search
string. Click on the padlock icon and you can see the details of the
certificate issued for your website.

9. Set up periodical renewal of your SSL/TLS certificate
    
  By default, LetsEncrypt certificate is valid for 90 days, so it is
recommended to renew it at least every 60 days or more frequently.
You can test renewal command in dry-run mode

    sudo certbot renew --dry-run

  ![sudo certbot renew --dry-run](https://github.com/user-attachments/assets/c695aa3d-527f-4541-8faf-dacf9066a298)

  Best pracice is to have a scheduled job that to run renew command
periodically. Let us configure a cronjob to run the command twice a day.

10. To do so, lets edit the crontab file with the following command:
crontab -e

  Add following line:

    * */12 * * * root /usr/bin/certbot renew > /dev/null 2>&1

  ![crontab -e](https://github.com/user-attachments/assets/1d2e5bfb-9f4e-415f-9206-caba2fe74a68)

  
  You can always change the interval of this cronjob if twice a day is too
often by adjusting schedule expression.

You have just implemented an Nginx Load Balancing Web Solution with
secured HTTPS connection with periodically updated SSL/TLS certificates.**

