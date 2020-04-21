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
    stage('Test') {
      steps {
        sh "${mvnCmd} test"
        step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
      }
    }

    // Add Lab 5 Here
    stage('Code Analysis') {
      steps {
        script {
          sh "${mvnCmd} sonar:sonar -Dsonar.host.url=http://sonarqube:9000 -DskipTests=true"
        }
      }
    }

    // Add Lab 6 Here
    stage('Archive App') {
      steps {
        sh "${mvnCmd} deploy -DskipTests=true -P nexus3"
      }
    }

    // Add Lab 7 Here
    stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject("dev-student1") {
              return !openshift.selector("bc", "tasks").exists();
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("dev-student1") {
              openshift.newBuild("--name=tasks", "--image-stream=jboss-eap72-openshift:1.1", "--binary=true")
            }
          }
        }
      }
    }
    // Add Lab 8 Here
    stage('Build Image') {
      steps {
        sh "rm -rf oc-build && mkdir -p oc-build/deployments"
        sh "cp target/openshift-tasks.war oc-build/deployments/ROOT.war"

        script {
          openshift.withCluster() {
            openshift.withProject("dev-student1") {
              openshift.selector("bc", "tasks").startBuild("--from-dir=oc-build", "--wait=true")
            }
          }
        }
      }
    }

    // Add Lab 11 Here
    stage('Clair Container Vulnerability Scan') {
      steps {
            container("skopeo") {
                sh  '''
                  export KUBECONFIG=/tmp/kubeconfig
                  oc login -u student1 -p workshop1! --insecure-skip-tls-verify https://api.ocpws.kee.vizuri.com:6443 2>&1
                  skopeo --debug copy --src-creds="$(oc whoami)":"$(oc whoami -t)" --src-tls-verify=false --dest-tls-verify=false --dest-creds=student1:workshop1! docker://image-registry.openshift-image-registry.svc:5000/dev-student1/tasks:latest docker://quay.apps.ocpws.kee.vizuri.com/student1/tasks:1.0
                '''
            }
        }
    }
https://quay.apps.ocpws.kee.vizuri.com
    // Add Lab 9a Here
    stage('Create DEV') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject("dev-student1") {
              return !openshift.selector('dc', 'tasks').exists()
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("dev-student1") {
              def app = openshift.newApp("tasks:latest")
              app.narrow("svc").expose();

              def dc = openshift.selector("dc", "tasks")
              while (dc.object().spec.replicas != dc.object().status.availableReplicas) {
                  sleep 10
              }
              openshift.set("triggers", "dc/tasks", "--manual")
            }
          }
        }
      }
    }
    // Add Lab 9b Here
    stage('Deploy DEV') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("dev-student1") {
              openshift.selector("dc", "tasks").rollout().latest();
            }
          }
        }
      }
    }
    // Add Lab 10a Here
    stage('Promote to STAGE?') {
      steps {
        timeout(time:15, unit:'MINUTES') {
            input message: "Promote to STAGE?", ok: "Promote"
        }

        script {
          openshift.withCluster() {
            openshift.tag("dev-student1/tasks:latest", "stage-student1/tasks:${version}")
          }
        }
      }
    }
    // Add Lib 10b Here
    stage('Deploy STAGE') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("stage-student1") {
              if (openshift.selector('dc', 'tasks').exists()) {
                openshift.selector('dc', 'tasks').delete()
                openshift.selector('svc', 'tasks').delete()
                openshift.selector('route', 'tasks').delete()
              }

              openshift.newApp("tasks:${version}").narrow("svc").expose()
            }
          }
        }
      }
    }
  }
}
