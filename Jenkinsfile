node {

	stage('Set Variable') {
        openshift.withCluster() {
        	openshift.withProject() {
		        openshift.set("env", "dc/node-app-dev", "NOMEARQUIVO='${NOMEARQUIVO}'")
            }
        }
    }

    stage('Set Replicas') { 
        openshift.withCluster() {
        	openshift.withProject() {
		        def dccontab = openshift.Selector("dc", "node-app-dev")
                dccontab.scale("--replicas=3")                
            }
        }
    }






    stage('UnSet Variable') {
        openshift.withCluster() {
        	openshift.withProject() {
		        openshift.set("env", "dc/node-app-dev", "NOMEARQUIVO=''")
            }
        }
    }

    stage('UnSet Replicas') { 
        openshift.withCluster() {
        	openshift.withProject() {
		        def dccontab = openshift.Selector("dc", "node-app-dev")
                dccontab.scale("--replicas=1")                
            }
        }
    }



}