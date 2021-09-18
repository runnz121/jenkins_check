pipeline {
    // 스테이지 별로 다른 거
    agent any

    //pipline을 얼마주기로 트리깅 할지 설정 )
    triggers {
        pollSCM('*/3 * * * *')
    }

    //aws 환경변수 입력 -> EC2에 배포시 
    //jenkins 크리덴셜에 입력한 id
    environment {
      AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
      AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
      AWS_DEFAULT_REGION = 'ap-northeast-2'
      HOME = '.' // Avoid npm root owned
    }

    //단계 설정
    //stage마다 설정 
    stages {
        // 레포지토리를 다운로드 받음
        stage('Prepare') {
            agent any
            //실행되는 단계
            steps {
                echo 'Clonning Repository'

                git url: 'https://github.com/runnz121/jenkins_check.git',
                    branch: 'main',
                    credentialsId: 'jenkins_credential_test' //jenkins 토큰 등록했던 ID 입력
            }
            //step 이 완료되면 추후실행되는 명령어
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

        stage('git_push'){
          steops {
            echo 'git push'

            
            git url: 'https://github.com/runnz121/jenkins_check.git',
                branch: 'main',
                credentialsId: 'jenkins_credential_test' 
            }

            post {
              success {
                echo 'Success git push'
              }
              always {
                echo "i tried..."
              }
              cleanup {
                echo "after all other post condition"
              }
            }
          }
        }
        
        // aws s3 에 파일을 올림
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // 프론트엔드 디렉토리의 정적파일들을 S3 에 올림, 이 전에 반드시 EC2 instance profile 을 등록해야함.
            // s3로 지정해놓은 버킷 이름을 입력 
            dir ('./website'){
                sh '''
                aws s3 sync ./ s3://jenkins-test-front-yong
                '''
            }
          }

          post {
              // If Maven was able to run the tests, even if some of the test
              // failed, record the test results and archive the jar file.
              success {
                  echo 'Successfully Cloned Repository'

                  mail  to: 'runnz121@gmail.com',
                        subject: "Deploy Frontend Success",
                        body: "Successfully deployed frontend!"

              }

              failure {
                  echo 'I failed :('

                  mail  to: 'runnz121@gmail.com',
                        subject: "Failed Pipelinee",
                        body: "Something is wrong with deploy frontend"
              }
          }
        }
        
        stage('Lint Backend') {
            // Docker plugin and Docker Pipeline 두개를 깔아야 사용가능!
            agent {
              docker {
                 // 이미지를 받아옴
                image 'node:latest'
              }
            }
            //lint option 
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
        
        stage('Bulid Backend') {
          agent any
          steps {
            echo 'Build Backend'
               // 도커를 직접 깔고 그 후 배포 
               // 도커로 빌드해서 디플로이 할 것 
               // server 라는 이름으로 이미지 빌드 함 
            dir ('./server'){
                sh """
                docker build . -t server --build-arg env=${PROD}
                """
            }
          }

          post {
            failure {
              error 'This pipeline stops here...'
            }
          }
        }
        
        stage('Deploy Backend') {
          agent any

          steps {
            echo 'Build Backend'
            //   docker rm -f $(docker ps -aq) -> 돌고있는 컨테이너가 있으면 모두 지워줌 첫 실행시에는 아무것도 없음으로 처음에만 뺴줌 나중에 다시 아래에 추가 
            // server라는 이름 붙여서 빌드한것임으로 Server라는 이미지를 실행시켜라는 듯 
            dir ('./server'){
                sh '''        
                docker run -p 80:80 -d server
                '''
            }
          }

          post {
            success {
              mail  to: 'runnz121@gmail.com',
                    subject: "Deploy Success",
                    body: "Successfully deployed!"
                  
            }
          }
        }
    }
