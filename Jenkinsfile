appName = "node-app";
gitUrl = 'https://gitlab.com/openshift-samples/node-example.git';

node {

    stage('Build Configuration') {

        def hasBc = false;

        openshift.withCluster() {
          hasBc = openshift.selector("bc", appName).exists();
        }

       if(hasBc){
         echo "Starting Build"
         openshift.withCluster() {
           openshift.selector("bc", appName).startBuild().logs("-f")
         }
       }else{
             echo "Creating Build Configuration"
             openshift.withCluster() {
               def bc = openshift.newBuild("--name=$appName", "registry.access.redhat.com/rhscl/nodejs-8-rhel7~$gitUrl", "-l template=$appName").narrow("bc")
               sleep 5
               bc.logs("-f")
             }
       }
    }

    stage('Promote to DEV') {
      openshift.withCluster() {
        openshift.tag("$appName:latest", "$appName:dev")
      }
    }

    stage('Create DEV') {
      def hasApp = false
      openshift.withCluster() {
        hasApp = openshift.selector('dc', "$appName-dev").exists()
      }

      if(!hasApp){
        openshift.withCluster() {
          openshift.newApp("$appName:latest", "--name=$appName-dev", "-l template=$appName").narrow('svc').expose()
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
        openshift.tag("$appName:latest", "$appName:prod")
      }
    }

    stage('Create PROD') {
      def hasApp = false
      openshift.withCluster() {
        hasApp = openshift.selector('dc', "$appName-prod").exists()
      }

      if(!hasApp){
        openshift.withCluster() {
          openshift.newApp("$appName:prod", "--name=$appName-prod", "-l template=$appName").narrow('svc').expose()
        }
      }
    }

}
