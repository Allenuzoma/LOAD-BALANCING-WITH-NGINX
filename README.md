# LOAD-BALANCING-WITH-NGINX

![image](https://github.com/user-attachments/assets/d5c0b8ee-4983-480d-a17e-6d588ab6a8c7)

Load Balancer
Solution With Nginx
and SSL/TLS


Part 1 - Configure Nginx As A Load Balancer

We will create a new load balancer

1. We will create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx
LB (do not forget to open TCP port 80 for HTTP connections, also open
TCP port 443 - this port is used for secured HTTPS connections)

3. Update /etc/hosts file for local DNS with Web Servers' names
(e.g. Web1 and Web2) and their local IP addresses

5. Install and configure Nginx as a load balancer to point traffic to the
 resolvable DNS names of the webservers
 Update the instance and Install Nginx Install Nginx

    sudo apt update
    sudo apt install nginx
Configure Nginx LB using Web Servers' names defined in /etc/hosts
Hint: Read this blog to read about /etc/host
Open the default nginx configuration file


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

Restart Nginx and make sure the service is up and running:


    sudo systemctl restart nginx
    sudo systemctl status nginx



