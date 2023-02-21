pipeline {
    // 스테이지 별로 다른 거
    // email 제거 버전
    agent any

    triggers {
        pollSCM('*/3 * * * *')
    }

    environment {
      AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
      AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
      AWS_DEFAULT_REGION = 'ap-northeast-2'
      HOME = '.' // Avoid npm root owned
      // PROD = 'PROD'
    }

    stages {
        // 레포지토리를 다운로드 받음
        stage('Prepare') {
            agent any
            
            steps {
                echo 'Clonning Repository'

                git url: 'https://github.com/seungunleeee/jenkinsdockerPLZ.git',
                    branch: 'master',
                    credentialsId: 'gittest'
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    echo 'Successfully Cloned Repository'
                }

                always {
                  echo "i tried..."
                }

                cleanup {
                  echo "after all other post condition"
                }
            }
        }

       
        
        // aws s3 에 파일을 올림
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // 프론트엔드 디렉토리의 정적파일들을 S3 에 올림, 이 전에 반드시 EC2 instance profile 을 등록해야함.
            dir ('./website'){
                sh '''
                aws s3 sync ./ s3://seungunleedockerjenkinstest
                '''
            }
          }

          post {
              // If Maven was able to run the tests, even if some of the test
              // failed, record the test results and archive the jar file.
              success {
                  echo 'Successfully Cloned Repository'

                  // mail  to: 'evan980825@gmail.com',
                  //       subject: "Deploy Frontend Success",
                  //       body: "Successfully deployed frontend!"

              }

              failure {
                  echo 'I failed :('

                  // mail  to: 'evan980825@gmail.com',
                  //       subject: "Failed Pipelinee",
                  //       body: "Something is wrong with deploy frontend"
              }
          }
        }
        
        stage('Lint Backend') {
            // Docker plugin and Docker Pipeline 두개를 깔아야 사용가능!
            agent {
              docker {
                image 'node:latest'
              }
            }
            
            steps {
              dir ('./server'){
                  sh '''
                  npm install&&
                  npm run lint
                  '''
              }
            }
        }
        
        stage('Test Backend') {
          agent {
            docker {
              image 'node:latest'
            }
          }
          steps {
            echo 'Test Backend'
            
            dir ('./server'){
                sh '''
                npm install
                npm run test
                '''
            }
          }
        }
        //-arg env=${PROD} 예는 나중에 어떻게 사용할지 해보자
        stage('Bulid Backend') {
          echo 'we are in!'
          agent {
             docker {
              image 'node:latest'
            }
          }
          echo 'we are out!'
          steps {
            echo 'Build Backend'
            echo 'Build Process immediate'
            dir ('./server'){
                sh """
                docker build . -t server 
                """
            }
            echo 'we get here'
          }

          post {
            failure {
              error 'This pipeline stops here...'
            }
          }
        }
        //실제 환경에선 ecs update
        //두번쨰 배포시   docker rm -f $(docker ps -aq)
        stage('Deploy Backend') {
          agent any

          steps {
            echo 'Deploy Backend'

            dir ('./server'){
                sh '''
              
                docker run -p 80:80 -d server
                '''
            }
          }

          post {
            success {
              mail  to: 'frontalnh@gmail.com',
                    subject: "Deploy Success",
                    body: "Successfully deployed!"
                  
            }
          }
        }
    }
}
