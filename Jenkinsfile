pipeline {
    agent { 
        label 'yalu'
    }
    tools {
        jdk 'jdk17-agent'
        gradle 'gradle8'
    }
    environment {
        WORK_SPACE = "/home/yalu/Desktop/agent/workspace"
        BASE_URL = "http://20.249.0.81"
        JAVA_HOME = "/home/yalu/agent/tools/hudson.model.JDK/jdk17-agent" // JDK가 설치된 경로에 맞게 설정
        PATH = "$JAVA_HOME/bin:$PATH"
        ACCESS_TOKEN = ""
    }



    stages {
        // stage('Test Backend') {
        //     steps {
        //         script {
        //             try {
        //                 sh 'curl --output /dev/null --silent --head --fail $BASE_URL:8090'
        //                 echo 'Test Running'
        //                 sh 'cd $WORK_SPACE/docker-flighter/flighter-backend && ./gradlew test'
        //             } catch (Exception e) {
        //                 echo 'I can\'t test because the application is not running'
        //             }
        //         }
        //     }
        // 


        stage('PreBuild') {
            steps {
                sh 'docker compose down'
                sh 'docker rmi -f docker-flighter-postgres'
                sh 'docker rmi -f docker-flighter-frontend'
                sh 'docker rmi -f docker-flighter-backend'
            }
        }


        stage('Build') {
            steps {
                sh 'cd $WORK_SPACE/docker-flighter/flighter-backend && ./gradlew clean build -x test'
            }
        }

        stage('Deploy') {
            steps {
              script {
                    def deploy = {
                        sh '''
                        cd $WORK_SPACE/docker-flighter && docker compose up -d
                        envFile=$(cat .env)
                        email=$(echo "$envFile" | grep '^ADMIN_EMAIL=' | cut -d= -f2-)
                        password=$(echo "$envFile" | grep '^ADMIN_PASSWORD=' | cut -d= -f2-)

                        while true; do
                            ACCESS_TOKEN=$(curl -s -X POST -H "Content-Type: application/json" -d '{ "email": "'${email}'", "password": "'${password}'" }' ${BASE_URL}:8090/api/login 2>/dev/null | jq -r '.accessToken')
                            if [ -n "$ACCESS_TOKEN" ]; then
                                echo "Successfully received the JWT token."
                                break
                            fi
                            printf '.'
                            sleep 5
                        done
                        '''
                        // sh "curl -X POST -H 'Authorization:Bearer ${token}' ${serverUrl}/deploy"
                    }
                    
                    try {
                        deploy()
                    } catch (Exception e) {
                        retry(3) {
                            if (e.getMessage().contains("TLS handshake timeout")) {
                                echo "Caught TLS handshake timeout error. Retrying deploy..."
                                deploy()
                            } else {
                                echo 12
                                printf(e.getMessage())
                                echo 34
                            }
                        }
                    }
                }
            }
        }
    }
}
