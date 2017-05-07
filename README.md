# Linux-Server-Configuration
Turning a brand-new, bare bones, Linux server into the secure and efficient web application host.
I host my previous project [Item-Catalog](https://github.com/Hemalatah/Item-Catalog) in this Linux Server instance.

Networking:
  - Public IP: 34.205.26.212
  - SSH Port: 22
  - Project URL: http://ec2-34-205-26-212.compute-1.amazonaws.com/

# Steps to complete this project:
 
  - Start a new Ubuntu Linux server instance on Amazon Lightsail.
    1. First, log in to [Lightsail](https://lightsail.aws.amazon.com/). If you don't already have an Amazon Web Services account, you'll be prompted to create one.
    2. Create an instance - Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.
    3. Choose an instance image: Ubuntu
Lightsail supports a lot of different instance types. An instance image is a particular software setup, including an operating system and optionally built-in applications.

        For this project, you'll want a plain Ubuntu Linux image. There are two settings to make here. First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu as the operating system.
        
    4. Choose your instance plan.
The instance plan controls how powerful of a server you get. It also controls how much money they want to charge you. For this project, the lowest tier of instance is just fine. And as long as you complete the project within a month and shut your instance down, the price will be zero.

    5. Give your instance a hostname.
Every instance needs a unique hostname. You can use any name you like, as long as it doesn't have spaces or unusual characters in it. Your instance's name will be visible to you and to the project reviewer.

    6. Wait for it to start up.
It may take a few minutes for your instance to start up.
_While your instance is starting up, Lightsail shows you a grayed-out display._
_Once your instance is running, the display gets brighter._

    7. It's running; let's use it!
        The public IP address of the instance is displayed along with its name. 
The big orange "Connect using SSH" button is the next step.
Explore the other tabs of this user interface to find the Lightsail firewall and other settings. You'll need to configure the Lightsail firewall as one step of the project.When you SSH in, you'll be logged as the ubuntu user. 
 - Download the default key-pair from the amazon lightsail and copy to /.ssh folder.
 - Now Use the command ssh -i ~/.ssh/key.pem ubuntu@34.205.26.212 -p 22 to create the instance on your terminal
 - Create a new user named grader
    1. sudo adduser grader
    2. optional: install finger to check user has been added apt-get install finger
    3. finger grader
 - Give the grader the permission to sudo
    1. sudo visudo
    2. inside the file add grader ALL=(ALL:ALL) ALL below the root user under "#User privilege specification"
    3. save file(nano: ctrl+x, Y, Enter)
	4. sudo touch /etc/sudoers.d/grader
    5. sudo nano /etc/sudoers.d/grader
    6. sudo touch /etc/sudoers.d/root
    7.sudo nano /etc/sudoers.d/root
    8. Add grader to /etc/suoders.d/ and type in grader ALL=(ALL:ALL) ALL
    9. Add root to /etc/suoders.d/ and type in root ALL=(ALL:ALL) ALL
 - Update all currently installed packages
    1. Find updates:sudo apt-get update
    2. Install updates:sudo sudo apt-get upgrade Hit Y for yes and give yourself a break while it installs.
 - Change the SSH port from 22 to 2200 and other SSH configuration required
   1. nano /etc/ssh/sshd_config add port 2200 below port 22
while in the file also change PermitRootLogin prohibit-password to PermitRootLogin no to disallow root login. Change PasswordAuthentication from no to yes. We will change back after finishing SHH login setup
   2. save file(nano: ctrl+x, Y, Enter)
   3. restart ssh servicesudo service ssh reload
 - Create SSH keys and copy to server manually
   1. On your local machine generate SSH key pair with: ssh-keygen
   2. save youkeygen file in your ssh directory /Users/your home directory/.ssh/id_rsa
You can add a password to use encase your keygen file gets compromised(you will be prompted to enter this password when you connect with key pair)
   3. Change the SSH port number configuration in Amazon lightsail in networking tab to 2200.
   4. login into grader account using password set during user creation ssh -v grader@*Public-IP-Address* -p 22
   5. Make .ssh directory mkdir .ssh
   6. make file to store key touch .ssh/authorized_keys
   7. On your local machine read contents of the public key cat .ssh/id_rsa.pub
   8. Copy the key and paste in the file you just created in grader nano .ssh/authorized_keys paste contents(ctr+v)
   9. save file(nano: ctrl+x, Y, Enter)
   10. Set permissions for files: chmod 700 .ssh chmod 644 .ssh/authorized_keys
   11. Change PasswordAuthentication from yes back to no. nano /etc/ssh/sshd_config
   12. save file(nano: ctrl+x, Y, Enter)
   13. login with key pair: ssh grader@34.205.26.212 -p 22 -i ~/.ssh/id_rsa
 - Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
   1. Check UFW status to make sure its inactivesudo ufw status
   2. Deny all incoming by defaultsudo ufw default deny incoming
   3. Allow outgoing by defaultsudo ufw default allow outgoing
   4. Allow SSH sudo ufw allow ssh
   5. Allow SSH on port 2200sudo ufw allow 2200/tcp
   6. Allow HTTP on port 80sudo ufw allow 80/tcp
   7. Allow NTP on port 123sudo ufw allow 123/udp
   8. Turn on firewallsudo ufw enable
 
 - Configure the local timezone to UTC
    1. run sudo dpkg-reconfigure tzdata from prompt: select none of the above. Then select UTC.
 - Install and configure Apache to serve a Python mod_wsgi application
   1. sudo apt-get install apache2 Check if "It works!" at you public IP address given during setup.
   2. install mod_wsgi: sudo apt-get install libapache2-mod-wsgi
configure Apache to handle requests using the WSGI module sudo nano /etc/apache2/sites-enabled/000-default.conf
    3. add WSGIScriptAlias / /var/www/html/myapp.wsgi before </VirtualHost> closing line
   4. save file(nano: ctrl+x, Y, Enter)
   5. Restart Apache sudo apache2ctl restart

 - install git
    1. sudo apt-get install git
 - install python dev and verify WSGI is enabled
    1. Install python-dev packagesudo apt-get install python-dev
    2. Verify wsgi is enabled sudo a2enmod wsgi
 - Create flask app
    1. cd /var/www
    2. sudo mkdir catalog
    3. cd catalog
    4. sudo mkdir catalog
    5. cd catalog
    6. sudo mkdir static templates
    7. sudo nano __init__.py
    ```sh
        from flask import Flask
        app = Flask(__name__)
        @app.route("/")
        def hello():
        	return "Hello, world (Testing!)"
        if __name__ == "__main__":
        	app.run()
    ```
 - install flask
    1. sudo apt-get install python-pip
    2. sudo pip install virtualenv
    3. sudo virtualenv venv
    4. sudo chmod -R 777 venv
    5. source venv/bin/activate
    6. pip install Flask
    7. python __init__.py
    8. deactivate
    9. Configure And Enable New Virtual Host
 - Create host config file sudo nano /etc/apache2/sites-available/catalog.conf
paste the following:
    ```sh 
        <VirtualHost *:80>
          ServerName 34.201.114.178
          ServerAdmin admin@34.201.114.178
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
          ErrorLog ${APACHE_LOG_DIR}/error.log
          LogLevel warn
          CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
    ```
    1. save file(nano: `ctrl+x`, `Y`, Enter)
    2. Enable `sudo a2ensite catalog`
 -  Create the wsgi file
    1. `cd /var/www/catalog`
    2. `sudo nano catalog.wsgi`
    ```sh 
        #!/usr/bin/python
          import sys
          import logging
          logging.basicConfig(stream=sys.stderr)
          sys.path.insert(0,"/var/www/catalog/")
        
          from catalog import app as application
          application.secret_key = 'Add your secret key'
    ```
    3. save file(nano: ctrl+x, Y, Enter)
    4. sudo service apache2 restart
    
 - Clone Github Repo
    1. sudo git clone https://github.com/Hemalatah/Item-Catalog
    2. make sure you get hidden files iin move shopt -s dotglob. Move files from clone directory to catalog mv /var/www/catalog/Item-Catalog/* /var/www/catalog/catalog/
    3. remove clone directory sudo rm -r Item-Catalog

 - make .git inaccessible
    1. from cd /var/www/catalog/ create .htaccess file sudo nano .htaccess
    2. paste in RedirectMatch 404 /\.git
    3. save file(nano: ctrl+x, Y, Enter)

 - install dependencies:

    1. source venv/bin/activate
    2. pip install httplib2
    3. pip install requests
    4. sudo pip install --upgrade oauth2client
    5. sudo pip install sqlalchemy
    6. pip install Flask-SQLAlchemy
    7. sudo apt install python-psycopg2
    8. If you used any other packages in your project be sure to install those as well.

 - Install and configure PostgreSQL:

    1. Install postgres sudo apt-get install postgresql
    2. install additional models sudo apt-get install postgresql-contrib
    3. by default no remote connections are not allowed
    4. config database_setup.py sudo nano database_setup.py by replacing the engine as engine = create_engine('postgresql://catalog:db-password@localhost/catalog')
    5. repeat for project.py
    6. copy your main project.py file into the init.py file mv project.py __init__.py
    7. Add catalog user sudo adduser catalog
    8. login as postgres super usersudo su - postgres
    9. enter postgrespsql
    10. Create user catalogCREATE USER catalog WITH PASSWORD 'db-password';
    11. Change role of user catalog to creatDBALTER USER catalog CREATEDB;
    12. List all users and roles to verify\du
    13. Create new DB "catalog" with own of catalogCREATE DATABASE catalog WITH OWNER catalog;
    14. Connect to database\c catalog
    15. Revoke all rights REVOKE ALL ON SCHEMA public FROM public;
    16. Give accessto only catalog roleGRANT ALL ON SCHEMA public TO catalog;
    17. Quit postgres\q
    18. logout from postgres super userexit
    19. Setup your database schema python database_setup.py
    
 - fix OAuth to work with hosted Application

    1. Google wont allow the IP address to make redirects so we need to set up the host name address to be usable.
    2. go to http://www.hcidata.info/host2ip.cgi to get your host name by entering your public IP address Udacity gave you.
    3. open apache configbfile sudo nano /etc/apache2/sites-available/catalog.conf
    4. below the ServerAdmin paste ServerAlias YOURHOSTNAME
    5. make sure the virtual host is enabled sudo a2ensite catalog
    6. restart apache server sudo service apache2 restart
    7. In your google developer console and facebook developers page add your host name and IP address to Authorized Javascript origins. And add YOURHOSTNAME/ouath2callback to the Authorized redirect URIs.
