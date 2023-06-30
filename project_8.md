# LOAD BALANCER SOLUTION WITH APACHE

Scope: to set up and configure load balancer to distribute incoming traffic to the web servers


1. Spin up an Ubuntu Server 20.04 EC2 instance and name it **Project-8-apache-lb**


2. Open TCP port 80 Inbound rules in Security groups


3. Run the following commands to install Apache Load Balancer on Project-8-apache-lb, and direct traffic coming to it to the Web servers from project 7   

    ```
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

    #Confirm that apache2 is running
    sudo systemctl status apache2
    ```


4. Open apache2 configuration file and set it up to target the web servers

    ```
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

    #### OPTIONAL STEP 
    instead of the ip address you can use a domain name for your web servers, although this ste up only works locally from the configured server (i.e., LB server)

    ```
    #Open this file on your LB server

    sudo vi /etc/hosts

    #Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

    <WebServer1-Private-IP-Address> Web1
    <WebServer2-Private-IP-Address> Web2 
    ```

    The load balancer configuration file then becomes this

    ![config_file](./images/load_balancer_config_file.png)


5. In project 7 you mounted `/var/log/httpd` from both web servers to the NFS server's /mnt/logs - unmount them with `sudo umount -f /var/log/httpd`


6. Test your configuration - try to access your LB’s public IP address or Public DNS name from your browser

    `http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php`


7. Run `sudo tail -f /var/log/httpd/access_log
` for each web server to check each server's log file - if you refresh your browser on the LB's page you will see new records appearing in the log file
    
    ![log](./images/logs_for_web.png)