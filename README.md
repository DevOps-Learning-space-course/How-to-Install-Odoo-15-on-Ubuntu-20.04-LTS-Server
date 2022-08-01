How to Install Odoo 15 on Ubuntu 20.04 LTS Server

sudo apt-get update
sudo apt-get upgrade


<h1>Step 2: Secure Server</h1>
Ensure the system is secure from ssh attacks, the use of Fail2ban will help to prevent ssh attacks:

<h4>sudo apt-get install openssh-server fail2ban</h4>

<h1>Step 3: Install Python 3 and its Dependencies</h1>

Install the required python packages for Odoo:
Install pip3:

sudo apt-get install -y python3-pip


Then install Packages and libraries:

sudo apt-get install python-dev python3-dev libxml2-dev libxslt1-dev zlib1g-dev libsasl2-dev libldap2-dev build-essential libssl-dev libffi-dev libmysqlclient-dev libjpeg-dev libpq-dev libjpeg8-dev liblcms2-dev libblas-dev libatlas-base-dev

Make sure that all packages are installed correctly without any errors. After successful installation of Python packages, some web dependencies are also needed to be installed. 


sudo apt-get install -y npm

sudo ln -s /usr/bin/nodejs /usr/bin/node

sudo npm install -g less less-plugin-clean-css

sudo apt-get install -y node-less

Step 4: Setup Database Server(PostgreSQL)
Odoo uses PostgreSQL as its database server. Follow the steps to install and setup database server for Odoo:


sudo apt-get install postgresql



In the next step, create a Postgres user to handle the database. The user and given password are needed for the conf file later.
Postgres has its own system user called ‘Postgres to perform the operations. So next command for change user to Postgres:


sudo su - postgres

Next, let's create a database user for Odoo15. When you enter the following command, it will ask for a password and re-enter it again. Remember this for later use:



createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo15




The following command ensures that the user has superuser access rights:


psql
ALTER USER odoo15 WITH SUPERUSER;


Exit from psql and Postgres user:

\q
exit


Step 5: System User

Next let's create a system user to perform Odoo roles and also for security purposes. All the files and directories of Odoo’s access and operations will be limited for this user. 
Now let us create a new system user for the Odoo service and further then we will limit the permissions of all Odoo related files and directories for this specific user.


sudo adduser --system --home=/opt/odoo --group odoo


Step 6: Clone Odoo Source from Github Repository

With the Community Edition source code, we can directly clone from Odoo’s GitHub repository. You can add the Enterprise edition add-ons after the installation process is completed.
So first install git to the server:

sudo apt-get install git

Next, switch system user to ‘odoo’ and the files will be added into the user’s home directory:

sudo su - odoo -s /bin/bash


The following command will clone the source directory and the operator dot(.) at the end of the command is used to clone the files to the home directory of the current user which is /opt/odoo and is the same home directory mentioned at the time of user creation:

git clone https://www.github.com/odoo/odoo --depth 1 --branch 15.0 --single-branch .

exit


Step 7: Install Required Python Packages
The next step is to install the required packages. All the packages are listed in the requirement.txt file. Therefore, we can easily install these packages with a single command:

sudo pip3 install -r /opt/odoo/requirements.txt


To run Odoo smoothly, all the packages should be installed properly and you should ensure that.




Step 8: Install Wkhtmltopdf
Odoo supports printing reports as PDF files. Wkhtmltopdf helps to generate PDF reports from HTML data format. Moreover, the Qweb template reports are converted to HTML format by the report engine and Wkhtmltopdf will produce the PDF report:



sudo wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.bionic_amd64.deb

sudo dpkg -i wkhtmltox_0.12.5-1.bionic_amd64.deb

sudo apt install -f

Step 9: Setup Conf file
Next, we have to configure the conf file for Odoo which contains certain necessary information such as the addons path, database-related parameters, proxy parameters, and many more. 
Therefore, you should create a configuration file inside the /etc directory. There is a sample conf file inside Odoo’s source, in the Debian directory. To copy from Debian to the /etc directory use the following command:

sudo cp /opt/odoo/debian/odoo.conf /etc/odoo.conf


sudo nano /etc/odoo.conf

Update admin password and db_password from the following sample

[options]
   ; This is the password that allows database operations:
   admin_passwd = admin
   db_host = localhost
   db_port = 5432
   db_user = odoo15
   db_password = odoo
   addons_path = /opt/odoo/addons
   logfile = /var/log/odoo/odoo.log


Finally, you should set access rights of the conf file for the system user odoo:

sudo chown odoo: /etc/odoo.conf

sudo chmod 640 /etc/odoo.conf

And create a log directory to store the log file of odoo which will help you to find Odoo related issues and also set permissions for the user odoo as we did earlier:


sudo mkdir /var/log/odoo
sudo chown odoo:root /var/log/odoo


Step 10: Odoo service file
Finally, we have to create a service to run Odoo. Let’s create a service file ‘odoo.service’ in /etc/systemd/system:



sudo nano /etc/systemd/system/odoo.service

[Unit]
   Description=Odoo
   Documentation=http://www.odoo.com
   [Service]
   # Ubuntu/Debian convention:
   Type=simple
   User=odoo
   ExecStart=/opt/odoo/odoo-bin -c /etc/odoo.conf
   [Install]
   WantedBy=default.target



Next set the permissions for the root user to this service file:


sudo chmod 755 /etc/systemd/system/odoo.service

sudo chown root: /etc/systemd/system/odoo.service

Step 11: Test Odoo 15
Now all the steps of installation are completed. Let's test the Odoo instance with the following command:

sudo systemctl start odoo.service


Then check the status of the service using the following command. And if it depicts as active, the installation of Odoo was successful:


sudo systemctl status odoo.service


Now you can access Odoo by entering the following URL:

“http://<your_domain_or_IP_address>:8069”


Check Odoo logs
You can also check the logs of the Odoo platform that you have set up if you are facing any issues related to the installation or any other reasons with the following command. This command will show you the live logs in the terminal:


sudo tail -f /var/log/odoo/odoo.log




At last, if you want to start the Odoo service automatically after rebooting the server, use the following command:


sudo systemctl enable odoo.service

If you have made any changes in the addons, restart the Odoo service to reflect the updates on your instance using the following command:


sudo systemctl restart odoo.service







