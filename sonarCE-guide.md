# SonarQube v9.6 Install Instructions

**Creds:** maintuser/pw

[YouTube Link for Directions](https://www.youtube.com/watch?v=Afkp0aM8mmU&t=200s&ab_channel=MivoCloud)

## 1. Install OpenJDK
Install OpenJDK Version 11

```bash
# Sudo as root
sudo su -

# Prep System
sudo apt install unzip
sudo apt update
sudo apt install -y

# Install Java
sudo apt install default-jdk
```
## 2. Install PostgreSQL 13

```bash
# Import Repo Key
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -

# Add Repository
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'

sudo apt update

# Install PostgreSQL
sudo apt install postgresql-13

sudo systemctl status postgresql
```

## 3. Configure PostgreSQL
This step we will configure PostgreSQL and its Database

```bash
# Log into shell
sudo -u postgres psql

# Perform Configs
CREATE USER sonarqube WITH PASSWORD 'Password'; #Mivo
CREATE DATABASE sonarqube OWNER sonarqube;
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonarqube;

# Quit
\l
\q
```

## 4. Configure PostgreSQL

```bash
# Create User
sudo useradd -b /opt/sonarqube -s /bin/bash sonarqube

# Create this file and add contents to the end
sudo nano /etc/sysctl.conf

# add to end of file
vm.max_map_count=524288
fs.file-max=131072

# restart sysctl
sudo sysctl --system

# pass in ulimits. Note: this may not persist over restarts 
ulimit -n 131072
ulimit -u 8192

# Update the file limits
sudo nano /etc/security/limits.d/99-sonarqube.conf

sonarqube   -   nofile   131072
sonarqube   -   nproc    8192

# Crab sonarqube zip file
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.6.1.59531.zip
unzip sonarqube-9.6.1.59531.zip

# Move and change ownership to `sonaruser`
mv sonarqube-9.6.1.59531 /opt/sonarqube
sudo chown -R sonarqube:sonarqube /opt/sonarqube
```

# 4. Configure Sonarqube Server

Open the Sonarqube Configuration File

```bash
# Open File
nano /opt/sonarqube/conf/sonar.properties

# Modify jdbc
sonar.jdbc.username=sonaruser
sonar.jdbc.password=Mivo

# Modify jdbc url
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube

# Uncomment Elastic Memory Heap
#JVM options of Elasticsearch process
sonar.search.javaOpts=-Xmx512m.....

# Uncomment Logs
sonar.log.level-INFO
sonar.path.logs=

# Create this file 
sudo nano /etc/systemd/system/sonarqube.service

# Add properties
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonarqube
Group=sonarqube
Restart=always
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target

# Restart the services
sudo systemctl daemon-reload
sudo systemctl start sonarqube.service
sudo systemctl enable sonarqube.service
sudo systemctl status sonarqube.service
```

## 5. Check Statuses
Run the below commands to check the status

```bash
# Install Net-Tools
sudo apt install net-tools
netstat -plnt

# Check the logs
tail /opt/sonarqube/logs/sonar.log
```
