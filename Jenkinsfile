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
                    def namespace = "swag-intg" // Namespace for the Redis pod
                    def timeoutMinutes = 10 // Set the timeout duration
                    def running = false
                    
                    // Loop until all Redis instances are running or timeout
                    timeout(time: timeoutMinutes, unit: 'MINUTES') {
                        redisHosts.each { host ->
                            // Check each Redis cluster pod
                            while (!running) {
                                running = true // Assume all are running until proven otherwise
                                // Check the specific Redis pod status
                                echo "kubectl get pods ${host} -n ${namespace} --no-headers"
                                def podStatus = bat(script: "kubectl get pods ${host} -n ${namespace}  --no-headers", returnStdout: true).trim()
                                echo "${podStatus}"
                                if (!podStatus.contains("Running") ) {
                                    running = false // Set to false if the specific pod is not running
                                    echo "${host} in namespace ${namespace} is not running yet."
                                } else {
                                    echo "${host} in namespace ${namespace} is running."
                                }
                                // Wait before the next check
                                echo "Waiting before checking again..."
                                sleep(time: 20, unit: 'SECONDS')
                            }
                        }
                    }
                }
            }
        }


    }
}
