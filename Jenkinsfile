pipeline {
    agent any
    
    parameters{
	  string(name:'version',defaultValue:'1.0.0', description:'The version of artifact')
	  booleanParam(name:'promote',defaultValue: false, description:'Publish new version')
    }

    stages {
        stage('Clone'){
            steps {
                echo 'Cloning.. And setting up voulumes..'
                sh 'docker system prune --all --volumes -f'
                sh 'docker volume create vol-in'
                sh 'docker volume create vol-out'
                sh 'docker build -t cloner:latest . -f /var/jenkins_home/workspace/DevOpsPipeline3/docker-clone'
                sh 'docker run --mount source=vol-in,destination=/inputVol cloner:latest'
            }

        }

        stage('Build') {
            steps {
                echo 'Building..'
                sh 'docker build -t builder:latest . -f /var/jenkins_home/workspace/DevOpsPipeline3/docker-build'
                sh 'docker run -d -it -p 8889:8080 --network app_network --mount source=vol-in,destination=/inputVol --mount source=vol-out,destination=/outputVol builder:latest'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
		            sh 'docker build -t tester:latest . -f /var/jenkins_home/workspace/DevOpsPipeline3/docker-test'
                sh 'docker run --mount source=vol-in,destination=/inputVol --mount source=vol-out,destination=/outputVol tester:latest'
            }
        }
        stage('Deploy') {
            steps {
		            echo 'Deploying..'
                sh 'docker build -t deploy:latest . -f /var/jenkins_home/workspace/DevOpsPipeline2/backend/docker-deploy'
                sh 'docker run --name deploy-container --mount source=vol-out,destination=/outputVol deploy:latest'
                sh 'rm -rf artifact'
		            sh 'mkdir artifact'
		            sh 'docker cp deploy-container:outputVol/shopBastim.jar ./artifact'
            }
        }
        stage('Publish') {
	          steps {
	            	echo 'Publishing..'
                script{
                    if(params.promote){
                        sh "mv ./artifact/shopBastim.jar ./artifact/shopBastim-${params.version}.jar"
                        archiveArtifacts artifacts: "artifact/shopBastim-${params.version}.jar"
                    }
                    else{
                        echo 'Pipeline finished work sucessfully but new version wasn\'t published.'
                    }
                }
            }
        }
    }
}
