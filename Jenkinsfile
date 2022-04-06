pipeline {
    agent any
    
   environment {
		PROJECT_ID = 'dockerorc'
        CLUSTER_NAME = 'gcpcluster-1'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'k8_Cluster'		
	}
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            
            steps {
                sh "sudo docker build -t timjar3/train-schedule:${env.BUILD_NUMBER} ."
                }
            }
        stage('Push Docker Image') {
          
            steps {
              sh "sudo docker push timjar3/train-schedule:${env.BUILD_NUMBER}" 
             }
        }
            
        stage('CanaryDeploy') {
            
            environment { 
                CANARY_REPLICAS = 1
            }
    
               steps{
			    echo "Deployment started ..."
			    sh 'ls -ltr'
			    sh 'pwd'
			    sh "sed -i 's/tagversion/${env.BUILD_NUMBER}/g' train-schedule-kube-canary.yml"
				sh "sed -i 's/tagversion/${env.BUILD_NUMBER}/g' train-schedule-kube.yml"
			    echo "Start deployment of train-schedule-kube-canary.yml"
			    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'train-schedule-kube-canary.yml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
				echo "Start deployment of train-schedule-kube.yml"
				step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'train-schedule-kube.yml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Deployment Finished ..."
         }
      }
  }
}
