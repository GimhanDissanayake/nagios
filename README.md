# Infastructure and Services Monitoring with Nagios Core 

Using a monitoring system like Nagios is an essential tool for any production environment. You can monitoring uptime, CPU usage, disk space and so on. So you can head off problems before they occur, or before your users call you.You can setup 1 Server with Nagios Core to monitor 2 remote hosts with the following guide.
Here I setup Nagios Server with Nagios Core to Monitor Two Remote Hosts configured as [MySQL Master Slave Replication Servers](https://github.com/GimhanDissanayake/MySQL-Master-Slave-Replication). 


1. Configure Nagios Server with Nagios Core to Monitor localhost(itself)
2. Install Check NRPE Plugging on Nagios Server
3. Install Nagios Plugins and NRPE Daemon on a remote servers (MySQL Master and Slave Servers)
4. Configure Nagios Server to Monitor remote servers (MySQL Master and Slave Servers)


Configure Nagios Server with Nagios Core to Monitor localhost(itself)
========================================================================
- Nagios server needs Apache and PHP installed
- Nagios runs behind a hardware firewall or VPN
- If your Nagios server is exposed to the public internet, must secure the Nagios web interface by installing a TLS/SSL certificate. 

## Step 1 — Installing Nagios 4

- Nagios server needs Apache and PHP installed

### Configure ufw firewall
```
ufw app list
ufw allow OpenSSH
ufw eanble
ufw status
```
### Installing Apache and Updating the Firewall
```
sudo apt update
sudo apt install apache2
```
- apt will tell you which packages it plans to install and how much extra disk space they will take up
```
sudo ufw app list
sudo ufw app info "Apache Full"
sudo ufw allow in "Apache Full"
```
- to find public ip
```
sudo apt install curl
curl http://icanhazip.com
```
### Installing PHP
- PHP  process code to display dynamic content 
- It can run scripts, connect to your MySQL databases to get information, and hand the processed content over to your web server to display.
- PHP code can run under the Apache server and talk to your MySQL database
```
sudo apt install php libapache2-mod-php php-mysql
```
- tell the web server to prefer PHP files over others
```
sudo nano /etc/apache2/mods-enabled/dir.conf
---------------------------------------------
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
---------------------------------------------
service apache2 restart
```

### Installing Nagios
- building Nagios and its components from source
- must install a few development libraries to complete the build, including compilers, development headers, and OpenSSL.
```
sudo apt update
```
- install the required packages:
```
sudo apt install autoconf gcc make unzip libgd-dev libmcrypt-dev libssl-dev dc snmp libnet-snmp-perl gettext
```
- Download the 'source code' for the latest stable release of Nagios Core. 
- Go to the Nagios downloads page.
- Copy the link address for the latest stable release.
```
cd /tmp
curl -L -O https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.6.tar.gz
tar zxf nagios-4.4.4.tar.gz
cd nagioscore-nagios-4.4.4
```
- Before building Nagios, run the 'configure script' and specify the Apache configerations directory:
```
./configure --with-httpd-conf=/etc/apache2/sites-enabled
```
- 'make all' to compile the main program and CGIs.
* Makefile is a program building tool aids in simplifying building program executables that may need various modules.
```
make all
```
- create a nagios user and nagios group. They will be used to run the Nagios process:
```
sudo make install-groups-users
```
- run these make commands to install Nagios binary files, service files, and its sample configuration files:
```
sudo make install
sudo make install-daemoninit
sudo make install-commandmode
sudo make install-config
```
- You will use Apache to serve Nagios web interface
- so run the following to install the Apache configuration files and configure its settings:
```
sudo make install-webconf
```
- Enable the Apache rewrite and cgi modules with the a2enmod command
```
sudo a2enmod rewrite
sudo a2enmod cgi
```
- to issue external commands via the web interface to Nagios, add the web server user, www-data, to the nagios group:
```
sudo usermod -a -G nagios www-data
```
- Use the htpasswd command to create an admin user called nagiosadmin that can access the Nagios web interface:
```
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```
- Enter a password at the prompt. Remember this password
```
nagios123
```
---- Warning: If you create a user with a name other than nagiosadmin, you will need to edit /usr/local/nagios/etc/cgi.cfg and change all the nagiosadmin references to the user you created.
* A CGI script is any program that runs on a web server. (Common Gateway Interface)
* CGI defines a standard way in which information may be passed to and from the browser and server.
* The program run by CGI can be any type of executable file on the server platform. 
* example: C, C++, Perl, Unix shell scripts, Fortran, or any other compiled or interpreted language.
- Restart Apache to load the new Apache configuration:
```
sudo systemctl restart apache2
```
- to work, necessary to install the Nagios Plugins

Installing the Nagios Plugins
==============================
- Nagios needs plugins to operate properly. 
- The official Nagios Plugins package contains over 50 plugins that allow you to monitor basic services 
- such as uptime, disk usage, swap usage, NTP, and others.
- You can find the latest version of the Nagios Plugins on the official site.
```
cd /tmp
curl -L -O https://nagios-plugins.org/download/nagios-plugins-2.3.3.tar.gz
tar zxf nagios-plugins-2.3.3.tar.gz
cd nagios-plugins-2.3.3
```
- configure their installation:
```
./configure
```
- build and install the plugins:
*  A simple make will build the first target in the list, which is put-files . make all will build the target all .
```
make
sudo make install
```
- need one more plugin for monitoring remote servers

Installing the check_nrpe Plugin
================================
- Nagios monitors remote hosts using the "Nagios Remote Plugin Executor"( NRPE ). 
- It consists of two pieces:
	- check_nrpe plugin --> that the Nagios server uses.
	- NRPE daemon --> runs on the remote hosts and sends data to the Nagios server.

### Install the check_nrpe plugin on our Nagios server
- Find the download URL for the latest stable release of NRPE at the GitHub page.
```
cd /tmp
curl -L -O https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.0.3/nrpe-4.0.3.tar.gz
tar zxf nrpe-4.0.3.tar.gz
cd nrpe-4.0.3
```
- Configure the check_nrpe plugin: 
```
./configure
```
*bash script/Guess values for system-dependent variables and create Makefiles.

- build and install check_nrpe plugin:
* convert (a program) into a machine-code or lower-level form in which the program can be executed.
```
make check_nrpe
sudo make install-plugin
```
Configuring Nagios
==================
- perform the initial Nagios configuration
- involves editing some configuration files
- Open the main Nagios configuration file 
```
sudo nano /usr/local/nagios/etc/nagios.cfg
```
- Find this line in the file: - Uncomment this line 
```
#cfg_dir=/usr/local/nagios/etc/servers
```
* tell nagios to config all config files in this directory

- create the directory that will store the configuration file for each server that you will monitor:
```
sudo mkdir /usr/local/nagios/etc/servers
```
- Open the Nagios contacts configuration in your text editor:
```
sudo nano /usr/local/nagios/etc/objects/contacts.cfg
```
- Find the email directive and replace its value with your own email address:
```
define contact{
        contact_name                    nagiosadmin             ; Short name of user
        use                             generic-contact         ; Inherit default values from generic-contact template (defined above)
        alias                           Nagios Admin            ; Full name of user
        email                           your_email@your_domain.com        ; <<***** CHANGE THIS TO YOUR EMAIL ADDRESS ******
--------------------------------------------------------------------------------------------------------------------------------------
Save and exit the editor.
```
- add a new command to your Nagios configuration that lets you use the check_nrpe command in Nagios service definitions. 

- Open the file /usr/local/nagios/etc/objects/commands.cfg in your editor:
```
sudo nano /usr/local/nagios/etc/objects/commands.cfg
```
- Add the following to the end of the file to define a new command called check_nrpe:
```
define command{
        command_name check_nrpe
        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
```
- This defines the name and specifies the command-line options to execute the plugin.

- start Nagios and enable it to start when the server boots:
```
sudo systemctl start nagios
```
- Nagios is now running
```
http://nagios_server_public_ip/nagios
```
Installing Nagios Plugins and NRPE Daemon on a Host
===================================================
- Install the Nagios Remote Plugin Executor (NRPE) on the remote host
- Install some plugins, and configure the Nagios server to monitor this host.
- Log in to the second server
- create a nagios user which will run the NRPE agent:
```
sudo useradd nagios
```
- Install NRPE from source
- need the same development libraries you installed on the Nagios server in 
- Update your package sources and install the NRPE prerequisites
```
sudo apt update
sudo apt install autoconf gcc libmcrypt-dev make libssl-dev wget dc build-essential gettext
```
- Find the latest release of Nagios Plugins from the downloads page.
```
cd /tmp
curl -L -O https://nagios-plugins.org/download/nagios-plugins-2.3.3.tar.gz
tar zxf nagios-plugins-2.3.3.tar.gz
cd nagios-plugins-2.3.3
```
```
./configure
```
```
make
sudo make install
```
- Install NRPE daemon. 
```
cd /tmp
curl -L -O https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.0.3/nrpe-4.0.3.tar.gz
tar zxf nrpe-4.0.3.tar.gz
cd nrpe-4.0.3
```
```
./configure
```
*build and install NRPE and its startup script with these commands:
```
make nrpe
sudo make install-daemon
sudo make install-config
sudo make install-init
```
- update the NRPE configuration file and add some basic checks that Nagios can monitor.
- Use the df -h command to look for the root filesystem. 
- You’ll use this filesystem name in the NRPE configuration:
```
df -h /
```
You Will see output similar to this
```
Output
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1      7.7G  2.8G  5.0G  36% /

/dev/xvda1      7.7G  2.8G  5.0G  36% /
```
- open /usr/local/nagios/etc/nrpe.cfg file in your editor:
```
sudo nano /usr/local/nagios/etc/nrpe.cfg
```
- Locate these settings and alter them appropriately:
```
server_address: Set to the private IP address of the monitored server.
allowed_hosts: Add the private IP address of your Nagios server to the comma-delimited list.
command[check_vda1]: Change /dev/xvda1 to whatever your root filesystem is called.
```
```
server_address=second_ubuntu_server_private_ip

allowed_hosts=127.0.0.1,::1,your_nagios_server_private_ip

command[check_vda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/vda1
```

- start NRPE
```
sudo systemctl start nrpe.service
sudo systemctl status nrpe.service
```
- If you are using UFW, configure it to allow TCP connections to port 5666 
```
sudo ufw allow 5666/tcp
```
- Run the following command on the Nagios server
```
/usr/local/nagios/libexec/check_nrpe -H second_ubuntu_server_ip
/usr/local/nagios/libexec/check_nrpe -H 172.31.18.102        
/usr/local/nagios/libexec/check_nrpe -H 172.31.21.216        
-------------------------------------------------------------------
Output
NRPE v3.2.1
```
* Error --- no route to host ----->  Configure Firewall Setting
 
- Repeat the steps in this section for each additional server you want to monitor.

- have to add these hosts to your Nagios server configuration before it will start monitoring them. 

Monitoring Hosts with Nagios
============================
- add configuration files for each host specifying what you want to monitor. 
- create a new configuration file for each of the remote hosts that you want to monitor in /usr/local/nagios/etc/servers/. 
```
sudo nano /usr/local/nagios/etc/servers/your_monitored_server_host_name.cfg
```
- replacing the host_name value with your remote hostname, the alias value with a description of the host, and the address value with the private IP address of the remote host:
```
/usr/local/nagios/etc/servers/your_monitored_server_host_name.cfg
------------------------------------------------------------------------
define host {
        use                             linux-server
        host_name                       your_monitored_server_host_name
        alias                           My client server
        address                         your_monitored_server_private_ip
        max_check_attempts              5
        check_period                    24x7
        notification_interval           30
        notification_period             24x7
}
```
- With this configuration, Nagios will only tell you if the host is up or down. Let’s add some services to monitor.

- add this block to monitor load average:
```
/usr/local/nagios/etc/servers/your_monitored_server_host_name.cfg
--------------------------------------------------------------------------
define service {
        use                             generic-service
        host_name                       your_monitored_server_host_name
        service_description             Load average
        check_command                   check_nrpe!check_load
}
```

- The use generic-service directive tells Nagios to inherit the values of a service template called generic-service, which is predefined by Nagios.

- add this block to monitor disk usage:
```
/usr/local/nagios/etc/servers/your_monitored_server_host_name.cfg
-------------------------------------------------------------------
define service {
        use                             generic-service
        host_name                       your_monitored_server_host_name
        service_description             /dev/vda1 free space
        check_command                   check_nrpe!check_vda1
}
```
- Now save and quit. Restart the Nagios service to put any changes into effect:
```
sudo systemctl restart nagios
```
- Nagios will check the new hosts and you’ll see them in the Nagios web interface. 


Monitor the MySQL Server
========================

Configuring Nagios Server 
========================
```
*Got Error      Can't locate DBD/mysql.pm    --->     apt-get install libdbd-mysql-perl
	              Cant Connect to MYSQL server  -->    vim /etc/mysql/mysql.conf.d/mysql.cnf
					      bind_address = 127.0.0.1 ---> 0.0.0.0	
```
```
wget  https://labs.consol.de/assets/downloads/nagios/check_mysql_health-2.2.2.tar.gz
tar zxvf check_mssql_health-2.2.2.tar.gz
cd check_mysql_health-2.2.2
```
```
./configure --prefix=/usr/local/nagios --with-nagios-user=nagios --with-nagios-group=nagios --with-perl=/usr/bin/perl
```
```
make
make install
```
- Define check_mysql_health command as follows:
```
vi /usr/local/nagios/etc/objects/commands.cfg
---------------------------------------------
define command{
command_name check_mysql_health
command_line $USER1$/check_mysql_health -H $ARG4$ --username $ARG1$ --password $ARG2$ --port $ARG5$ --mode $ARG3$
}
```
- Enter services to be monitored in master and server cfg files
```
vi /usr/local/nagios/etc/servers/master.cfg
-------------------------------------------
define service{
use local-service
host_name master
service_description MySQL connection-time
check_command check_mysql_health!nagios!nagios!connection-time!172.31.18.102!3306!
}

define service{
use local-service
host_name master
service_description MySQL slave-io-running
check_command check_mysql_health!nagios!nagios!slave-io-running!172.31.18.102!3306!
}

define service{
use local-service
host_name master
service_description MySQL slave-sql-running
check_command check_mysql_health!nagios!nagios!slave-sql-running!172.31.18.102!3306!
}
```
- do same to slave.cfg

- monitor 3 services: Connection-time, io thread and sql thread (replication) status. 

Configure Remote Servers
========================
- Create nagios database user to access database:
```
mysql
grant usage, replication client on *.* to 'nagios'@'172.31.29.46' identified by 'nagios';
```
- add hardcoded command argument to nrpe.cfg
```
vim /usr/local/nagios/etc/nrpe.cfg
```
command[check_mysql_health]=/usr/local/nagios/libexec/check_mysql_health -w 20% -c 10%
```
service nrpe restart
service nrpe status
```
- You can monitor more parameters described here: http://labs.consol.de/nagios/check_mysql_health/

Note: Every time you change configuration file, verify before starting nagios using command
```
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```
- Finally start nagios service and you are done with nagios installation and configuration for monitoring MySQL

- Log in 

![](https://github.com/GimhanDissanayake/nagios/blob/main/login%20prompt.PNG)

- Nagios Web Interface - Home

![](https://github.com/GimhanDissanayake/nagios/blob/main/web%20interface.PNG)

- Nagios Web Interface - Hosts

![](https://github.com/GimhanDissanayake/nagios/blob/main/Host.PNG)
