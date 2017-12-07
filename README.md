# Udacity Linux Server Config

This is a project assigned by Udacity in part of the Udacity Full Stack Nanodegree program.

## About

This project is prepare a Linux server using AWS Lightsail to host a web apps. The server needs to be: secured from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

The server ip address is http://54.200.192.158/


## Walkthrough Steps

1. Connect to the Server SSH

2. Upgrade/Update existing packages
  * Update packages `sudo apt-get update`
  * Upgrade packages `sudo apt-get upgrade`

3. Create user access and permission.
  * `sudo adduser grader`
  * `sudo nano /etc/sudoers.d/grader`
  * Insert `grader ALL=(ALL:ALL) ALL`

4. Create ssh Access
  * Generate SSH Key locally `ssh-keygen`    
  * `mkdir /home/grader/.ssh`
  * `nano ~/.ssh/authorized_keys`
  * paste the public key that was created locally.
  * Test ssh connection using the key ssh grader  `ssh -i udacity-grader-keys -p 2200 grader@54.200.192.158`

5. Secure the ports and connections
  * Run `sudo nano /etc/ssh/sshd_config`
  * Change port `22` to `2200`
  * Change `PermitRootLogin` to `no`
  * `sudo service sshd restart`

  * Add the Firewall for port 2200 on the AWS Control Panel   Networking tab  
  * Restricts ports for: SSH(2200), HTTP(80), and NTP(123)
    ```
    sudo ufw allow 2200/tcp
    sudo ufw allow 80/tcp
    sudo ufw allow 123/udp
    sudo ufw enable```

6. Set the local timezone `sudo dpkg-reconfigure tzdata`

7. Prepare Webserver to host the Apache and other dependencies to run the web app.
  * `sudo apt-get install apache2`
  * `sudo apt-get install libapache2-mod-wsgi`
  * `sudo apt-get install python-flask python-pip python-setuptools`
  * 'sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils requests'

8. Prepare Database(PostgreSQL) for serving datas to the Application.  
  * `sudo apt-get install postgresql`
  * `sudo su - postgres`
  * Run postgreSQL Shell `psql`
  * Run the Shell
    ```
    CREATE DATABASE catalog;
    CREATE USER local;
    ALTER ROLE local WITH PASSWORD 'password';
    GRANT ALL PRIVILEGES ON DATABASE catalog TO local;
    ```  
9. Install Web Catalog Web Apps from Gitub.
  * `sudo apt-get install git`
  * `sudo git clone https://github.com/lauwrentius/udacity-item-catalog-project.git /var/www/item-catalog`
  * Move catalog folder contents into item-catalog folder
  * Create Entry point for the application `nano /var/www/item-catalog/catalog-app.wsgi`
    ```
    #!/usr/bin/env python

    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/item-catalog/")

    from webserver import app as application
    application.secret_key = '<SECRETKEY>'
    ```
  * Edit `database_setup.py`, `webserver.py` to use postgresql
  `engine = create_engine('postgresql://local:password@localhost/catalog')`

10. Configure virtual host
  * sudo nano /etc/apache2/sites-available/catalog-app.conf
    ```
    <VirtualHost *:80>
      ServerAdmin webmaster@localhost
      DocumentRoot /var/www/html
      WSGIScriptAlias / /var/www/item-catalog/catalog-app.wsgi
      <Directory /var/www/item-catalog/>
        Order allow,deny
        Allow from all
      </Directory>
      Alias /static /var/www/item-catalog/static
      <Directory /var/www/item-catalog/static/>
        Order allow,deny
        Allow from all
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
  * sudo a2ensite catalog-app
  * sudo service apache2 reload
