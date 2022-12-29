

library identifier: "pipeline-library@v1.5",
retriever: modernSCM(
  [
    $class: "GitSCMSource",
    remote: "https://github.com/redhat-cop/pipeline-library.git"
  ]
)

appName = "sre-java-buildconfig"

pipeline {
    agent { label "maven" }
    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }
      stage('preamble') {
        steps {
            script {
                openshift.withCluster() {
                    openshift.withProject() {
                        echo "Using project: ${openshift.project()}"
                    }
                }
            }
        }
    }



       stage('build') {
      steps {
        script {
            openshift.withCluster() {
                openshift.withProject() {
                  def builds = openshift.selector("bc", appName).startBuild("--from-dir=.")
                  builds.logs('-f')

                }
            }
        }
      }
    }
     //tagging the image to build number
     stage("TEST: Can tag image") {
       steps{
    tagImage([
            sourceImagePath: "shruti-poc-devops",
            sourceImageName: "springboot-app",
            sourceImageTag : "latest",
            toImagePath: "shruti-poc-devops",
            toImageName    : "springboot-app",
            toImageTag     : "${env.BUILD_NUMBER}"

    ])
       }
}
      // triggering another jenkins job 
      stage("Trigger ManifestUpdate"){
        steps{
          build job:'updatemanifest' , parameters: [string(name: 'DOCKERTAG',value: env.BUILD_NUMBER)]
        }
      }
//         stage("Docker Build") {
//             steps {
//                 // This uploads your application's source code and performs a binary build in OpenShift
//                 // This is a step defined in the shared library (see the top for the URL)
//                 // (Or you could invoke this step using 'oc' commands!)
//                 binaryBuild(buildConfigName: appName, buildFromPath: ".")
//                 buildAndTag(imageName: appName, imageNamespace: "shruti-poc-devops", imageVersion: "${env.BUILD_NUMBER}", registryFQDN: "https://kubernetes.default.svc", fromFilePath:".")

//             }
//         }


    }
}
