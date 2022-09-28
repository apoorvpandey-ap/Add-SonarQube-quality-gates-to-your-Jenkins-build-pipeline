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
#
### Step 2. Go http://localhost:8080
Jenkins appears and its needs a password for login so go to `cat /var/jenkins_home?secrets/initailpassword`
![image](https://user-images.githubusercontent.com/66588814/191177812-64ce5182-8ede-4e41-bc72-500ac7c7e8cd.png)
#
### Step 3. Plugin` pipeline` and` git` plugin into Jenkins
![image](https://user-images.githubusercontent.com/66588814/191178183-c070f78a-fe80-49e1-a6e7-8adf71d1f27c.png)


Once complete head over to Manage Jenkins > Manage Plugins > Available and search for sonar. Select the SonarQube Scanner plugin and click Install without restart.

![image](https://user-images.githubusercontent.com/66588814/191178371-eb9cae1a-0068-4243-9a76-ad69ebafee91.png)
#
### Step 4. 
Go to **Manage Jenkins > Configure System** and scroll down to the **SonarQube servers** section.

Click the **Add SonarQube button**. Now add a Name for the server, such as SonarQube. The Server URL will be **http://sonarqube:9000**.

![image](https://user-images.githubusercontent.com/66588814/192819404-0ec006ed-ce85-4dac-a4f6-7e628e7a0974.png)
#
### Step 5. Now go to **Administration > Configuration > Webhooks**. 

Let’s jump over to [SonarQube]**(http://localhost:9000/)**. Click Login at the top-right of the page, and log in with the default **credentials** of **admin/admin**. You’ll then have to set a new password.

Click **Create**, and in the popup that appears give the webhook a name of Jenkins, set the URL to **http://jenkins:8080/sonarqube-webhook**, and click Create.
![image](https://user-images.githubusercontent.com/66588814/192821983-b4586b93-77cc-4bc1-aa07-2ddc34e269cc.png)

**Create**
![image](https://user-images.githubusercontent.com/66588814/192822147-f06245a7-f90a-4110-aea1-c08e78a0ef0b.png)
#
### Step 6. Adding a quality gate
SonarQube comes with its own Sonar way quality gate enabled by default. If you click on **Quality Gates** you can see the details of this.

![image](https://user-images.githubusercontent.com/66588814/192823438-c7f58ade-8c9f-49d6-8345-3f105a5a73c4.png)

It’s all about making sure that **new code** is of high quality. In this example we want to check the quality of **existing code**, so we need to create a new **quality gate**.

Click **Create**, then give the quality gate a name. I’ve called mine _Tom Way_ 😉

![image](https://user-images.githubusercontent.com/66588814/192825237-a614a7e9-6e77-4f83-b0da-439e1cc11c67.png)


Click **Save** then on the next screen click **Add Condition**. Select On **Overall Code**. Search for the metric **Maintainability Rating** and choose worse than A. This means that if existing code is not maintainable then the quality gate will fail. Click **Add Condition** to save the condition.

![image](https://user-images.githubusercontent.com/66588814/192825741-907c0e86-395e-46e1-84e6-91efccb0e02f.png)


Finally, click **Set as Default** at the top of the page to make sure that this quality gate will apply to any new code analysis.

![image](https://user-images.githubusercontent.com/66588814/192825893-7ed95ea3-d96d-43f7-9245-b9de706655b2.png)
#
### Step 7 Go back to Jenkins and create Pipelines 
The last thing to do is set up two Jenkins pipelines:

1. A pipeline that runs against a code project over at the [sonarqube-jacoco-code-coverage GitHub repository](https://github.com/tkgregory/sonarqube-jacoco-code-coverage.git). The code here is decent enough that the pipeline should pass.

2. A pipeline that runs against the same project, but uses the [bad-code branch](https://github.com/tkgregory/sonarqube-jacoco-code-coverage/tree/bad-code). The code here is so bad that the pipeline should fail.

**Good code pipeline**
Back in [Jenkins ](http://localhost:8080/)click **New Item** and give it a name of sonarqube-good-code, select the **Pipeline** job type, then click **OK**.

Scroll down to the **Pipeline** section of the configuration page and enter the following declarative pipeline script in the **Script** textbox

```
pipeline {
    agent any
    stages {
        stage('Clone sources') {
            steps {
                git url: 'https://github.com/tkgregory/sonarqube-jacoco-code-coverage.git'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "./gradlew sonarqube"
                }
            }
        }
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
    }
}
```

The script has three stages:

1. In the **Clone sources** stage code is cloned from the GitHub repository mentioned earlier
 
2. In the **SonarQube analysi**s stage we use the withSonarQubeEnv('Sonarqube') method exposed by the plugin to wrap the Gradle build of the code repository. This provides all the configuration required for the build to know where to find SonarQube. Note that the project build itself must have a way of running SonarQube analysis, which in this case is done by running ./gradlew sonarqube. For more information about running SonarQube analysis in a Gradle build see [this article](https://tomgregory.com/how-to-measure-code-coverage-using-sonarqube-and-jacoco/)
3. In the **Quality gate** stage we use the waitForQualityGate method exposed by the plugin to wait until the SonarQube server has called the Jenkins webhook. The about pipeline flag means if the SonarQube analysis result is a failure, we abort the pipeline.
Click **Save** to save the pipeline.

**Bad code pipeline**

Create another pipeline in the same way, but name it sonarqube-bad-code. The pipeline script is almost exactly the same, except this time we need to check out the bad-code branch of the same repository.

```
pipeline {
    agent any
    stages {
        stage('Clone sources') {
            steps {
                git branch: 'bad-code', url: 'https://github.com/tkgregory/sonarqube-jacoco-code-coverage.git'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "./gradlew sonarqube"
                }
            }
        }
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
    }
}

```

Again, click **Save.**

![image](https://user-images.githubusercontent.com/66588814/192828023-d222da68-a208-4181-ab5f-d1d546cdb975.png)

Yes, that’s right, now it’s time to run our pipelines! ✅

Let’s run the **sonarqube-good-code** pipeline first.

You should get a build with all three stages passing.

Now let’s run the **sonarqube-bad-code pipeline**. Remember this is running against some really bad code!

You’ll be able to see that the Quality gate stage of the pipeline has failed. Exactly what we wanted, blocking any future progress of this pipeline.

In the build’s Console Output you’ll see the message ERROR: Pipeline aborted due to quality gate failure: ERROR which shows that the pipeline failed for the right reas












