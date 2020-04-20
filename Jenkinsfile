def version, mvnCmd = "mvn -s configuration/cicd-settings-nexus3.xml"

pipeline {
  agent {
    label 'maven'
  }
  stages {

    // Add Lab 3 Here
    stage('Build App') {
      steps {
        git branch: 'eap-7', url: 'http://gogs.apps.ocpws.kee.vizuri.com/student1/openshift-tasks.git'
        script {
            def pom = readMavenPom file: 'pom.xml'
            version = pom.version
        }
        sh "${mvnCmd} install -DskipTests=true"
      }
    }
 
    // Add Lab 4 Here

    // Add Lab 5 Here

    // Add Lab 6 Here

    // Add Lab 7 Here
  }
}
