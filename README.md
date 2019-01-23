# Linux Server Configuration

This is a project for Udacity's [Full Stack Web Developer Nanodegree](https://classroom.udacity.com/nanodegrees/nd004/dashboard/overview). 

## Technologies Used: 
- The Linux distribution is [Ubuntu](https://www.ubuntu.com/download/server) 16.04.
- The virtual private server is [Amazon Lighsail](https://lightsail.aws.amazon.com/).
- The web application to clone  [Item Catalog project](https://github.com/khyatiContractor/ItemCatalog.git)
- The database server is [PostgreSQL](https://www.postgresql.org/).

Deployed Website: http://34.210.184.116

## Steps to setup the Linux server on Amazon lightsail to host above mentioned application on Apache.

### 1. Start a new Ubuntu Linux server instance on Amazon Lightsail 

- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account.
- Go to Create instance. 
- Select `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04`.
- Rename instance.
- Create the instance.
- Instance will be p and running in a while.

### 2. To SSH into the server

- Go to  `Account` on Amazon Lightsail, click `SSH keys` and download Default Private Key.
- Move this file `LightsailDefaultKey-us-west-2.pem` into local folder `~/.ssh` and rename it `lightsail_key.rsa`.
- In git terminal, type: `chmod 600 ~/.ssh/lightsail_key.rsa`.
- To connect to the instance via the terminal: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@http://34.210.184.116`.

### 3. Update and upgrade installed packages

```
sudo apt-get update
sudo apt-get upgrade
```

### 4. Change the SSH port from 22 to 2200

- Edit the `/etc/ssh/sshd_config` file: `sudo nano /etc/ssh/sshd_config`.
- Change the port number on line 5 from `22` to `2200`.
- Restart SSH: `sudo service ssh restart`.

### 5. Configure the Uncomplicated Firewall (UFW)

  ```
  sudo ufw status                  
  sudo ufw default deny incoming   
  sudo ufw default allow outgoing  
  sudo ufw allow 2200/tcp          
  sudo ufw allow www               
  sudo ufw allow 123/udp           
  sudo ufw deny 22  
  sudo ufw enable
  sudo ufw status
  ```

- Status of UFW:

  ```
  Status: active
  
  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                  
  22                         DENY        Anywhere                  
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)             
  22 (v6)                    DENY        Anywhere (v6)
  ```

- `exit`.

- Go to `Manage` option of the Amazon Lightsail Instance, go to `Networking` , and then change the firewall configuration to allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

- From git terminal, run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@34.210.184.116`


## Give `grader` access


### 6. Create a user account named `grader`

- Add user: `sudo adduser grader`. 


### 7. Give `grader` the permission to sudo

- Edits the sudoers file: `sudo visudo`.
- Search for the line that looks like this:  root    ALL=(ALL:ALL) ALL
- Below this line, add a new line to give sudo privileges to `grader` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

### 8. Create an SSH key pair for `grader` using the `ssh-keygen` tool

- On your local git terminal:
  - `ssh-keygen`
  - Enter file `grader_key` in the local directory `~/.ssh`
  - Two files will be generated (  `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
  - `cat ~/.ssh/grader_key.pub` and copy the contents of the file
  - Go to the grader's virtual machine
- Grader's virtual machine:
  - `mkdir .ssh`
  - `sudo nano ~/.ssh/authorized_keys` and paste the content here.
  - `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
  - Check `/etc/ssh/sshd_config` for `PasswordAuthentication` is set to `no`
  - `sudo service ssh restart`
- On the local git terminal: `ssh -i ~/.ssh/grader_key -p 2200 grader@34.210.184.116`.

## Deploy the project

### 9. Configure the local timezone to UTC

- `sudo dpkg-reconfigure tzdata`

### 10. Install and configure Apache to serve a Python mod_wsgi application

- `sudo apt-get install apache2`
- Go to [http:\\34.210.184.116](http:\\34.210.184.116) and you can see default apache home page.
- `sudo a2enmod wsgi`.


### 11. Install and configure PostgreSQL

- `sudo apt-get install postgresql`
- `sudo su - postgres`.
- `psql`.
- Create `catalog` user with the ability to create databases:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'password';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```

- `\du`
  ```
                                     List of roles
   Role name |                         Attributes                         | Member of 
  -----------+------------------------------------------------------------+-----------
   catalog   | Create DB                                                  | {}
   postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
  ```

- `\q`.
- `exit` to exit postgres user
- Create user `catalog`: `sudo adduser catalog`. 
- Give to `catalog` user the permission to sudo. 
- `sudo visudo`.
- ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  catalog  ALL=(ALL:ALL) ALL
  ```
- As `catalog`, `createdb catalog`.

### 12. Install git

- As `grader`, `sudo apt-get install git`.
- Go to `/var/www/FlaskApp/` directory.
- `sudo git clone https://github.com/khyatiContractor/ItemCatalog.git`
- `sudo chown -R grader:grader catalog/`
- Go to `/var/www/FlaskApp/FlaskApp`.
- `mv project.py __init__.py`.

- In `__init__.py` change below mentiond line:
  ```
  # app.run(host="0.0.0.0", port=8000, debug=True)
  app.run()
  ```

- In `database_setup.py`, `CatalogDB.py` and `__init__.py` change below mentiond line:
   ```
   # engine = create_engine("sqlite:///catalog.db")
   engine = create_engine('postgresql://catalog:password@localhost/catalog')
   ``` 

### 13. Install the virtual environment and dependencies

- As `grader` do `sudo apt-get install python-pip`.
- `sudo apt-get install python-virtualenv`
- Go to `/var/www/FlaskApp/FlaskApp/` directory.
- `sudo virtualenv -p python venv`.
- `sudo chown -R grader:grader venv/`.
- `. venv/bin/activate`.
- ```
  pip install httplib2
  pip install requests
  pip install --upgrade oauth2client
  pip install sqlalchemy
  pip install flask
  sudo apt-get install libpq-dev
  pip install psycopg2
  ```
- Run `python __init__.py` and you should see:


### 14. Set up and enable a virtual host

- In file `/etc/apache2/mods-enabled/wsgi.conf`
  ```
  #WSGIPythonPath directory|directory-1:directory-2:...
  WSGIPythonPath /var/www/FlaskApp/FlaskApp/venv/lib/python2.6/site-packages
  ```
- Create `/etc/apache2/sites-available/FlaskApp.conf`

  ```
  <VirtualHost *:80>
	  ServerName 34.210.184.116
	  WSGIScriptAlias / /var/www/FlaskApp/FlaskApp.wsgi
	  <Directory /var/www/FlaskApp/FlaskApp/>
	  	Order allow,deny
		  Allow from all
	  </Directory>
	  Alias /static /var/www/FlaskApp/FlaskApp/static
	  <Directory /var/www/FlaskApp/FlaskApp/static/>
		  Order allow,deny
		  Allow from all
	  </Directory>
	  ErrorLog ${APACHE_LOG_DIR}/error.log
	  LogLevel warn
	  CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

- `sudo a2ensite catalog`. 
- Reload : `sudo service apache2 reload`.


### 15. Set up the Flask application

- Create `/var/www/FlaskApp/FlaskApp.wsgi`:

  ```
  activate_this = '/var/www/FlaskApp/FlaskApp/venv/bin/activate_this.py'
  with open(activate_this) as file_:
      exec(file_.read(), dict(__file__=activate_this))

  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/FlaskApp/FlaskApp/")
  sys.path.insert(1, "/var/www/FlaskApp/")

  from catalog import app as application
  application.secret_key = "super_secret_key"
  ```

- `sudo service apache2 restart`.

### 16. Set up the database schema and populate the database
- `. venv/bin/activate`.
- `python CatalogDB.py`.
- `deactivate`.

### 17. Disable the default Apache site

- `sudo a2dissite 000-default.conf`
- `sudo service apache2 reload`.

### 18. Launch the Web Application

- `sudo service apache2 restart`.
- Go to [http://34.210.184.116](http://34.210.184.116)



### Resources used :
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://www.digitalocean.com/community/tutorials/how-to-tune-your-ssh-daemon-configuration-on-a-linux-vps
https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps
https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
