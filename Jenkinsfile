templateName = "node-app";

genericAppName = "node-app";
genericProjectName = "vital-app";

devProjectName = "$genericProjectName-dev";
devAppName= "$genericAppName-dev";

hmlProjectName = "$genericProjectName-hml";
hmlAppName= "$genericAppName-hml";

prodProjectName = "$genericProjectName-prod";
prodAppName= "$genericAppName-prod";

builderImage = 'registry.access.redhat.com/rhscl/nodejs-8-rhel7';
gitUrl = 'https://gitlab.com/openshift-samples/node-example.git';

node {

	stage('Build Configuration') {

		def hasBc = false;

		openshift.withCluster() {
			openshift.withProject(devProjectName) {
				hasBc = openshift.selector("bc", devAppName).exists();
			}
		}

		if(hasBc) {
			echo "Starting Build"
			openshift.withCluster() {
				openshift.withProject(devProjectName) {
					openshift.selector("bc", devAppName).startBuild().logs("-f")
				}
   			}
		} else {
		   echo "Creating Build Configuration"
		   openshift.withCluster() {
			   openshift.withProject(devProjectName) {
				   def bc = openshift.newBuild("--name=$devAppName", "$builderImage~$gitUrl", "-l template=$templateName", "--image-stream=$genericAppName").narrow("bc")
				   sleep 5
				   bc.logs("-f")
				}
			}
		}
	}

	stage('Promote to DEV') {
		openshift.withCluster() {
			openshift.withProject(devProjectName) {
				openshift.tag("$genericAppName:latest", "$genericAppName:dev")
			}
		}
	}

	stage('Create DEV') {

		def hasApp = false;

		openshift.withCluster() {
			openshift.withProject(devProjectName) {
				hasApp = openshift.selector('dc', "$devAppName").exists()
			}
		}
	
		if(!hasApp) {
			openshift.withCluster() {
				openshift.withProject(devProjectName) {
					openshift.newApp("$genericAppName:dev", "--name=$devAppName", "-l template=$templateName").narrow('svc').expose()
				}
			}
		}
	}

	stage('Approval DEV-PROD'){
		timeout(time:3, unit:'HOURS') {
			input message:'Approve deployment from DEV to PROD?'
		}
	}

	stage('Promote to PROD') {
		openshift.withCluster() {
			openshift.withProject(devProjectName) {
				openshift.tag("$genericAppName:latest", "$genericAppName:prod")
			}
		}
	}

	stage('Create PROD') {

		def hasApp = false

		openshift.withCluster() {
			openshift.withProject(prodProjectName) {
				hasApp = openshift.selector('dc', "$prodAppName").exists()
			}
		}

		if(!hasApp){
			openshift.withCluster() {
				openshift.withProject(prodProjectName) {
					openshift.newApp("$devProjectName/$genericAppName:prod", "--name=$prodAppName", "-l template=$templateName").narrow('svc').expose()
				}
			}
		}
	}
}