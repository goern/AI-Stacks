// Openshift project
OPENSHIFT_SERVICE_ACCOUNT = 'jenkins'
DOCKER_REGISTRY = env.CI_DOCKER_REGISTRY ?: 'docker-registry.default.svc.cluster.local:5000'
CI_NAMESPACE= env.CI_PIPELINE_NAMESPACE ?: 'ai-coe'
CI_TEST_NAMESPACE = env.CI_AISTACKS_TEST_NAMESPACE ?: 'thoth-test-aistacks'

// github-organization-plugin jobs are named as 'org/repo/branch'
// we don't want to assume that the github-organization job is at the top-level
// instead we get the total number of tokens (size) 
// and work back from the branch level Pipeline job where this would actually be run
// Note: that branch job is at -1 because Java uses zero-based indexing
tokens = "${env.JOB_NAME}".tokenize('/')
org = tokens[tokens.size()-3]
repo = tokens[tokens.size()-2]
branch = tokens[tokens.size()-1]

echo "${org} ${repo} ${branch}"

// If this PR does not include an image change, then use this tag
STABLE_LABEL = branch
tagMap = [:]

// Initialize
tagMap['tensorflow-fedora27'] = STABLE_LABEL
tagMap['tensorflow-fedora27-test'] = STABLE_LABEL
tagMap['tensorflow-centos7-python3'] = STABLE_LABEL
tagMap['scikit-image-centos7-python3'] = STABLE_LABEL
tagMap['scikit-image-centos7-python2'] = STABLE_LABEL
tagMap['gcc630-centos7'] = STABLE_LABEL
tagMap['tensorflow-neural-style-s2i'] = STABLE_LABEL
tagMap['tensorflow-build-s2i'] = STABLE_LABEL
tagMap['tensorflow-serving-gpu-s2i'] = STABLE_LABEL
tagMap['jupyter-notebook-py35'] = STABLE_LABEL
tagMap['tf-base-notebook'] = STABLE_LABEL
tagMap['base-notebook'] = STABLE_LABEL

// IRC properties
IRC_NICK = "aicoe-bot"
IRC_CHANNEL = "#thoth-station"

properties(
    [
        buildDiscarder(logRotator(artifactDaysToKeepStr: '30', artifactNumToKeepStr: '', daysToKeepStr: '90', numToKeepStr: '')),
        disableConcurrentBuilds(),
    ]
)


library(identifier: "cico-pipeline-library@master",
        retriever: modernSCM([$class: 'GitSCMSource',
                              remote: "https://github.com/CentOS/cico-pipeline-library",
                              traits: [[$class: 'jenkins.plugins.git.traits.BranchDiscoveryTrait'],
                                       [$class: 'RefSpecsSCMSourceTrait',
                                        templates: [[value: '+refs/heads/*:refs/remotes/@{remote}/*']]]]])
                            )
library(identifier: "ci-pipeline@master",
        retriever: modernSCM([$class: 'GitSCMSource',
                              remote: "https://github.com/CentOS-PaaS-SIG/ci-pipeline",
                              traits: [[$class: 'jenkins.plugins.git.traits.BranchDiscoveryTrait'],
                                       [$class: 'RefSpecsSCMSourceTrait',
                                        templates: [[value: '+refs/heads/*:refs/remotes/@{remote}/*']]]]])
                            )
library(identifier: "ai-stacks-pipeline@master",
        retriever: modernSCM([$class: 'GitSCMSource',
                              remote: "https://github.com/goern/AI-Stacks-pipeline",
                              traits: [[$class: 'jenkins.plugins.git.traits.BranchDiscoveryTrait'],
                                       [$class: 'RefSpecsSCMSourceTrait',
                                        templates: [[value: '+refs/heads/*:refs/remotes/@{remote}/*']]]]])
                            )

