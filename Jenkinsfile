pipeline {
  environment {
    //
    // "REGISTRY" isn't required if we're using docker hub, I'm leaving it here in case you want to use a different registry
    // REGISTRY = 'registry.hub.docker.com'
    //
    // you need a credential named 'docker-hub' with your DockerID/password to push images
    CREDENTIAL = "docker-hub"
    DOCKER_HUB = credentials("$CREDENTIAL")
    REPOSITORY = "${DOCKER_HUB_USR}/${JOB_BASE_NAME}"
    //TAG = "build-${BUILD_NUMBER}"
    TAG = "${BRANCH_NAME}"
    IMAGELINE = "${REPOSITORY}:${TAG} Dockerfile"
    BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
    //
    // we will need these if we're using anchore-cli
    // we'll need the anchore credential to pass the user
    // and password to anchore-cli so it can upload the results
    // ANCHORE_CREDENTIAL = "AnchoreJenkinsUser"
    // use credentials to set ANCHORE_USR and ANCHORE_PSW
    // ANCHORE = credentials("${ANCHORE_CREDENTIAL}")
    // api endpoint of your anchore instance
    // ANCHORE_URL = "http://anchore3-priv.novarese.net:8228/v1"
    //
  } // end environment 
  
  agent any
  
  stages {
    
    stage('Checkout SCM') {
      steps {
        checkout scm
      } // end steps
    } // end stage "checkout scm"
    
    stage('Install and Verify Tools') {
      steps {
        sh """
          which docker 
          curl -sSfL  https://anchorectl-releases.anchore.io/anchorectl/install.sh  | sh -s -- -b $HOME/.local/bin
          chmod 0755 $HOME/.local/bin/anchorectl
          export PATH="$HOME/.local/bin/:$PATH"
          """
      } // end steps
    } // end stage "Verify Tools"

       
    stage('Analyze Image') {
      steps {
        //   # if we want to use anchorectl instead you'll need three variables as secrets/credentials:
        //   # ANCHORECTL_URL         e.g. http://anchore.example.com:8228/
        //   # ANCHORECTL_USERNAME    
        //   # ANCHORECTL_PASSWORD
        //   #
        //   # and then do this:
        //
          sh """
            export ANCHORECTL_URL="http://api:8228/"
            export ANCHORECTL_USERNAME="admin"
            export ANCHORECTL_PASSWORD="foobar"
            /root/.local/bin/anchorectl -vv image add --wait --no-auto-subscribe busybox:latest
            """
        //
        // if you want continuous re-evaluation in the background, you can turn it on with these:
        //   anchore-cli subscription activate policy_eval ${REPOSITORY}:${TAG1}
        //   anchore-cli subscription activate vuln_update ${REPOSITORY}:${TAG1}
        // and in this case you would probably also want to configure "policy & vulnerability" updates
        // in "Events & Notifications" -> "Manage Notification Endpoints" 
        //
      } // end steps
    } // end stage "analyze image 1 with anchore plugin"     
  } // end stages
} // end pipeline 
