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

        stage('Verify Deployment') {
            steps {
                script {
                    // Verify that Redis pods are up and running
                    bat 'kubectl get pods -l app=redis'
                }
            }
        }

        stage('Initialize Redis Cluster') {
            steps {
                script {
                    // Get Redis pod names for initialization
                    def redisPods = sh(returnStdout: true, script: "kubectl get pods -l app=redis -o jsonpath='{.items[*].metadata.name}'  -n swag-intg").trim()
                    def redisNodes = []
                    
                    // Get the IP addresses of Redis nodes
                    for (pod in redisPods.tokenize(' ')) {
                        redisNodes.add(sh(returnStdout: true, script: "kubectl get pod ${pod} -o jsonpath='{.status.podIP}' -n swag-intg").trim())
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
