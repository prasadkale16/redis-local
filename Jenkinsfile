pipeline {
    agent any

    environment {
        KUBECONFIG = 'C:\\Users\\Lenovo\\.kube\\config'  // Kubernetes config file path
    }

    stages {
        stage('Deploy ConfigMap') {
            steps {
                script {
                    // Apply Redis ConfigMap for Redis configuration
                    bat 'kubectl apply -f redis-configmap.yaml'
                }
            }
        }

        stage('Deploy Headless Service') {
            steps {
                script {
                    // Apply Redis Headless Service for cluster communication
                    bat 'kubectl apply -f redis-headless-service.yaml'
                }
            }
        }

        stage('Deploy Redis StatefulSet') {
            steps {
                script {
                    // Apply Redis StatefulSet for cluster setup
                    bat 'kubectl apply -f redis-statefulset.yaml'
                }
            }
        }

        stage('Check Redis Instances') {
            steps {
                script {
                    def redisHosts = (0..5).collect { "redis-${it}" }
                    def redisSpecificPod = "redis-0" // Specific Redis pod to check
                    def namespace = "swag-intg" // Namespace for the Redis pod
                    def timeoutMinutes = 10 // Set the timeout duration
                    def allRunning = false
                    
                    // Loop until all Redis instances are running or timeout
                    timeout(time: timeoutMinutes, unit: 'MINUTES') {
                        while (!allRunning) {
                            allRunning = true // Assume all are running until proven otherwise
                            
                            // Check each Redis cluster pod
                            redisHosts.each { host ->
                                // Check the specific Redis pod status
                                def podStatus = bat(script: "kubectl get pods ${host} -n ${namespace} ", returnStdout: true).trim()
                            
                                if (podStatus.contains("Running") ) {
                                    allRunning = false // Set to false if the specific pod is not running
                                    echo "${redisSpecificPod} in namespace ${namespace} is not running yet."
                                } else {
                                    echo "${redisSpecificPod} in namespace ${namespace} is running."
                                }
                            }

                            // Wait before the next check
                            sleep(time: 5, unit: 'SECONDS')
                        }
                    }
                }
            }
        }

        stage('Initialize Redis Cluster') {
            steps {
                script {
                    // Get Redis pod names for initialization
                    def redisPods = bat(returnStdout: true, script: "kubectl get pods -l app=redis -o jsonpath='{.items[*].metadata.name}'  -n swag-intg").trim()
                    def redisNodes = []
                    
                    // Get the IP addresses of Redis nodes
                    for (pod in redisPods.tokenize(' ')) {
                        redisNodes.add(bat(returnStdout: true, script: "kubectl get pod ${pod} -o jsonpath='{.status.podIP}' -n swag-intg").trim())
                    }

                    // Form a Redis cluster
                    def clusterCommand = "kubectl exec -it ${redisPods[0]} -- redis-cli --cluster create  -n swag-intg"
                    for (ip in redisNodes) {
                        clusterCommand += "${ip}:6379 "
                    }
                    clusterCommand += "--cluster-replicas 1"

                    // Run the Redis cluster creation command
                    bat clusterCommand
                }
            }
        }
    }
}
