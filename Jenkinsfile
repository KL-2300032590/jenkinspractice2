pipeline {
    agent any

    environment {
        PATH = "/opt/homebrew/bin:/Users/pardhasaradhireddy/maven/bin:${env.PATH}"
        TOMCAT_HOME = "/Users/pardhasaradhireddy/apache-tomcat-10.1.43"
    }

    stages {

        // ===== FRONTEND BUILD =====
        stage('Build Frontend') {
            steps {
                dir('reactfrontend') {
                    sh '''
                    npm install
                    npm run build
                    '''
                }
            }
        }

        // ===== FRONTEND DEPLOY =====
        stage('Deploy Frontend to Tomcat') {
            steps {
                sh '''
                FRONTEND_PATH="$TOMCAT_HOME/webapps/frontendjenkinsp"

                echo "Deploying React frontend to $FRONTEND_PATH"

                # Remove old frontend build
                rm -rf "$FRONTEND_PATH"
                mkdir -p "$FRONTEND_PATH"

                # Copy new build output
                if [ -d "reactfrontend/dist" ]; then
                    cp -R reactfrontend/dist/* "$FRONTEND_PATH/"
                elif [ -d "reactfrontend/build" ]; then
                    cp -R reactfrontend/build/* "$FRONTEND_PATH/"
                else
                    echo "❌ No frontend build folder found!"
                    exit 1
                fi
                '''
            }
        }

        // ===== BACKEND BUILD =====
        stage('Build Backend') {
            steps {
                dir('springbootbackend') {
                    sh '''
                    mvn clean package -DskipTests
                    '''
                }
            }
        }

        // ===== BACKEND DEPLOY =====
        stage('Deploy Backend to Tomcat') {
            steps {
                sh '''
                WEBAPPS_PATH="$TOMCAT_HOME/webapps"

                echo "Deploying Spring Boot WAR to Tomcat..."

                # Clean old deployment
                rm -f "$WEBAPPS_PATH/springprojects.war"
                rm -rf "$WEBAPPS_PATH/springprojects"

                # Copy the new WAR
                cp springbootbackend/target/springprojects.war "$WEBAPPS_PATH/"
                '''
            }
        }

        // ===== RESTART TOMCAT =====
      stage('Restart Tomcat') {
            steps {
                sh '''
                $TOMCAT_HOME/bin/shutdown.sh || true
                sleep 3
                $TOMCAT_HOME/bin/startup.sh
                '''
            }
      }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
        }
        failure {
            echo '❌ Pipeline Failed.'
        }
    }
}
