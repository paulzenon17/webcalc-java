# Dev-sec-Ops
### Github+Jenkins+Maven+Tomcat+MySQL
Requirements:
Ubuntu 22.04

Jenkins

Maven

Tomcat

MySQL

Git commands:
------------

```
git clone <repo_url>
git add .
git commit -m "message"
git push origin -u <branch_name>
git branch 			#To check barnch name
git checkout -b <branch_name> 	#To create banch name
git checkout <branch-name> 	#To switch branch
```

#### Jenkins Installations in ubuntu22.04:

```
sudo apt update
sudo apt install openjdk-11-jdk -y
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key |sudo gpg --dearmor -o /usr/share/keyrings/jenkins.gpg
sudo sh -c 'echo deb [trusted=yes] https://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
sudo systemctl status jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Once complete the installation open your browser 
```
http://<your_ip>:8080/
```
ex: http://192.168.138.110:8080/

Now give administrative password  
to check administrative password ```sudo cat /var/lib/jenkins/secrets/initialAdminPassword```

now select "Install suggested plugins" option 

Create user details then save & continue.

##### Install plugins: Maven, deploy to container
To install plugins: Go to Manage Jenkins > Plugins > available plugins

Maven installation
-----------------
```
sudo apt install maven -y
```
integrate java and maven with jenkins :
---------------------------------------
configure JAVA_HOME // to get this ```sudo find / -name java-11-openjdk*```

path like : /usr/lib/jvm/java-11-openjdk-amd64


Configure MAVEN_HOME // to get this ```mvn --version```

path like : /usr/share/mvn

Go to jenkins>manage Jenkins> Tools> Maven
![Screenshot from 2024-02-29 15-33-12](https://github.com/beeru405/webcalc-java/assets/101712802/8d4879a5-0e31-419e-bb04-032acc3fc225)

Go to jenkins>manage Jenkins> Tools> jdk
![Screenshot from 2024-02-29 15-32-25](https://github.com/beeru405/webcalc-java/assets/101712802/8fd6e732-9792-4e69-90fd-8bdd955c5324)

Tomcat Installation:
------------------
```
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.86/bin/apache-tomcat-9.0.86.tar.gz
tar -xvzf apache-tomcat-9.0.86.tar.gz
```
To start tomact 
```
cd apache-tomcat-9.0.86
cd bin/
./startup.sh
```
To stop tomcat
```
cd apache-tomcat-9.0.86
cd bin/
./shutdown.sh
```

If running jenkins and tomcat in single machine you shoud change port of tomcat or jenkins because both default port is 8080

To change port number of tomcat

go to tomcat 
```
cd apache-tomcat-9.0.86.tar.gz
cd conf/
#nano server.xml
#change port number instead 8080 change any number like 8090
```
#### Check point :
Access tomcat application from browser on port 8090  
 - http://<Public_IP>:8090

1. now application is accessible on port 8090. but tomcat application doesnt allow to login from browser. changing a default parameter in context.xml does address this issue
   ```sh
   #search for context.xml
   find / -name context.xml
   ```
1. above command gives 3 context.xml files. comment (<!-- & -->) `Value ClassName` field on files which are under webapp directory. 
After that restart tomcat services to effect these changes. 
At the time of writing this lecture below 2 files are updated. 
   ```sh 
   /opt/tomcat/webapps/host-manager/META-INF/context.xml
   /opt/tomcat/webapps/manager/META-INF/context.xml
   
   # Restart tomcat services
   ```
1. Update users information in the tomcat-users.xml file
goto tomcat home directory and Add below users to conf/tomcat-users.xml file
   ```sh
	<role rolename="manager-gui"/>
	<role rolename="manager-script"/>
	<user username="admin" password="admin" roles="manager-gui, manager-script"/>
	<user username="deployer" password="deployer" roles="manager-script"/>
	<user username="tomcat" password="tomcat" roles="manager-gui"/>
   ```
1. Restart serivce and try to login to tomcat application from the browser. This time it should be Successful

Now Setup tomcat in Jenkins:
---------------------------
Go to mange jenkins>credentials>system>globalcredential> Add credential

give tomcat credentials
![Screenshot from 2024-02-29 15-30-12](https://github.com/beeru405/webcalc-java/assets/101712802/625c1ee2-7ceb-4d51-ac88-0d76e21c0e77)



-------
Install MySQL:
--------------
```
sudo apt update
sudo apt install mysql-server -y
sudo systemctl start mysql.service
sudo mysql_secure_installation
```
mysql user creation
```
CREATE USER 'mysql'@'%' IDENTIFIED BY 'mysql' ;
```
Grant All Privileges:
```
GRANT ALL PRIVILEGES ON *.* TO 'mysql'@'%' WITH GRANT OPTION;
```
Flush Privileges:
```
FLUSH PRIVILEGES;
```
Create database 
```
create database myDB;
```
use database
```
use myDB
```
create table
```
CREATE TABLE calculations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    operation VARCHAR(255) NOT NULL,
    result BIGINT NOT NULL
);
```

To check the data 
```
select * from calculations;
```
## Sonarqube Installation
-----------------------------
We will be installing Docker for installing Sonarqube in the container
```
sudo apt-get update
sudo apt-get install docker.io -y
```

After this gran the Ubuntu user the permission to access Docker
```
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 777 /var/run/docker.sock
```
Starting Sonarqube container:
```
docker run -d --name sonarqube-cont -p 9000:9000 sonarqube:lts-community
```
#To login
```
http://<PUBLIC-IP-OF-INSTANCE>:9000
```
Login the SonarQube using the default credentials
```
username: admin
password: admin
```
![image](https://github.com/beeru405/webcalc-java/assets/101712802/b2d8cfcf-3bba-4a64-862e-a0fa507e9213)

After Deafult login, You will be asked for the new SonarQube password. Give the password and click Save

![image](https://github.com/beeru405/webcalc-java/assets/101712802/3453feb2-319e-4ff1-ad0d-0e8062bfa1d9)

Jenkins PLugin setup for SonarQube,

Step 1: Go to Manage Jenkins in the Jenkins Dashboard.

![image](https://github.com/beeru405/webcalc-java/assets/101712802/b585d642-5900-4aef-948c-972a52b7e020)

Step 2: Select Plugin from the option.

![image](https://github.com/beeru405/webcalc-java/assets/101712802/447a4a72-dfd3-4e4b-9796-c4bda8fa97f0)


Step 3: Select Available plugins search for SonarQube Scanner, and click on Install. (without restart)

![image](https://github.com/beeru405/webcalc-java/assets/101712802/4371ea0f-f39d-4c4f-a44a-4db37a43f10d)



After this stage, we will set up the SonarQube to be integrated with Jenkins, Go to Server Url, and follow the below images.

![image](https://github.com/beeru405/webcalc-java/assets/101712802/4304fabf-9746-4063-9936-fe1111da690f)

![image](https://github.com/beeru405/webcalc-java/assets/101712802/2051540d-c225-4ca6-8665-f9c3053d4ae6)

![image](https://github.com/beeru405/webcalc-java/assets/101712802/32112bc7-49e7-4511-bb0a-1bd814354802)

Copy the Unique credential for the token and go the the jenkins server URL. In the Jenkins Server URL, In Dashboard click manage Jenkins-> credentials and click on global.

![image](https://github.com/beeru405/webcalc-java/assets/101712802/baf15221-3674-4b7f-a82c-be0578cf2a4b)

Click on the Add Credentials, and fill in the details of the token in the dialogue box.

![image](https://github.com/beeru405/webcalc-java/assets/101712802/deb792bc-3f4b-41f6-a76e-4c0bdf8ae527)


Choose kind as Secret Text and scope as Global. Put the copied unique code in the secret field, and add the ID and description to the token. Later this token will be used to authenticate to SonarQube Server.


![image](https://github.com/beeru405/webcalc-java/assets/101712802/1d78c494-e893-4807-9335-72bc30ba5b33)

Click on Create to create the Token.

![image](https://github.com/beeru405/webcalc-java/assets/101712802/62662133-e251-4b20-a544-f9ceaebb94c2)


Now set up the SonarQube tool in the ManageJenkins -> system section and provide the SonarQube Url and Authentication token ID that we just created in jenkins Global Credentials.

![image](https://github.com/beeru405/webcalc-java/assets/101712802/23650527-097e-4838-8cc4-8497f7b7169c)

Set up the SonarQube scanner plugin installation in the Global Tools settings for easy SonarQube integration with Jenkins.

![image](https://github.com/beeru405/webcalc-java/assets/101712802/9709352a-fde9-4533-ab8d-d510125763a5)

Set up the Webhook connection with SonarQube.In Administration -> Configuration -> WebHooks.Webhooks in Sonarqube are used to tell the third Party(Jenkins) when the code analysis is complete.

```
http://<JENKINS-IP:8090>/sonarqube-webhook
```

![image](https://github.com/beeru405/webcalc-java/assets/101712802/348af651-5a1d-4b8e-b465-402ebb2a7bd2)


![image](https://github.com/beeru405/webcalc-java/assets/101712802/74a29ecb-0fce-45aa-a9e5-5e93c46aafab)


![image](https://github.com/beeru405/webcalc-java/assets/101712802/f9b5e5b9-ff1c-4742-8ad5-af550fd1e0b5)


![Screenshot from 2024-03-11 14-37-54](https://github.com/beeru405/webcalc-java/assets/101712802/c581d32c-eaa9-43d0-8451-7f5b11ab4683)

After the SonarQube Pipeline is successful you can see the SonarQube section in the left panel. Click on it to see the full report.



![image](https://github.com/beeru405/webcalc-java/assets/101712802/efbe818c-9d32-441e-a114-2ed0fcf8a0ba)




#### changes you need to do in Github
1. calculator.java - Give your MySQL details
2. Jenkinsfile - Github link,Tomcat details

##### Reference
GitHub Link: ```https://github.com/kishore3k/webcalculator/tree/master```
