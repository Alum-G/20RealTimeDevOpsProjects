Lets Get Started!

### Prerequisites / Tools Used
Vagrant<br>
MariaDB<br>
MySQL<br>
JDK<br>

Clone the Github repository to local machine <br>
```git clone https://github.com/devopshydclub/vprofile-project```

Switch to the local-setup branch<br>
```git checkout local-setup```

Head to the vagrant directory and switch to a Manual_provisioning file that fits your OS. I am on MacOS M1 so I'll go into that<br>
```cd vagrant && cd Manual_provisioning_MacOSM1```

Install the vagrant host manager plug-in to add host enteries and map it with the IP addresses of the VMs<br>
```vagrant plugin install vagrant-hostmanager```

The ... uses the vagrant file to create virtual machines<br>

## Provisioning the MySQL service

In the present directory run the vagrant file <br>
```vagrant up```

ssh into the db01 Virtual machine, set up services manually<br>
```vagrant ssh db01```

Switch to the root user<br>
```sudo -i```

Update to all the latest packages on all VMs<br>
```yum update -y```

Set up a password variable for the database VM<br>
```DATABASE_PASS='{Your password here}'```

Right now this is temporary. Make it permanent by placing the above variable into an /etc/profile file<br>
```vi /etc/profile/``` 

Install the MariaDB package using<br>
```yum install mariadb-server -y```

Start the MariaDB service<br>
```systemctl start mariadb```

Enable the MariaDB service<br>
```systemctl enable mariadb```

Confirm status of the MariaDB service<br>
``` systemctl status mariadb```



Clone the source code to get the SQL file to enable us initialize our Database<br>
```git clone -b local-setup https://github.com/devopshydclub/vprofile-project.git```

Go into the resources directory<br>

Set up Database "accounts" in resource directly<br>
```mysql -u root -p"$DATABASE_PASS" -e "create database accounts"```

Add Admin user in the DB accesible from app01 and has full privileges on the accounts database<br>
```mysql -u root -p"$DATABASE_PASS" -e "grant all privileges on accounts.* TO 'admin'@'app01' identified by '{Your Password}'"```

Head to project directory and run the db_backup.sql query on DB<br>
``` mysql -u root -p"DATABASE_PASS" accounts < src/main/resources/db_backup.sql```
``` mysql -u root -p"DATABASE_PASS" accounts < src/main/resources/db_backup.sql```

Verify database<br>
```
mysql -u root -p"$DATABASE_PASS"
MariaDB [(none)]> show databases;
MariaDB [(none)]> use accounts;
MariaDB [(none)]> show tables;
exit
```

Logout of VM<br>

## Provisioning Memcache Service
* ssh into the mc01 Virtual machine, set up services manually<br>
```vagrant ssh mc01```

* Switch to the root user<br>
```sudo -i```

* Update to all the latest packages<br>
```yum update -y```

* Install Memchached Package
```yum install memcached -y```

* Start the Memcached service<br>
```systemctl start memcached```

* Enable the Memcached service<br>
```systemctl enable memcached```

* Confirm status of the Memcached service<br>
```systemctl status memcached```

* Run the command below to ensure listenability on TCP Port 11211 and UDP port 11111
```memcached -p 11211 -U 11111 -u memcached -d```

* Logout of VM

## Provisioning RabbitMQ Service
* ssh into the mc01 Virtual machine, set up services manually<br>
```vagrant ssh mc01```

* Switch to the root user<br>
```sudo -i```

* Update to all the latest packages<br>
```yum update -y```

* Install wget<br>
```yum install wget -y```

* Installing dependencies<br>

* cd into the /tmp directory<br>

* Download and install rpm to get erlang repo using wget<br>
```wget http://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpm```

* Download and excecute shell script to get RabbitMQ server packages<br>
```sudo rpm -Uvh erlang-solutions-2.0-1.noarch.rpm```

* Install erlang<br>
```sudo yum -y install erlang socat```

* Download the script and pipe with bash to download RabbitMQ<br>
```curl -s https://packagecloud.io/install/repositories/rabbitmq-server/script.rpm.sh | sudo bash```

Install RabbitMQ Server
```sudo yum install rabbitmq-server-y```

* Start the Memcached service<br>
```systemctl start memcached```

* Enable the Memcached service<br>
```systemctl enable memcached```

* Confirm status of the Memcached service<br>
```systemctl status memcached```


