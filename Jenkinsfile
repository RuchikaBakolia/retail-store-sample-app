pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-northeast-3' // Set your AWS region
        PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:$PATH"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh 'which terraform'
                sh 'which helm'
            }
        }

        stage('Debug Environment') {
            steps {
                sh 'echo $PATH'
                sh 'which sh'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'terraform --version'
                sh 'helm version'
                sh 'aws --version'
            }
        }

        // stage('Terraform Init & Plan') {
        //     steps {
        //         echo 'Initializing and planning Terraform...'
        //         dir('terraform/eks/default') {
        //             sh 'terraform init'
        //             sh 'terraform plan -out=tfplan'
        //         }
        //     }
        // }

        // stage('Terraform Apply') {
        //     steps {
        //         echo 'Applying Terraform changes...'
        //         dir('terraform/eks/default') {
        //             sh 'terraform apply -auto-approve tfplan'
        //         }
        //     }
        // }

        // stage('Generate Kubeconfig') {
        //     steps {
        //         echo 'Generating kubeconfig...'
        //         dir('terraform/eks/default') {
        //             script {
        //                 // Use AWS CLI to generate kubeconfig for the EKS cluster
        //                 sh '''
        //                 aws eks update-kubeconfig \
        //                     --region ${AWS_REGION} \
        //                     --name $(terraform output -raw eks_cluster_name)
        //                 '''
        //             }
        //         }
        //     }
        // }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'
                sh 'docker build -t my-app-catalog ./src/catalog'
                sh 'docker build -t my-app-cart ./src/cart'
                sh 'docker build -t my-app-checkout ./src/checkout'
                sh 'docker build -t my-app-orders ./src/orders'
                sh 'docker build -t my-app-ui ./src/ui'
            }
        }

        stage('Push Docker Images') {
            steps {
                echo 'Pushing Docker images to registry...'
                sh 'docker tag my-app-catalog rbakolia132/my-app-catalog:latest'
                sh 'docker push rbakolia132/my-app-catalog:latest'
                sh 'docker tag my-app-cart rbakolia132/my-app-cart:latest'
                sh 'docker push rbakolia132/my-app-cart:latest'
                sh 'docker tag my-app-checkout rbakolia132/my-app-checkout:latest'
                sh 'docker push rbakolia132/my-app-checkout:latest'
                sh 'docker tag my-app-orders rbakolia132/my-app-orders:latest'
                sh 'docker push rbakolia132/my-app-orders:latest'
                sh 'docker tag my-app-ui rbakolia132/my-app-ui:latest'
                sh 'docker push rbakolia132/my-app-ui:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes using Helm...'
                dir('terraform/eks/default') {
                    sh 'helm upgrade --install catalog ./src/catalog/chart --namespace catalog --values ./terraform/eks/default/values/catalog.yaml'
                    sh 'helm upgrade --install cart ./src/cart/chart --namespace carts --values ./terraform/eks/default/values/carts.yaml'
                    sh 'helm upgrade --install checkout ./src/checkout/chart --namespace checkout --values ./terraform/eks/default/values/checkout.yaml'
                    sh 'helm upgrade --install orders ./src/orders/chart --namespace orders --values ./terraform/eks/default/values/orders.yaml'
                    sh 'helm upgrade --install ui ./src/ui/chart --namespace ui --values ./terraform/eks/default/values/ui.yaml'
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            // cleanWs()
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed!'
            // dir('terraform/eks/default') {
            //     sh 'terraform init' // Ensure backend is initialized
            //     sh 'terraform destroy -auto-approve'
            }
        }
    }
    