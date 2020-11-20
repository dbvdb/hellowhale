pipeline {

  agent any

  stages {

    stage('Checkout Source') {
      steps {
        git url:'https://github.com/dbvdb/hellowhale.git', branch:'master', credentialsId: 'gitlabonbital'
      }
    }
    
      stage("Build image") {
            steps {
                script {
                    myapp = docker.build("dbvdb/test:${env.BUILD_ID}")
                }
            }
        }
    
      stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

    stage("Delete Previous Deployment") {
        steps {
            script {
            sh '''
                for service in $(ssh root@192.168.10.10 "kubectl get svc | awk '!/NAME/{print $1}'");
                do
                    if [ $service = "hello-whale-svc" ];
                        then
                            ssh root@192.168.10.10 "kubectl delete svc hello-whale-svc";
                    fi
                done

                for deployment in $(ssh root@192.168.10.10 "kubectl get deployment | awk '!/NAME/{print $1}'");
                do
                    if [ $deployment = "hello-blue-whale" ];
                        then
                            ssh root@192.168.10.10 "kubectl delete deployment hello-blue-whale";
                    fi
                done
            '''
            }
        }
    }
    stage('Deploy App') {
      steps {
        script {
          kubernetesDeploy(configs: "hellowhale.yml", kubeconfigId: "mykubeconfig")
        }
      }
    }

  }

}
