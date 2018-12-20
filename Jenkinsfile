node('master') {
    
cleanWs()
   
stage('Download kubectl'){
    sh '''curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl && curl -LO https://k8s.io/examples/application/mysql/mysql-deployment.yaml && tac mysql-deployment.yaml | sed "1,7d"|tac > mysql.yaml'''

}

stage('Deploy MySQL'){
    sh '''./kubectl create -f mysql.yaml'''
}

//stage('Instana Agent'){
//    sh '''curl -o setup_agent.sh https://setup.instana.io/agent && chmod 700 ./setup_agent.sh && sudo ./setup_agent.sh -a ${INSTANA_AGENT_KEY} -t dynamic -l us'''
//}

if (params.DELETE_DEPLOY == true) {
	sh '''./kubectl delete service mysql && ./kubectl delete deployment mysql'''
    echo 'MySQL kube deployment is deleted !!!'
}
    
}
