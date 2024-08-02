
# vagrant-multi-tier-app

This project outlines the creation and management of a multi-tier web application environment using Vagrant. The infrastructure consists of five virtual machines, each dedicated to a specific role: MySQL, Memcached, RabbitMQ, Tomcat, and Nginx. The project aims to streamline the development and deployment process by automating the provisioning and configuration of these virtual machines.




![Logo](https://vrofile-poto.s3.amazonaws.com/vprofile.png)


## VM SETUP

- Clone source code
- Cd into the repository
- copy vagrant file into another directory
- bring vm's up


```bash
  vagrant up
```
## MYSQL SETUP

- ssh into db01

```bash
  vagrant ssh db01
```

- Install Maria DB Package
- Start and enable Maria DB 
- Create database named accounts and grant all privileges 
- Initialize database

```bash
  mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
```
- Restart mariadb-server

## MEMCACHE SETUP

- ssh into mc01

```bash
  vagrant ssh mc01
```

- Install, start & enable memcache

```bash
 sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached

```
- Restart memcached

## RABBITMQ SETUP

- ssh into rmq01

```bash
  vagrant ssh rmq01
```

- Install Dependencies

```bash
 dnf -y install centos-release-rabbitmq-38
 dnf --enablerepo=centos-rabbitmq-38 -y install rabbitmq-server
 systemctl enable --now rabbitmq-server
```

- Setup access to user test and make it admin

```bash
sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
sudo rabbitmqctl add_user test test
sudo rabbitmqctl set_user_tags test administrator
```
- restart rabbitmq-server 

## MYSQL SETUP

- ssh into tomcat vm

```bash
  vagrant ssh app01
```

- Install Dependencies

```bash
  dnf -y install java-11-openjdk java-11-openjdk-devel
  dnf install git maven wget -y
```
- Download & Tomcat Package

## CODE BUILD & DEPLOY

- Download Source code

```bash
  git clone -b main https://github.com/Induru-dev/vagrant-multi-tier-app.git

```
- Update configuration

```bash
   vim src/main/resources/application.properties
   Update file with backend server details
```

- Update file with backend server details

### Build code

```bash
  mvn install
```
- Deploy artifact

```bash
  systemctl stop tomcat
  rm -rf /usr/local/tomcat/webapps/ROOT*
  cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
  systemctl start tomcat
  chown tomcat.tomcat /usr/local/tomcat/webapps -R
  systemctl restart tomcat

```
## NGINX SETUP

- Login to the Nginx vm

```bash
  vagrant ssh web01
```
- Install nginx
- Create Nginx conf file
- Update with below content

```bash
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
- Remove default nginx conf

```bash
# rm -rf /etc/nginx/sites-enabled/default
```

- Create link to activate website

```bash
# ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
```

- Restart Nginx


### Now your web app shold work with web01 ip address
