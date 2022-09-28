https://tomgregory.com/sonarqube-quality-gates-in-jenkins-build-pipeline/

# Add-SonarQube-quality-gates-to-your-Jenkins-build-pipeline

### Flow Diagram

![image](https://user-images.githubusercontent.com/66588814/191176147-fc7da82d-2065-405d-a36a-9d6329ba377a.png)
#

### Step 1. Running Jenkins and SonarQube as docker so write one docker-compose 
```
version: "3"
services:
  sonarqube:
    image: sonarqube:lts
    ports:
      - 9000:9000
    networks:
      - mynetwork
    environment:
      - SONAR_FORCEAUTHENTICATION=false
  jenkins:
    image: jenkins/jenkins:2.319.1-jdk11
    ports:
      - 8080:8080
    networks:
      - mynetwork
networks:
  mynetwork:
```
### Step 2. Go http://localhost:8080
Jenkins appears and its needs a password for login so go to `cat /var/jenkins_home?secrets/initailpassword`
![image](https://user-images.githubusercontent.com/66588814/191177812-64ce5182-8ede-4e41-bc72-500ac7c7e8cd.png)
#
### Step 3. Plugin` pipeline` and` git` plugin into Jenkins
![image](https://user-images.githubusercontent.com/66588814/191178183-c070f78a-fe80-49e1-a6e7-8adf71d1f27c.png)


Once complete head over to Manage Jenkins > Manage Plugins > Available and search for sonar. Select the SonarQube Scanner plugin and click Install without restart.

![image](https://user-images.githubusercontent.com/66588814/191178371-eb9cae1a-0068-4243-9a76-ad69ebafee91.png)

