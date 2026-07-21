pipeline {
  agent any
  tools { 
        maven 'Maven_3_8_4'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=asgbuggywebapp07_asgbuggywebapp -Dsonar.organization=asgbuggywebapp07 -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=6cf85169973ff3bd51a6fdf21d9a81be089ff77c'
			}
    }
	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'Snyk_token', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }
	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("asg")
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('043272858830.dkr.ecr.us-east-1.amazonaws.com/asg', 'ecr:us-east-1:aws-credentials') {
                    app.push("latest")                                                                        
                    }
                }
            }
    	}
	   stage('Kubernetes Deployment of ASG Bugg Web Application') {
            steps {
                withAWS(credentials: 'aws-credentials', region: 'us-east-1') {

      // Generate fresh kubeconfig that contains a valid IAM token
            

            sh 'kubectl get nodes'
            sh 'kubectl delete all --all -n devsecops || true'
            sh 'kubectl apply -f deployment.yaml --namespace=devsecops'
    }
  }
}

	   
	stage ('wait_for_testing'){
	   steps {
		   sh 'pwd; sleep 100; echo "Application Has been deployed on K8S"'
	   	}
	   }
	   
	stage('RunDASTUsingZAP') {
    steps {
        withAWS(credentials: 'aws-credentials', region: 'us-east-1') {

            sh '''
            pkill -f zap || true

            aws eks update-kubeconfig \
              --name kubernetes-cluster \
              --region us-east-1

            kubectl get nodes
            kubectl get svc -n devsecops

            HOST=$(kubectl get svc asgbuggy \
                -n devsecops \
                -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

            echo "HOST=$HOST"

            zap.sh -cmd \
                -port 8090 \
                -cmd \
                -quickurl http://$HOST:8002 \
                -quickprogress \
                -quickout ${WORKSPACE}/zap_report.html
            '''

            archiveArtifacts 'zap_report.html'
        }
      }
    }
  }
}
