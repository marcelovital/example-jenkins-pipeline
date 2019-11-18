appName = "node-app-dev"
projName = "mvitalbr-dev"

node {

	stage('Set Variable') {
        openshift.withCluster() {
        	openshift.withProject(projName) {
		        openshift.set("env", "dc/${appName}", "NOMEARQUIVO='${NOMEARQUIVO}'")
            }
        }
    }

    stage('Set Replicas') { 
        openshift.withCluster() {
        	openshift.withProject(projName) {
		        def dccontab = openshift.Selector("dc", "${appName}")
                dccontab.scale("--replicas=3")                
            }
        }
    }






    stage('UnSet Variable') {
        openshift.withCluster() {
        	openshift.withProject(projName) {
		        openshift.set("env", "dc/${appName}v", "NOMEARQUIVO=''")
            }
        }
    }

    stage('UnSet Replicas') { 
        openshift.withCluster() {
        	openshift.withProject(projName) {
		        def dccontab = openshift.Selector("dc", "${appName}")
                dccontab.scale("--replicas=1")                
            }
        }
    }



}