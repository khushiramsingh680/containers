+++
title = "Project 01"
weight = 12
+++

## -------------------------------
### 1. DB Server Configuration
## -------------------------------
```txt
yum install mariadb-server -y
systemctl enable mariadb
systemctl start mariadb
```
## Configure Database
```txt
mysql -e "
CREATE DATABASE studentapp;
USE studentapp;
CREATE TABLE Students (
    student_id INT NOT NULL AUTO_INCREMENT,
    student_name VARCHAR(100) NOT NULL,
    student_addr VARCHAR(100) NOT NULL,
    student_age VARCHAR(3) NOT NULL,
    student_qual VARCHAR(20) NOT NULL,
    student_percent VARCHAR(10) NOT NULL,
    student_year_passed VARCHAR(10) NOT NULL,
    PRIMARY KEY (student_id)
);
GRANT ALL PRIVILEGES ON studentapp.* TO 'student'@'%' IDENTIFIED BY 'student@1';
FLUSH PRIVILEGES;
"
```
### -------------------------------
### 2. Tomcat (Application Server)
## -------------------------------
```sh
yum install java -y
cd /root
wget -qO- http://www-us.apache.org/dist/tomcat/tomcat-8/v8.5.27/bin/apache-tomcat-8.5.27.tar.gz | tar -xz
cd apache-tomcat-8.5.27
rm -rf webapps/*
wget https://github.com/cit-latex/stack/raw/master/mysql-connector-java-5.1.40.jar -O lib/mysql-connector-java-5.1.40.jar
wget https://github.com/cit-latex/stack/raw/master/student.war -O webapps/student.war
```
## Edit context.xml (replace <IP-ADDRESS-OF-DB-SERVER> accordingly)
```sh
sed -i '/<\/Context>/i \<Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource" maxTotal="100" maxIdle="30" maxWaitMillis="10000" username="student" password="student@1" driverClassName="com.mysql.jdbc.Driver" url="jdbc:mysql://<IP-ADDRESS-OF-DB-SERVER>:3306/studentapp"/>' conf/context.xml

### Start Tomcat
sh bin/startup.sh
```
## -------------------------------
### 3. Web Server (Apache HTTPD)
## -------------------------------
```sh
yum install httpd httpd-devel gcc -y
cd /root
wget -qO- http://www-eu.apache.org/dist/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.42-src.tar.gz | tar -xz
cd tomcat-connectors-1.2.42-src/native
./configure --with-apxs=/usr/bin/apxs
make
make install

## Configure worker.properties (replace <IP-ADDRESS-OF-TOMCAT-SERVER> accordingly)
cat << EOF > /etc/httpd/conf.d/worker.properties
worker.list=tomcatA
worker.tomcatA.type=ajp13
worker.tomcatA.host=<IP-ADDRESS-OF-TOMCAT-SERVER>
worker.tomcatA.port=8009
EOF

### Configure mod_jk.conf
cat << EOF > /etc/httpd/conf.d/mod_jk.conf
LoadModule jk_module modules/mod_jk.so
JkWorkersFile conf.d/worker.properties
JkMount /student tomcatA
JkMount /student/* tomcatA
EOF

### Enable and start HTTPD
systemctl enable httpd
systemctl start httpd
```

## 3 tier App in conatiners using docker compose 

### Create necessary directory
```sh
mkdir -p docker-app && cd docker-app
```
### Create context.xml
```sh
cat << EOF > context.xml
<Context>
  <Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
            maxTotal="100" maxIdle="30" maxWaitMillis="10000"
            username="student" password="student@1"
            driverClassName="com.mysql.jdbc.Driver"
            url="jdbc:mysql://mariadb:3306/studentapp"/>
</Context>
EOF
```
### Download mysql connector
```sh
wget https://github.com/cit-latex/stack/raw/master/mysql-connector-java-5.1.40.jar -O mysql-connector-java-5.1.40.jar
```
### Download student.war
```sh
wget https://github.com/cit-latex/stack/raw/master/student.war -O student.war
```
### Create worker.properties
```sh
cat << EOF > worker.properties
worker.list=tomcatA
worker.tomcatA.type=ajp13
worker.tomcatA.host=tomcat
worker.tomcatA.port=8009
EOF
```
### Create mod_jk.conf
```sh
cat << EOF > mod_jk.conf
LoadModule jk_module modules/mod_jk.so
JkWorkersFile conf.d/worker.properties
JkMount /student tomcatA
JkMount /student/* tomcatA
EOF
```
### Create docker-compose.yml
```sh
cat << EOF > docker-compose.yml
version: '3.8'

services:

  mariadb:
    image: mariadb:10.5
    container_name: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: studentapp
      MYSQL_USER: student
      MYSQL_PASSWORD: student@1
    networks:
      - app-network
    volumes:
      - mariadb_data:/var/lib/mysql

  tomcat:
    image: tomcat:8.5
    container_name: tomcat
    depends_on:
      - mariadb
    ports:
      - "8080:8080"
    volumes:
      - ./context.xml:/usr/local/tomcat/conf/context.xml
      - ./mysql-connector-java-5.1.40.jar:/usr/local/tomcat/lib/mysql-connector-java-5.1.40.jar
      - ./student.war:/usr/local/tomcat/webapps/student.war
    networks:
      - app-network

  httpd:
    image: httpd:2.4
    container_name: httpd
    depends_on:
      - tomcat
    ports:
      - "80:80"
    volumes:
      - ./worker.properties:/etc/httpd/conf.d/worker.properties:ro
      - ./mod_jk.conf:/etc/httpd/conf.d/mod_jk.conf:ro
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  mariadb_data:
EOF
```
### Run the setup
docker-compose up -d

### Show running containers
docker ps
