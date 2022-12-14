pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('abdullahkimrigh-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('abdullahkimrigh-aws-secret-access-key')

        AWS_S3_BUCKET = "abdullah-kimrigh-belt2day2-artifacts-123456"
        ARTIFACT_NAME = "hello-world.jar"
        AWS_EB_APP_NAME = "abdullah-kimrigh-java-maven-app"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "Abdullahkimrighjavamavenapp-env"
        
        SONAR_PROJECT_NAME = "online-Abdullah-Kimrigh-B2D2"
        SONAR_IP = "35.172.191.120"
        SONAR_TOKEN = "sqp_b00d1703c003b6a872448bd62b0ea75c9186f0e6"
    }

    stages {
        stage('Validate') {
            steps {
                sh "mvn validate"
                sh "mvn clean"
            }
        }

         stage('Build') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test"
            }

            post {
                always {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }

        stage('Quality Scan'){
            steps {
                sh '''
                mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=$SONAR_PROJECT_NAME \
                    -Dsonar.host.url=http://$SONAR_IP \
                    -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }

        stage('Package') {
            steps {                
                sh "mvn package"
            }

            post {
                success {
                    archiveArtifacts artifacts: '**/target/**.jar', followSymlinks: false                  
                }
            }
        }

        stage('Publish Artifacts') {
            steps {
                sh "aws configure set region us-east-1"
                sh "aws s3 cp ./target/**.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"               
            }
        }

        stage('Deploy') {
            steps {
                sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
                sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'                
            }
        } 
    }
}