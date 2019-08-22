pipeline {
 environment {
    dockerhub_repo = 'himanshuchourasia/train-schedule'
    dockerhub_credential = 'dockerhubcreds' 
 }

    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build and push  docker image') {
           when {
               branch 'master'
               
           }
           steps{
         	script {
    			def myImage = docker.build("${dockerhub_repo}:${env.BUILD_ID}")
    			docker.withRegistry('', dockerhub_credential) {
    			myImage.push()
    			myImage.push('latest')
    		
     }
    			
}     
               
           }
       }    
           
           stage('deploy container to production'){
             when {
                 branch 'master'
                 
             }

             steps{
                withCredentials([usernamePassword(credentialsId: 'production_server', passwordVariable: 'USERPASS', usernameVariable: 'USERNAME')]) {
 				input 'Deploy container to production ?'
                milestone label:'container ready for production', ordinal:1
                sh "sshpass -p ${USERPASS} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${env.Production_IP} \" sudo docker pull ${dockerhub_repo}\""
 				script{
 				    node {
						try {

 				    		sh "sudo docker container stop train-schedule"
 				    		sh "sudo docker container rm train-schedule"
 				   }
 					catch (Exception e ){
 				         echo 'Cannot remove container see details' + e.toString()  				                     
 				  }
								 				        
 				    }

 				}

 				  sh "sudo docker container run --restart always -p 8080:8080 -d --name train-schedule"

		
 			          
            }
         }                        
       }
     }
   }