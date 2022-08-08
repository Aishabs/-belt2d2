pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('aisha-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aisha-aws-secret-access-key')
        AWS_S3_BUCKET = "aisha-belt2d2-artifacts-123456"
        ARTIFACT_NAME = "hello-world.jar"
        AWS_EB_APP_NAME = "Aisha-application"
        AWS_EB_ENVIRONMENT = "Aishaapplication-env"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        SONAR_IP = "52.23.193.18"
        SONAR_TOKEN = "sqp_04bf54cee0db534df076f1db8c993531d9d24e5d"
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
            post{
                always{
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }

        stage('Quality Scan'){
            steps {
                sh '''
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=online-Aisha_AlssubaieE-B2D2 \
                    -Dsonar.host.url=http://$SONAR_IP \
                    -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }
        stage('Package') {
            steps {
                sh "mvn package"
            }
            post{
                success{
                    archiveArtifacts artifacts: '**/target/**.jar', followSymlinks: false
                }
            }
        }
        stage('Publish artifacts to S3 Bucket') {
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