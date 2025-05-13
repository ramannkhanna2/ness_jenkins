# ness_jenkins

```



imp :
https://github.com/ramannkhanna2/ness_jenkins.git

my name is raman khanna ..

ci cd ... jenkins


jenkins : opensource tool : community based tool .....

systemtic design ..



sdlc
build a code  >>  test the code seleium , snarqube>>    artefact(.jar.war) nexus    >>  teraform to create those qa servers >>     deploymnt of qa >> prodn


jenkins : workflow : series of steps in steps : pipeline 

ci-cd pipeline 


ci -cdlievery , deployment


==================================================
https://www.jenkins.io/download/

2 servers/machines : master . agent

ec2 instance : mahcine /vm : t2.medium as the capacity ...

created a sg (all traffic) , keypair , 20 gb storage root volume ..

==================================


https://www.jenkins.io/doc/book/installing/linux/#debianubuntu


install java , than jenkins

root@master:~# history
    1  sudo apt update
    2  sudo apt install fontconfig openjdk-21-jre
    3  java -version
    4  openjdk version "21.0.3" 2024-04-16
    5  OpenJDK Runtime Environment (build 21.0.3+11-Debian-2)
    6  OpenJDK 64-Bit Server VM (build 21.0.3+11-Debian-2, mixed mode, sharing)
    7  java --version
    8  sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc   https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    9  echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]"   https://pkg.jenkins.io/debian-stable binary/ | sudo tee   /etc/apt/sources.list.d/jenkins.list > /dev/null
   10  sudo apt-get update
   11  sudo apt-get install jenkins
   12  systemctl status jenkins
   
   
   =====================
   
   -- browse using publicip:8080
   
   -- 

===================================

freestyle project : 


workflow :
    
mkdir /home/raman
touch /home/raman/test1
chmod 777 /home/raman/test1
4th step
5th step





complex :
mkdir /home/raman                            other operation


============================================================


ness 
  
)J{rG'4@
  
  https://685421549691.signin.aws.amazon.com/console 

=========================================

region : oregon 

======================================

1 master , 1 agent (multiple agent nodes)

ubuntu

ec2 instance : mahcine /vm : t2.micro as the capacity ...

created a sg (all traffic) (u can reuse it ) , keypair , 20 gb storage root volume ..

=======================================


https://www.jenkins.io/download/

-- jenkins installation on master not on agents ..


sudo -i


-- after jenkins installation , 

browse publicipMaster:8080


systemctl status jenkins...
--  adminstarror password 


-------------------------------------------------


vi /usr/lib/systemd/system/jenkins.service

#user, group = root

    2  systemctl daemon-reload
    3  systemctl restart jenkins
    
    ===================================================
    
    
on home screen of jenkins , setup agent ..


--login to agent ,,
 mkdir /home/jenkins

home dir of jenkins agent : /home/jenkins

-- only build jobs with labels : agent-1



--manage jenkins >> nodes 

-- create a remote root directory on slave to act as jenkins home directory
/home/jenkins 

-- CONNECT WITH JAVA AGENT :


nohup java -jar agent.jar -url http://18.231.162.71:8080/ \
-secret 90fed3e4a8924bac2dfc16a56a8a48d568aba454ae6fb6c763bb3781e7b22fb8 \
-name slave1 -webSocket -workDir "/home/jenkins" > agent.log 2>&1 &




nohup: Prevents the process from being killed when terminal closes
> agent.log: Redirects stdout and stderr to a log file
&: Runs the process in the background
✅ Now you can safely close the terminal without disconnecting the agent.


===================================================================


https://github.com/ramannkhanna2/DevOps-Jenkinsfile-Docker-Git.git


=======================================================================

install docker on agent :
    
    https://docs.docker.com/engine/install/ubuntu/
    
    

containerization :
    
    lightweight , faster ,,
    
    jenkins workflow ( jenkinsfile)
  dockerfile   >>  docker build    >> custom image (base image + ur configurations)     >>> container   >>> serve the application 




manuallly :
 
 docker ps
   20  docker images
   21  clear
   22  docker
   23  clear
   24  ls
   25  mkdir raman
   26  cd raman/
   27  ls
   28  vi dockerfile
   29  vi index.html
   30  ls
   31  docker build -t ramanimage:v1  .
   32  docker ps
   33  docker images
   34  clear
   35  docker images
   36  docker ps
   37  docker images
   38  docker run -d --name ramancon \\
   39  clear
   40  docker images
   41  docker run -d --name ramancon -p 8083:80 ramanimage:v1
   42  docker ps
   43  docker rm -f `docker ps -aq`
   44  docker rmi -f `docker images`
   45  cleqar
   46  clear
   47  docker images
   48  docker ps
   49  docker ps -a























pipeline {
    // Define the Jenkins agent (node) to run the pipeline
    agent { label 'agent-1' }

    // Define pipeline parameters that can be set before execution
    parameters {
        // String parameter for the application name (Docker image name)
        string(name: 'APP_NAME', defaultValue: 'myapp', description: 'Name of the Docker image to build')
        
        // Dropdown choice parameter for selecting the deployment environment
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'qa', 'prod'], description: 'Target deployment environment')
        
        // Boolean parameter to decide whether to clean up old Docker containers before deploying
        booleanParam(name: 'CLEANUP_OLD', defaultValue: true, description: 'Cleanup old Docker container before deployment')
        
        // String parameter for specifying the port on the host to expose the container
        string(name: 'PORT_ON_HOST', defaultValue: '8081', description: 'Host port to expose container on')
    }

    // Define environment variables that can be used throughout the pipeline
    environment {
        // BUILD_TS: Timestamp of the build, formatted as yyyyMMdd-HHmmss
        BUILD_TS = "${new Date().format('yyyyMMdd-HHmmss')}"
        
        // CONTAINER_NAME: Name of the Docker container
        CONTAINER_NAME = "container-${params.APP_NAME}"
        
        // HOST_PORT: Host port for container mapping, fetched from pipeline parameters
        HOST_PORT = "${params.PORT_ON_HOST}"
        
        // APP_PORT: Port inside the Docker container (fixed at 80 for this application)
        APP_PORT = '80'
    }

    // Pipeline-level options
    options {
        // Add timestamps to the console output
        timestamps()
        
        // Set a timeout for the pipeline execution of 5 minutes
        timeout(time: 5, unit: 'MINUTES')
        
        // Retry the pipeline 2 times if it fails
        retry(2)
    }

    // Define the stages of the pipeline
    stages {
        // Main stage for building and deploying the Docker container
        stage('Build & Deploy') {
            steps {
                script {
                    // Get the short Git commit ID and trim any whitespace
                    def commitId = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    
                    // Generate a Docker image tag that includes the application name, commit ID, build timestamp, and build number
                    def buildTag = "${params.APP_NAME}:${commitId}-${BUILD_TS}-b${env.BUILD_NUMBER}"
                    echo "Docker image tag to be used: ${buildTag}"

                    // If cleanup is enabled, check if an existing container with the same name is running
                    if (params.CLEANUP_OLD) {
                        sh """
                            # Check if the container is running
                            if docker ps -a --format '{{.Names}}' | grep -w ${CONTAINER_NAME}; then
                                echo "Stopping and removing existing container: ${CONTAINER_NAME}"
                                # Force remove the existing container
                                docker rm -f ${CONTAINER_NAME}
                            else
                                echo "No existing container to clean up."
                            fi
                        """
                    }

                    // Build the Docker image with the generated tag
                    sh "docker build -t ${buildTag} ."
                    # docker build -t ramanimage:v1  .

                    // Run a new Docker container with the specified name and port mapping
                    sh """
                        docker run -d --name ${CONTAINER_NAME} \\
                            -p ${HOST_PORT}:${APP_PORT} \\
                            ${buildTag}
                    """
                    # docker run -d --name ramancon \\
                           -p 8083:80 ramanimage:v1
                }
            }
        }
    }

    // Define post-build actions
    post {
        // When the pipeline succeeds
        success {
            echo "✅ Build & deployment completed successfully for ${params.APP_NAME}!"
        }

        // When the pipeline fails
        failure {
            echo "❌ Something went wrong during the build or deployment."
        }

        // Always execute after the pipeline completes, regardless of status
        always {
            echo "📦 Pipeline completed at: ${BUILD_TS}"
        }
    }
}


==========================================================================================================


rbac :
    
    manage jenkins >> users : create user
    
    to provide rbac >> manage jenkins >> security
    
      -- authorization >> project based matrix authaorization


=============================================================================

manual way to test the pipeline : 
    
    git clone https://github.com/ramannkhanna2/devops-maven-docker-jenkins.git
   35  ls
   36  cd devops-maven-docker-jenkins/
   37  ls
   38  clear
   39  ls
   40  rm -rf Jenkinsfile Jenkinsfile-devsecops JenkinsfileNexusAdded Jenkinsfile_Dynamic 
   41  ls
   42  mvn clean package
   43  ls
   44  cd target/

cat Dockerfile 
   53  ls
   54  docker build -t ramanappimage:v1 .
   55  apt  install docker.io
   56  ls
   57  docker build -t ramanappimage:v1 .
   58  docker images
   59  docker ps
   60  docker push ramanimage:v1
   61  docker push ramanappimage:v1
   62  docker login
   63  docker images
   64  docker push ramanappimage:v1
   65  docker tag ramanappimage:v1 ramann123/myimage:imageversion1
   66  docker images
   67  docker push ramann123/myimage:imageversion1
   68  clear
   69  docker images
   70  docker rmi -f `docker images`
   71  clear
   72  docker images
   73  docker run -d --name ramancon ramann123/myimage:imageversion1
   74  docker ps
   75  docker rmi -f `docker images`
   76  clear





in repo , dockerfile , jenkinsfile_dynamic 

    jenkins workflow ( jenkinsfile) : cicd :::
source code >>  build  (maven) >> artefact (war, jar) >>   dockerfile   >>  docker build    >> custom image (base image (tomcat) + ur configurations)    >>sending tha custime image to dockerhub  >>> container   >>> serve the application 
  
  
==================================================================


```
