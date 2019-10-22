def gitCommit() {
        sh "git rev-parse HEAD > GIT_COMMIT"
        def gitCommit = readFile('GIT_COMMIT').trim()
        sh "rm -f GIT_COMMIT"
        return gitCommit
    }

def answerQuestion = ''

    node {
        // Checkout source code from Git
        stage 'Checkout'
        checkout scm

	try {
          stage('Deploy') {
                withCredentials(
                [[
                  $class: 'UsernamePasswordMultiBinding',
                  credentialsId: 'ocp',
                  passwordVariable: 'OCP_PASSWORD',
                  usernameVariable: 'OCP_USERNAME'
                ]]
           ){
                sh "oc login https://api.aahk.aahk-hpe.com:6443 -u kubeadmin -p ${env.OCP_PASSWORD}"
                sh "oc new-project eap"
                sh "oc new-build registry.access.redhat.com/jboss-eap-7/eap72-openshift:1.0-13 --binary=true --name=esar"
                sh "oc start-build esar --from-file=ESAREAR.ear --wait"
                sh "oc new-app esar"
                sh "oc4 create route edge --service=esar --insecure-policy=Redirect --hostname=esar-eap.apps.aahk.aahk-hpe.com"}}
		currentBuild.result = 'SUCCESS'
		return
        }
          catch(e) {
                     build_ok = false
                     echo e.toString()
	}


        // Push the new war file to the project
        stage 'Push'
	withCredentials(
                [[
                  $class: 'UsernamePasswordMultiBinding',
                  credentialsId: 'ocp',
                  passwordVariable: 'OCP_PASSWORD',
                  usernameVariable: 'OCP_USERNAME'
                ]]
           ){
        sh "oc login https://api.aahk.aahk-hpe.com:6443 -u kubeadmin -p ${env.OCP_PASSWORD}"
        sh "oc project eap"
        sh "oc start-build esar --from-file=ESAREAR.ear --wait"}
    }

// remember to copy the dtr cert in /etc/pki/ca-trust/source/anchors
// run update-ca-trust
// run systemctl restart atomic-openshift-*
