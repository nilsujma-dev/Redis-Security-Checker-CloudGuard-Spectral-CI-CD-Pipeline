pipeline {
    agent any

    environment {
        // Use Jenkins credentials binding
        CHKP_CLOUDGUARD_ID = credentials('chkp-cloudguard-id')
        CHKP_CLOUDGUARD_SECRET = credentials('chkp-cloudguard-secret')
        SHIFTLEFT_REGION = "eu1"
        SPECTRAL_DSN = credentials('spectral-dsn')
    }

    stages {
        stage('Clone Github repository') {
            steps {
                checkout scm
            }
        }

        stage('install Spectral') {
            steps {
                sh "curl -L 'https://spectral-eu.checkpoint.com/latest/x/sh?dsn=$SPECTRAL_DSN' | sh"
            }
        }

        stage('Pre-Build Spectral Scan for Secrets,Misconfiguration and IaC') {
            steps {
                sh "$HOME/.spectral/spectral scan --engines secrets,oss --include-tags base,audit --ok"
            }
        }

        stage('Docker image Build and scan prep') {
            steps {
                sh 'docker build -t nilsujma/redis .'
                sh 'docker save nilsujma/redis -o redis.tar'
            }
        }

        stage('ShiftLeft Container Image Scan Online') {
            steps {
                script {
                    try {
                        sh 'chmod +x shiftleft'
                        sh './shiftleft image-scan -t 180 -i redis.tar -r -2002 -e 4b89d765-1dfd-4c19-bf26-a34374142d42'
                    } catch (Exception e) {
                        echo "ShiftLeft docker image scan failed."
                    }
                }
            }
        }

        stage('Shiftleft Container Image Scan Offline') {
            steps {
                script {
                    try {
                        sh 'chmod +x shiftleft'
                        sh './shiftleft image-scan -t 180 -i redis.tar'
                    } catch (Exception e) {
                        echo "ShiftLeft docker Image scan failed."
                    }
                }
            }
        }

        stage('Post-Build Spectral Scan for Secrets,Misconfiguration and IaC') {
            steps {
                sh "$HOME/.spectral/spectral scan --engines secrets,oss --include-tags base,audit --ok"
            }
        }

        stage('Login to Azure ACR') {
            steps {
                withCredentials([
                    string(credentialsId: 'azure-app-id', variable: 'APP_ID'),
                    string(credentialsId: 'azure-password', variable: 'PASSWORD'),
                    string(credentialsId: 'azure-tenant-id', variable: 'TENANT_ID')
                ]) {
                    script {
                        sh """
                            az login --service-principal -u $APP_ID -p $PASSWORD --tenant $TENANT_ID
                            az acr login --name aksk8sreg
                        """
                    }
                }
            }
        }

        stage('Push Image to ACR') {
            steps {
                script {
                    sh """
                        docker tag nilsujma/redis:latest aksk8sreg.azurecr.io/nilsujma/redis:latest
                        docker push aksk8sreg.azurecr.io/nilsujma/redis:latest
                    """
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                script {
                    // Login to AKS
                    sh """
                        az aks get-credentials --resource-group aks-k8s-cluster_group --name aks-k8s-cluster
                    """
                
                    // Apply the Kubernetes configurations to deploy the image
                    sh """
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                    """
                }
            }
        }
    }
}
