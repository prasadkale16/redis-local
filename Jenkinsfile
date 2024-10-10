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
                    echo "${redisHosts}"
                    // Loop until all Redis instances are running or timeout
                    timeout(time: timeoutMinutes, unit: 'MINUTES') {
                        redisHosts.each { host ->
                            echo "checking for ${host}"
                            running = false
                            // Check each Redis cluster pod
                            while (!running) {
                                // Check the specific Redis pod status
                                echo "kubectl get pods ${host} -n ${namespace} --no-headers"
                                def podStatus = bat(script: "kubectl get pods ${host} -n ${namespace}  --no-headers", returnStdout: true).trim()
                                echo "${podStatus}"
                                if (!podStatus.contains("Running") ) {
                                    echo "${host} in namespace ${namespace} is not running yet."
                                    // Wait before the next check
                                    echo "Waiting before checking again..."
                                    sleep(time: 10, unit: 'SECONDS')
                                } else {
                                    echo "${host} in namespace ${namespace} is running."
                                    running = true
                                }
                                
                            }
                        }
                    }
                }
            }
        }

        stage('Initialize Redis Cluster') {
            steps {
                script {
                    // Get Redis pod names for initialization
                    def podIPs = bat(script: 'kubectl get pods -l app=redis -o jsonpath="{.items[*].status.podIP}" -n swag-intg', returnStdout: true).trim().split('\n')[-1].trim()
                    // Output only the clean pod IPs
                    echo "Redis Pod IPs: ${podIPs}"

                    def ipList = ""
                    podIPs.split(" ").each { ip ->
                        ipList = ipList + ip+":6379 "
                    }

                    // Form a Redis cluster
                  //  def clusterCommand = "kubectl exec -it redis-0 -n swag-intg  -- redis-cli --cluster create  --cluster-replicas 1 ${ipList}"
                  def clusterCommand = "kubectl exec -i redis-0 -n swag-intg -- sh -c \"echo yes | redis-cli --cluster create ${ipList} --cluster-replicas 1\""
                    
                   // echo "redis command : ${clusterCommand}"
                    // Run the Redis cluster creation command
                   bat clusterCommand

                    //bat(script: 'kubectl exec -i redis-0 -n swag-intg -- sh -c "echo \\"yes\\" | redis-cli --cluster create ${ipList} --cluster-replicas 1')

                    
                }
            }
        }

    }
}
