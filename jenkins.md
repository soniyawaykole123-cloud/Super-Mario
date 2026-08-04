## CONFIGURING JENKKINS PIPELINE

### Install following plugins
````
terraform
stage view
aws credentials
````

### manage jenkins >> tools
**terraform** = terraform

### manage jenkins >> credentials 
kind: aws credentaials


### Job1 pipeline sysntax 
![parameters](./.Images/j1.png)

```groovy
pipeline {
    agent { label 'agent' }

    tools {
        terraform 'terraform'
    }

    parameters {
        choice(
            name: 'action',
            choices: ['apply', 'destroy'],
            description: 'Select Terraform Action'
        )
    }

    stages {

        stage('Code Pull') {
            steps {
                git branch: 'main',
                url: 'https://github.com/soniyawaykole123-cloud/Super-Mario.git'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('EKS-TF') {
                    withCredentials([
                        aws(
                            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                            credentialsId: 'aws-cred',
                            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                        )
                    ]) {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                dir('EKS-TF') {
                    sh 'terraform validate'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('EKS-TF') {
                    withCredentials([
                        aws(
                            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                            credentialsId: 'aws-cred',
                            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                        )
                    ]) {
                        sh 'terraform plan'
                    }
                }
            }
        }

        stage('Terraform Action') {
            steps {
                dir('EKS-TF') {
                    withCredentials([
                        aws(
                            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                            credentialsId: 'aws-cred',
                            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                        )
                    ]) {
                        sh "terraform ${params.action} -auto-approve"
                    }
                }
            }
        }

        stage('Trigger Job2') {
            when {
                expression { params.action == 'apply' }
            }
            steps {
                build job: 'job2', wait: true
            }
        }

        stage('Completed') {
            steps {
                echo 'Back from Job2'
            }
        }
    }
}
```
## job2 which will deploy your k8s pods to run an application 
```groovy
pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
    }

    stages {

        stage('Code Pull') {
            steps {
                git branch: 'main',
                url: 'https://github.com/soniyawaykole123-cloud/Super-Mario.git'
            }
        }

        stage('Deploy To EKS') {
            steps {

                withCredentials([
                    aws(
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        credentialsId: 'aws-cred',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {

                    sh '''
                    aws eks update-kubeconfig \
                    --region ap-south-1 \
                    --name EKS_CLOUD

                    kubectl get nodes

                    kubectl apply -f deployment.yaml

                    kubectl apply -f service.yaml

                    kubectl get pods

                    kubectl get svc
                    '''
                }
            }
        }
    }
}
```




















