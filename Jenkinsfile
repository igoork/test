pipeline {
    agent any
    parameters {
        booleanParam(defaultValue: false, description: 'Delete MySQL deployment at the end ?', name: 'DELETE_DEPLOY')
        string(defaultValue: "", description: 'Insert proper key', name: 'INSTANA_AGENT_KEY')
    }
    stages {
        stage('Download kubectl') {
        	steps {
        	    cleanWs()
        		sh '''curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x ./kubectl'''
        	}
        }
        stage ('Modify YAML') {
        	steps {
        	    script {
            		def filename = 'mysql-deployment.yaml'
            		def data = readYaml file: filename
            		
            		data[1].spec.template.spec.containers.first().remove('volumeMounts')
            		data[1].spec.template.spec.remove('volumes')
            		
            		sh "rm mysql-deployment.yaml"
            		writeYaml file: "mysql-service.yaml", data: data[0]
            		writeYaml file: "mysql-deployment.yaml", data: data[1]   
        	    }
        	}
        }
        stage ('Deploy MySQL') {
        	steps {
        		sh '''./kubectl create -f mysql-service.yaml && ./kubectl create -f mysql-deployment.yaml'''
        	}
        }
        stage ('Check kube deployment') {
        	steps {
        	    script {
        	        def service_status = sh(returnStatus: true, script: "./kubectl get services mysql")
        	        if (service_status != 0) {
        	            currentBuild.result = 'FAILED'
        	            error('Something went wrong…')
        	        }
        	        def deployment_status = sh(returnStatus: true, script: "./kubectl get deployments mysql")
        	        if (deployment_status != 0) {
        	            currentBuild.result = 'FAILED'
        	            error('Something went wrong…')
        	        }
        		}
        	}
        }
        stage ('Instana Agent') {
        	steps {
        		sh '''curl -o setup_agent.sh https://setup.instana.io/agent && chmod 700 ./setup_agent.sh && sudo ./setup_agent.sh -a ${INSTANA_AGENT_KEY} -t dynamic -l us'''
        	}
        }
        stage ('Delete MySQL deployment') {
            when {
                expression { params.DELETE_DEPLOY == true }
            }
        	steps {
        	    sh '''./kubectl delete services mysql && ./kubectl delete deployments mysql'''
        	}
        }            
    }
}
