templateName = "node-app";
appName= "node-app-vital"
projectName = "vital-app-safra";
gitUrl = 'https://gitlab.com/openshift-samples/node-example.git';

node {

    stage('Build Configuration') {

        def hasBc = false;

        openshift.withCluster() {
		openshift.withProject(projectName) {
	          hasBc = openshift.selector("bc", appName).exists();
        	}
	}

       if(hasBc){
         echo "Starting Build"
         openshift.withCluster() {
		openshift.withProject(projectName) {
	           openshift.selector("bc", appName).startBuild().logs("-f")
        	}
	 }
       }else{
             echo "Creating Build Configuration"
             openshift.withCluster() {
		openshift.withProject(projectName) {
               		def bc = openshift.newBuild("--name=$appName", "registry.access.redhat.com/rhscl/nodejs-8-rhel7~$gitUrl", "-l template=$templateName").narrow("bc")
               		sleep 5
               		bc.logs("-f")
		}
             }
       }
    }

    stage('Promote to DEV') {
      openshift.withCluster() {
	openshift.withProject(projectName) {
		openshift.tag("$appName:latest", "$appName:dev")
	}
      }
    }

    stage('Create DEV') {
      def hasApp = false
      openshift.withCluster() {
	openshift.withProject(projectName) {
        	hasApp = openshift.selector('dc', "$appName-dev").exists()
	}
      }

      if(!hasApp){
        openshift.withCluster() {
		openshift.withProject(projectName) {
			openshift.newApp("$appName:latest", "--name=$appName-dev", "-l template=$templateName").narrow('svc').expose()
		}
        }
      }
    }

    stage('Approval'){
      timeout(time:3, unit:'HOURS') {
          input message:'Approve deployment?'
      }
    }

    stage('Promote to PROD') {
      openshift.withCluster() {
	openshift.withProject(projectName) {
	        openshift.tag("$appName:latest", "$appName:prod")
	}
      }
    }

    stage('Create PROD') {
      def hasApp = false
      openshift.withCluster() {
	openshift.withProject(projectName) {
		hasApp = openshift.selector('dc', "$appName-prod").exists()
	}
      }

      if(!hasApp){
        openshift.withCluster() {
		openshift.withProject(projectName) {
          		openshift.newApp("$appName:prod", "--name=$appName-prod", "-l template=$templateName").narrow('svc').expose()
		}
        }
      }
    }

}