* Create user with password and grant administrative privileges<br>
```
cd ~
echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config
rabbitmqctl add_user test test
rabbitmqctl set_user_tags test administrator
```

* Bounce the RabbitMQ service and Check the rabbitmq status<br>
```systemctl restart rabbitmq-server```
```systemctl status rabbitmq-server```

* Exit the service<br>


## Provisioning Tomcat
---
* Log into the Tomcat VM<br>
```vagrant ssh app01```

* Swicth to root user<br>
```sudo -i```

* Install the tomcat service<br>
```sudo yum install tomcat -y```

* Install Java dependencies<br>
```yum install java-1.8.0-openjdk -y```

* Install Git, Maven, Wget<br>
```yum install git maven wget -y```

* In the tmp directory<br>
```wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.tar.gz```

* Add tomcat user - tomcat00<br>
```useradd --home-dir /usr/local/tomcat8 --shell /sbin/nologin tomcat00```

* Copy the apache-tomcat-8.5.37 file into the tomcat00 home directory<br>
```cp -r apache-tomcat-8.5.37/* /usr/local/tomcat8```

* Change the ownership of the /usr/local/tomcat8 directory and its contents recursively (-R option) to the user tomcat000 and the group tomcat000<br>
```ls -ld /usr/local/tomcat8```
```chown tomcat000.tomcat000 /usr/local/tomcat8 -R```

* Edit the systemd service unit file for the Tomcat service.
```
[Unit]
Description=Tomcat
After=network.target

[Service]
User=tomcat000
WorkingDirectory=/usr/local/tomcat8
Environment=JRE_HOME=/usr/lib/jvm/jre
Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_HOME=/usr/local/tomcat8
Environment=CATALINE_BASE=/usr/local/tomcat8
ExecStart=/usr/local/tomcat8/bin/catalina.sh run
ExecStop=/usr/local/tomcat8/bin/shutdown.sh
SyslogIdentifier=tomcat-%i

[Install]
WantedBy=multi-user.target
```

* Reload the systemd daemon to read any changes made to unit files<b r>
```systemctl daemon-reload```

* Start the Tomcat service<br>
```systemctl start tomcat```

* Enable the Tomcat service<br>
```systemctl enable tomcat```

* Confirm status of the Tomcat service<br>
```systemctl status tomcat```

* In the tmp directory, clone the source code
```git clone -b local-setup https://github.com/devopshydclub/vprofile-project.git```

* Head into src/main/resources/application.properties file to ensure setting are correct 
```
#JDBC Configutation for Database Connection
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://db01:3306/accounts?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
jdbc.username=admin
jdbc.password=admin123

#Memcached Configuration For Active and StandBy Host
#For Active Host
memcached.active.host=mc01
memcached.active.port=11211
#For StandBy Host
memcached.standBy.host=127.0.0.2
memcached.standBy.port=11211

#RabbitMq Configuration
rabbitmq.address=rmq01
rabbitmq.port=5672
rabbitmq.username=test
rabbitmq.password=test
```

* Go back to the home directory, Run the "mvn install" command which will create our artifact. The artifact will be created /tmp/

vprofile-project/target/vprofile-v2.war
```cd target/
ls
```

* Stop the default tomcat service<br>
```systemctl stop tomcat```

* Remove the default application<br>
```rm -rf /usr/local/tomcat8/webapps/ROOT```

* In the vprofile-project directory, replace the default application with the artifact<br>
```cp target/vprofile-v2.war /usr/local/tomcat8/webapps/ROOT.war```

* That copies the ROOT.war file into the ROOT directory

* Exit the service

## Provisioning the Nginx Service

* ssh into the web01 Virtual machine, set up services manually<br>
```vagrant ssh web01```

* web01 is an Ubuntu VM. Other VMs were CentOS but web01 is Ubuntu

* Update the packages in the VM
```sudo apt update && sudo apt upgrade -y```

* Switch to the root user<br>
```sudo -i```

* Install Nginx Package
```apt install nginx -y```

* We need to create a configuration file used to redirect requests from Nginx to Tomcat server on Port 8080
```vim /etc/nginx/sites-available/vproapp```

```
upstream vproapp {
server app01:8080;
}
server {
listen 80;
location / {
proxy_pass http://vproapp;
}
}
```

* Remove the default website in Nginx
```rm -rf /etc/nginx/sites-enabled/default```

* Create Link of our configuration file and replace with
```ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp```