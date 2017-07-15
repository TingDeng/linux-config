Project Description:

This is about how to use  Amazon Lightsail as Linux server to host an application, 

In this case, the application is called "itemcatalog_app". 

The IP address, SSH port and url info:

IP address:54.236.xx.xxx

SSH port:2200

url:http://54.236.xx.xxx

SSH key for user grader: xxxxxx

log in from terminal:

ssh -i ~/.ssh/your_key_name grader@54.236.xx.xxx -p 2200

Configuration & Softwares:

1. Log in Amazon Lightsail and create a instance

2. Add new user and give grader the permission to sudo

     a. In termial, create new user grader use: sudo adduser grader
     
	 b. sudo touch /etc/sudoers.d/grader
	 
	 c. sudo nano /etc/sudoers.d/grader, add line: grader=ALL=NOPASSWD: ALL, save
	 
	 d. .ssh-keygen on local machine to create private key 
	 
	 e. su - grader-->mkdir .ssh-->touch .ssh/authorized_keys-->sudo nano .ssh/authorized_keys to add key info that is just generated
	 
	 f. make user authorized to key related files by sudo chown command
	 
	 g. chmod 700 .ssh & chmod 644 .ssh/authorized_keys for grader
	 
3. change SSH port and set up firewall

    a. sudo nano /etc/ssh/sshd_config and then change Port 22 to Port 2200 
    
	b.sudo ufw default deny incoming
	
          sudo ufw default allow outgoing
      
	  sudo ufw allow 2200/tcp
	  
	  sudo ufw allow www
	  
	  sudo ufw allow ntp
	  
          sudo ufw enable
	  
4. On Amazon Lightsail, in networking tag creat a custom port 2200 and click allow

5. Set timezone: sudo timedatectl set-timezone UTC

6. Install the below software

    a. Install Apache:sudo apt-get install apache2
    
    b. Install the libapache2-mod-wsgi package:sudo apt-get install libapache2-mod-wsgi
	
    c.sudo apt-get install postgresql postgresql-contrib
	d.Others:
	
      sudo apt-get install python-psycopg2 python-flask
      
      sudo apt-get install python-sqlalchemy python-pip
      
      sudo pip install oauth2client
      
      sudo pip install requests
      
      sudo pip install httplib2
      
      sudo pip install flask-seasurf
      
      sudo apt-get install git
      
7. Set up postgresql with creating new user called catalog: sudo -u postgres createuser -P catalog

8. Create an database called catalog with: sudo -u postgres createdb -O catalog catalog

9. Create folders for application files 

    a. In /var/www/, create folder called catalog and inside the folder create another folder that stores
      the itemcatalog_app application files from the github by git clone command
      
    b. In /var/www/catalog, create catalog.wsgi and add the following lines:
    
       #!/usr/bin/python
       
       import sys
       
       import logging
       
       logging.basicConfig(stream=sys.stderr)
       
       sys.path.insert(0,"/var/www/catalog/")
       
       from catalog import app as application
       
       application.secret_key = 'Add your secret key'
       
10. Connect postgresql database:

    a. "cd /"in its directory and edit db_setup.py: sudo nano database_setup.py
    
       change line: engine = create_engine('postgresql://catalog:your_database_password@localhost/catalog')
       
    b. Also change it in itemcatalogapp.py and db_init.py
    
    c. change file name to __init__.py: sudo mv itemcatalogapp.py  __init__.py
    
    
11. Configure virtual host

    sudo nano /etc/apache2/sites-available/FlaskApp.conf
    
	Edit the following lines:
	
	<VirtualHost *:xx>
	
        ServerName 54.236.xx.xxx
	
        ServerAdmin admin@mywebsite.com
	
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	
        <Directory /var/www/catalog/catalog/>
	
            Order allow,deny
	    
            Allow from all
	    
        </Directory>
	
        Alias /static /var/www/catalog/catalog/static
	
        <Directory /var/www/catalog/catalog/static/>
	
            Order allow,deny
	    
            Allow from all
	    
        </Directory>
	
        Alias /templates /var/www/catalog/catalog/templates
		
        <Directory /var/www/catalog/catalog/templates/>
	
            Order allow,deny
	    
            Allow from all
	    
        </Directory>
	
        ErrorLog ${APACHE_LOG_DIR}/error.log
	
        LogLevel warn
	
        CustomLog ${APACHE_LOG_DIR}/access.log combined
	
    </VirtualHost>
    
	Enable virtual host using : sudo a2ensite FlaskApp
	
12. reset google and facebook log in:

    a. changes in json file, use absolute path:
	   example: open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']```
       and change other paths with the same issue in json file
       
    b. on developer pages for google and facebook:
    
	   change Valid OAuth redirect URIs
	   
    c. download json file again after changes and replace the old ones
    
13. Run the app:

    a. In the same directory : python db_setup.py, to set up the postgresql database 
    
    b. then type python db_init.py to populate data
    
    c. python __init__.py to start the application on the IP address
    
14. Make sure all packages are updated before install softwares and configuration:

    a.sudo apt-get update
    
    b.sudo apt-get upgrade
    
    
15. sudo service apache2 restart everytime you make configuration changes

Reference:

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

http://docs.sqlalchemy.org/en/latest/orm/cascades.html

Use stackoverflow answers and Udacity forum for multiple questions during configuration

	 

  


