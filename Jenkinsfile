pipeline {
            agent any
			parameters {
				string defaultValue: 'helloworld', description: '', name: 'BLDCFG', trim: false
				string defaultValue: 'helloworld', description: '', name: 'DPCFG', trim: false
				string defaultValue: 'helloworld', description: '', name: 'APPNAME', trim: false
				string defaultValue: 'helloworld-uat', description: '', name: 'UATDPCFG', trim: false
				string defaultValue: 'helloworld-uat', description: '', name: 'UATAPP', trim: false
				string defaultValue: 'helloworld', description: '', name: 'DEVPROJ', trim: false
				string defaultValue: 'helloworld', description: '', name: 'UATPROJ', trim: false
			}
            stages {
                stage('build') {
                    steps {
                        script {
                            openshift.withCluster() {
                                openshift.withProject("${params.DEVPROJ}") {
                                echo "Using project: ${openshift.project()} to wait for app to finish building"
                                openshift.selector("bc", "${params.BLDCFG}").startBuild("--wait=true")
                                openshift.tag("APPNAME:latest", "APPNAME:${BUILD_NUMBER}")     
                                openshift.selector("dc", "${params.DPCFG}").related('pods').untilEach(1) {
                                        echo "waiting for pod to finish deploy"
                                        return (it.object().status.phase == "Running")
                                        }
                                userInput = input(
                                        id: 'Proceed1', message: 'Is this ready to send to UAT environment?', parameters: [
                                        [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you want to send to the UAT environment']
                                    ])
                                
                                }
                            }
                        } // script
                    } // steps
                } // stage
                stage('uat-staging') {
                when {
                        expression {
                            openshift.withCluster() {
                                openshift.withProject("${params.UATPROJ}") {
                                    return !openshift.selector('dc', "${params.UATDPCFG}").exists()
                                }
                            }
                        }
                    } // when
                    steps {
                        script {
                            openshift.withCluster() {
                                openshift.withProject("${params.UATPROJ}") {
                                openshift.tag("${params.DEVPROJ}/APPNAME:latest", "${params.UATAPP}:latest")
                                echo "Using project: ${openshift.project()} to deploy uat"

                                openshift.newApp("${params.UATAPP}").narrow('svc').expose()
                                
                                    }    
                                    
                                }
                            
                        } // script
                    } // steps
                } // stage
                
            } // stages
        } // pipeline