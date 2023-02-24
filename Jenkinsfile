def mainDir="."
def ecrLoginHelper="docker-credential-ecr-login"
def region="ap-northeast-2"  //서울은 2
// def ecrUrl="598552988151.dkr.ecr.ap-northeast-1.amazonaws.com"  // 원장님꺼
def ecrUrl="134977872812.dkr.ecr.ap-northeast-2.amazonaws.com"  //본인 ecr url 확인
def repository="board0222"  // 리포지토리명 넣기
// def deployHost="54.168.148.170"  // 원장님꺼임
def deployHost="13.209.65.55"  // EC2 퍼블릭 ip 넣기

pipeline {
    agent any

    stages {
        stage('Pull Codes from Github'){
            steps{
                checkout scm
            }
        }
        stage('Build Codes by Gradle') {
            steps {
              sh "./gradlew clean build"
            }
        }
                stage('Build Docker Image by Jib & Push to AWS ECR Repository') {
                    steps {
                        withAWS(region:"${region}", credentials:"aws-key") { //젠킨스 크레덴셜 ID와 동일하게
                            ecrLogin()
                            sh """
                                curl -O https://amazon-ecr-credential-helper-releases.s3.us-east-2.amazonaws.com/0.4.0/linux-amd64/${ecrLoginHelper}

                                cd ${mainDir}
                                ./gradlew jib -Djib.to.image=${ecrUrl}/${repository}:${currentBuild.number} -Djib.console='plain'
                            """
                        }
                    }
                }
                 stage('Deploy to AWS EC2 VM'){
                            steps{
                                sshagent(credentials : ["deploy-key"]) {
                                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${deployHost} \
                                     'aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${ecrUrl}/${repository}; \
                                      docker run -d -p 80:8888 -t ${ecrUrl}/${repository}:${currentBuild.number};'"
                                }
                            }
                        }

    }
}
