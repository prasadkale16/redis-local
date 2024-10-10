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
                    
                    // Loop until all Redis instances are running or timeout
                    timeout(time: timeoutMinutes, unit: 'MINUTES') {
                        while (redisHosts.size() > 0) { // Continue until all hosts are confirmed running
                            redisHosts.each { host ->
                                // Check the specific Redis pod status
                                echo "Checking status of pod: ${host} in namespace: ${namespace}"
                                def podStatus = bat(script: "kubectl get pod ${host} -n ${namespace} --no-headers", returnStdout: true).trim()
                                echo "${podStatus}"

                                // Check if the output indicates that the pod is running
                                if (!podStatus.contains("Running")) {
                                    echo "${host} in namespace ${namespace} is not running yet."
                                } else {
                                    echo "${host} in namespace ${namespace} is running."
                                    redisHosts.remove(host) // Remove the host from the list if it's running
                                }
                            }

                            // Wait before the next check if there are still hosts left
                            if (redisHosts.size() > 0) {
                                echo "Waiting before checking again..."
                                sleep(time: 10, unit: 'SECONDS')
                            }
                        }

                        echo "All Redis pods are running."
                    }
                }
            }
        }

        


    }
}