pipeline {
    agent {
        kubernetes {
            cloud 'openshift'
            label 'ai-stacks'
            serviceAccount OPENSHIFT_SERVICE_ACCOUNT
            containerTemplate {
                name 'jnlp'
                args '${computer.jnlpmac} ${computer.name}'
                image DOCKER_REGISTRY + '/'+ CI_NAMESPACE +'/jenkins-aicoe-slave:' + STABLE_LABEL
                ttyEnabled false
                command ''
            }
        }
    }
    stages {
/*        stage("Get Image Puller Secret") {
            steps {
                node('master') {
                    sh 'oc extract --namespace=thoth-test-aistacks secret/deployer-dockercfg-vl9nl --to=-'
                }
            }
        } */
        stage("Get Changelog") {
            steps {
                node('master') {
                    script {
                        env.changeLogStr = pipelineUtils.getChangeLogFromCurrentBuild()
                        echo env.changeLogStr
                    }
                    writeFile file: 'changelog.txt', text: env.changeLogStr
                    archiveArtifacts allowEmptyArchive: true, artifacts: 'changelog.txt'
                }
            }
        }
        stage("Build Container Images") {
                parallel {
                    stage("Tensorflow: Fedora27") {
                        steps { // FIXME we could have a conditional build here
                            echo "Building Tensorflow container image..."
                            script {
                                tagMap['tensorflow-fedora27'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, "tensorflow-fedora27")
                            }

                            echo "Building Tensorflow Test container image..."
                            script {
                                tagMap['tensorflow-fedora27-test'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, "tensorflow-fedora27-test")
                            }
                            sh "curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' 'http://user-api.thoth-test-core.svc/api/v1/analyze?image=docker-registry.default.svc%3A5000%2Fthoth-test-aistacks%2Ftensorflow-fedora27-test%3Alatest&analyzer=fridex%2Fthoth-package-extract&registry_user=serviceaccount&registry_password=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ0aG90aC10ZXN0LWFpc3RhY2tzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlcGxveWVyLXRva2VuLThjNmZmIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlcGxveWVyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZWMwZDBmODYtM2Q2Yi0xMWU4LWIxMzMtZmExNjNlZjIwNDcxIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OnRob3RoLXRlc3QtYWlzdGFja3M6ZGVwbG95ZXIifQ.fYmyIgbm6Xu_fw-a-awJv48TvNj3G8WFZzVQxc_VuLDXKULatq2rST_1UUD-Ys_yAnQKtJ6GmJMbd1e1xQTNqD7SfLYeiCgfJi2i214PcWGXF7pXbCnyDycIzWmLkVbDv12rxe3jxPkCSX5L7zDgbE-tQY0EU2FLLnyBQFuL4_Qhd97XRx98Blws10QtPEBqxp2YcdEt-fRoAIUMlC_xfN6SkR-vIqkKpr62G7y7OMpA2NPTKhQdbZi-8ylP5r6Rv0HdbA3-nqgLxDLkyuCF85XDCbXxJme9QXFgT16tYtOAhSL7aT-9xIZRS3cqDKMb3hUcuKIGx9_caIh4IRypdA&debug=true'"
                        }          
                    }
                    stage("Tensorflow: CentOS7+Python3") {
                        steps { // FIXME we could have a conditional build here
                            script {
                                tagMap['tensorflow-centos7-python3'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, "tensorflow-centos7-python3")
                            }
                        }                
                    }
                    stage("Scikit-Image: CentOS7+Python3") {
                        steps { // FIXME we could have a conditional build here
                            script {
                                tagMap['scikit-image-centos7-python3'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, "scikit-image-centos7-python3")
                            }
                        }                
                    }
                    stage("Scikit-Image: CentOS7+Python2") {
                        steps { // FIXME we could have a conditional build here
                            script {
                                tagMap['scikit-image-centos7-python2'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, "scikit-image-centos7-python2")
                            }
                        }                
                    }
                    stage("radanalytics: Tensorflow: Neural-Style S2I") {
                        steps { // FIXME we could have a conditional build here
                            script {
                                tagMap['tensorflow-neural-style-s2i'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, 'tensorflow-neural-style-s2i')
                            }
                        }                
                    }
                    stage("radanalytics: Tensorflow: S2I") {
                        steps { // FIXME we could have a conditional build here
                            script {
                                tagMap['tensorflow-build-s2i'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, 'tensorflow-build-s2i')
                            }
                        }                
                    }
                    stage("radanalytics: Tensorflow: Serving GPU S2I") {
                        steps { // FIXME we could have a conditional build here
                            script {
                                tagMap['tensorflow-serving-gpu-s2i'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, 'tensorflow-serving-gpu-s2i')
                            }
                        }                
                    }
                    stage("radanalytics: Jupyter: Python 3.5") {
                        steps { // FIXME we could have a conditional build here
                            script {
                                tagMap['jupyter-notebook-py35'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, 'jupyter-notebook-py35')
                            }
                        }                
                    }
                    stage("radanalytics: Jupyer Notebooks") {
                        steps { 
                            script {
                                tagMap['base-notebook'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, 'base-notebook')
                            }
                            script {
                                tagMap['tf-base-notebook'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, 'tf-base-notebook')
                            }
                        }                
                    } /*
                    stage("GCC 6.3.0: CentOS7") {
                        steps { // FIXME we could have a conditional build here
                            script {
                                tagMap['gcc630-centos7'] = pipelineUtils.buildStableImage(CI_TEST_NAMESPACE, "gcc630-centos7")
                            }
                        }                
                    } */
                }
        }
        stage("Run Tests") {
            failFast true
            parallel {
                stage("Functional Tests") {
                    steps {
                        echo "TODO"
                    }
                }
                stage("Performance Tests") {
                    steps {
                        echo "TODO"
                    }
                }
            }
        }
        stage("Image Tag Report") {
            steps {
                script {
                    pipelineUtils.printLabelMap(tagMap)
                }
            }
        }
    }
    post {
        always {
            script {
                // junit 'reports/*.xml'

                pipelineUtils.sendIRCNotification("${IRC_NICK}", 
                    IRC_CHANNEL, 
                    "${JOB_NAME} #${BUILD_NUMBER}: ${currentBuild.currentResult}: ${BUILD_URL}")
            }
        }
        success {
            echo "All Systems GO!"
        }
        failure {
            script {
                mattermostSend channel: "#thoth-station", 
                    icon: 'https://avatars1.githubusercontent.com/u/33906690', 
                    message: "${JOB_NAME} #${BUILD_NUMBER}: ${currentBuild.currentResult}: ${BUILD_URL}"

                error "BREAK BREAK BREAK - build failed!"
            }
        }
    }

}