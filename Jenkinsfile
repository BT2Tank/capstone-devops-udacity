pipeline {
    agent any
    environment {
        PATH="$HOME/.local/bin:$PATH"
    }
    stages {
        stage('Installing Dependencies via Makefile') {
            steps {
                sh 'make install'
            }
        }
        stage('Lint Checks on the Code') {
            steps {
                sh 'make lint'
            }
        }
	    stage('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOC_USERNAME', passwordVariable: 'DOC_PASSWORD')]) {
                    sh 'docker build -t thanhta97/capstoneproject .'
                }
            }
        }
	    stage('Upload Docker Image to Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOC_USERNAME', passwordVariable: 'DOC_PASSWORD')]) {
                    sh '''
                        docker login -u $DOC_USERNAME -p $DOC_PASSWORD
                        docker push thanhta97/capstoneproject
                    '''
                }
            }
        }
		stage('Added New kubectl Context') {
			steps {
				withAWS(region:'us-west-2', credentials:'eks-credentials') {
					sh '''
					    aws eks --region us-west-2 update-kubeconfig --name clustercapstone
					'''
				}
			}
		}
        stage('Set Current kubectl Context') {
			steps {
				withAWS(region:'us-west-2', credentials:'eks-credentials') {
					sh '''
					    kubectl config use-context arn:aws:eks:us-west-2:802725115046:cluster/clustercapstone
					'''
				}
			}
		}
        stage('Deploy Blue Container') {
			steps {
				withAWS(region:'us-west-2', credentials:'eks-credentials') {
					sh '''
						kubectl apply -f ./deployment/blue-controller.json
					'''
				}
			}
		}
		stage('Deploy Green Container') {
			steps {
				withAWS(region:'us-west-2', credentials:'eks-credentials') {
					sh '''
						kubectl apply -f ./deployment/green-controller.json
					'''
				}
			}
		}
        stage('Create Service in the Cluster-Blue') {
			steps {
				withAWS(region:'us-west-2', credentials:'eks-credentials') {
					sh '''
						kubectl apply -f ./deployment/blue-service.json
					'''
				}
			}
		}
		stage('Wait for User Permission') {
            steps {
                input "Do you want to redirect traffic to green?"
            }
        }
        stage('Create Service in the Cluster-Green') {
			steps {
				withAWS(region:'us-west-2', credentials:'eks-credentials') {
					sh '''
						kubectl apply -f ./deployment/green-service.json
					'''
				}
			}
		}
    }
}
