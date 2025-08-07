pipeline {
    agent any

    environment {
        BACKEND_DIR = 'crud_backend/crud_backend-main'
        FRONTEND_DIR = 'crud_frontend/crud_frontend-main'

        TOMCAT_URL = 'http://35.179.160.11:9090/manager/text'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'admin'

        BACKEND_WAR = "${env.BACKEND_DIR}/springapp1.war"
        FRONTEND_WAR = "${env.FRONTEND_DIR}/frontapp1.war"
        
    }

    stages {
        stage('Clone Repositories') {
            steps {
                git url: 'https://github.com/veeru2005/frontendapp1.git', branch: 'main'
            }
        }

        stage('Build React Frontend') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    sh '''
                        export PATH=$NODE_HOME:$PATH
                        npm install
                        npm run build
                    '''
                }
            }
        }

        stage('Package React as WAR') {
            steps {
                script {
                    def warDir = "${env.FRONTEND_DIR}/war_content"
                    sh "rm -rf ${warDir}"
                    sh "mkdir -p ${warDir}/META-INF ${warDir}/WEB-INF"
                    sh "cp -r ${env.FRONTEND_DIR}/build/* ${warDir}/"
                    writeFile file: "${warDir}/WEB-INF/web.xml", text: '''
                        <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="3.1">
                            <display-name>ReactApp</display-name>
                        </web-app>
                    '''
                    sh "cd ${warDir} && jar -cvf ../frontapp1.war ."
                }
            }
        }

        stage('Build Spring Boot App') {
            steps {
                dir("${env.BACKEND_DIR}") {
                    sh '''
                        mvn clean package
                        mv target/*.war springapp1.war
                    '''
                }
            }
        }

        stage('Deploy Spring Boot WAR') {
            steps {
                sh "curl -u ${TOMCAT_USER}:${TOMCAT_PASS} --upload-file \"${BACKEND_WAR}\" \"${TOMCAT_URL}/deploy?path=/springapp1&update=true\""
            }
        }

        stage('Deploy Frontend WAR') {
            steps {
                sh "curl -u ${TOMCAT_USER}:${TOMCAT_PASS} --upload-file \"${FRONTEND_WAR}\" \"${TOMCAT_URL}/deploy?path=/frontapp1&update=true\""
            }
        }
    }

    post {
        success {
            echo "✅ Backend deployed: http://35.179.160.11:9090/springapp1"
            echo "✅ Frontend deployed: http://35.179.160.11:9090/frontapp1"
        }
        failure {
            echo "❌ Build or deployment failed"
        }
    }
}
