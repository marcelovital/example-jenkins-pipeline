templateName = "node-app";

// The service account for the jenkins service, if running on a separate project, must have the edit role in
// all projects in the pipeline
//		This can be accomplished with the command:
//  		oc policy add-role-to-group edit system:serviceaccount:$jenkinsProjectName:jenkins -n $devProjectName
//  		oc policy add-role-to-group edit system:serviceaccount:$jenkinsProjectName:jenkins -n $hmlProjectName
//  		oc policy add-role-to-group edit system:serviceaccount:$jenkinsProjectName:jenkins -n $prodProjectName


genericAppName = "node-app";
genericProjectName = "mvitalbr-app";

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
				   def bc = openshift.newBuild("--name=$devAppName", "$builderImage~$gitUrl", "-l template=$templateName", "--to=$genericAppName").narrow("bc")
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

	stage('Approval DEV-HML'){
		timeout(time:3, unit:'HOURS') {
			input message:'Approve deployment from DEV to HML?'
		}
	}

	stage('Promote to HML') {
		openshift.withCluster() {
			openshift.withProject(devProjectName) {
				openshift.tag("$genericAppName:dev", "$genericAppName:hml")
			}
		}
	}

	stage('Create HML') {

		def hasApp = false

		openshift.withCluster() {
			openshift.withProject(hmlProjectName) {
				hasApp = openshift.selector('dc', "$hmlAppName").exists()
			}
		}

//	Creating the ImageStream on the prodProjectName depends on the serviceaccounts having the image-puller role.
//	This can be accomplished with the command:
//  	oc policy add-role-to-group system:image-puller system:serviceaccounts:$hmlProjectName -n $devProjectName

		if(!hasApp){
			openshift.withCluster() {
				openshift.withProject(hmlProjectName) {
					openshift.newApp("$devProjectName/$genericAppName:hml", "--name=$hmlAppName", "-l template=$templateName").narrow('svc').expose()
				}
			}
		}
	}

	stage('Approval HML-PROD'){
		timeout(time:3, unit:'HOURS') {
			input message:'Approve deployment from HML to PROD?'
		}
	}

	stage('Promote to PROD') {
		openshift.withCluster() {
			openshift.withProject(devProjectName) {
				openshift.tag("$genericAppName:hml", "$genericAppName:prod")
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

//	Creating the ImageStream on the prodProjectName depends on the serviceaccounts having the image-puller role.
//	This can be accomplished with the command:
//  	oc policy add-role-to-group system:image-puller system:serviceaccounts:$prodAppName -n $devProjectName

		if(!hasApp){
			openshift.withCluster() {
				openshift.withProject(prodProjectName) {
					openshift.newApp("$devProjectName/$genericAppName:prod", "--name=$prodAppName", "-l template=$templateName").narrow('svc').expose()
				}
			}
		}
	}
}
